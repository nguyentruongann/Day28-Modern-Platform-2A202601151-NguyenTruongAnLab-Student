# Failure and Recovery Record

## Sự cố đã quan sát thật — Spark Connect OOM

### Dấu hiệu

- Full profile không ổn định: khi Airflow healthy, `spark-connect` chuyển thành
  `Exited (137)`.
- Lệnh kiểm tra trả `OOMKilled=true ExitCode=137`.
- Máy có 11.8 GiB RAM; Docker chỉ nhận 4.8 GiB; Task Manager ghi nhận khoảng
  81% RAM đang sử dụng.

Ảnh: [09-spark-oom-failure.png](docs/submission-screenshots/09-spark-oom-failure.png)
và [10-host-memory-pressure.png](docs/submission-screenshots/10-host-memory-pressure.png).

### Nguyên nhân

Giới hạn RAM của Docker/WSL thấp hơn mức full profile cần. Đây là lỗi tài nguyên,
không phải lỗi bốn hàm cốt lõi. Quá trình build ban đầu còn tải gói
`pyspark==4.2.0` khoảng 450 MB dù Airflow chỉ dùng Spark Connect client.

### Hành động khôi phục đã thực hiện

1. Thay dependency Airflow bằng `pyspark-client==4.2.0`; image Airflow build
   thành công.
2. Khởi động lại full profile mà không xóa volume/state.
3. Lần chạy cuối ghi nhận Spark Connect healthy, nhưng Airflow vẫn `Waiting`
   sau 1422.6 giây; chưa đạt trạng thái full stack ổn định.

### Kết luận no-data-loss

Không dùng `down -v` và không chạy `lab28 reset --yes`, do đó không chủ động xóa
state. Tuy nhiên chưa có Delta row count trước/sau hoặc J2 để chứng minh
no-data-loss end-to-end; mục này vẫn `UNVERIFIED`.

## Kịch bản lab còn cần chạy trên máy đủ tài nguyên

- Dependency: Feast.
- Loại dependency: tùy chọn trên serving path.
- Trạng thái live hiện tại: `UNVERIFIED` — cần full stack ổn định.
- Lý do chọn: minh họa rõ khác biệt giữa health và readiness `degraded` mà không
  xóa dữ liệu nguồn.

## Dự đoán trước khi inject

| Signal | Kết quả dự kiến |
|---|---|
| API `/health` | Vẫn trả thành công |
| API `/ready` | `degraded`, reason chỉ ra Feast |
| Feature lookup | Lỗi hoặc thiếu feature theo degraded policy |
| Delta | Số dòng và version không mất |
| Metrics | Error/latency của feature lookup tăng |
| Trace | Span `lab28.feast.get_online_features` thể hiện lỗi |

## Lệnh demo

```text
uv run lab28 ready
docker compose --env-file ports.template stop feast
uv run lab28 ready
docker compose --env-file ports.template logs feast
docker compose --env-file ports.template start feast
uv run lab28 ready
```

## Tiêu chí khôi phục

1. Feast health trở lại HTTP 200.
2. Readiness trở lại `ready`, hoặc chỉ còn `degraded` vì vLLM chưa nối.
3. Cùng `asker_id` đọc lại được online feature.
4. Delta row count trước và sau không giảm.
5. Không replay DLQ trước khi nguyên nhân lỗi được xử lý.

## Trường evidence cần điền từ lần chạy thật

| Trường | Giá trị |
|---|---|
| Timestamp bắt đầu | `UNVERIFIED` |
| Trace ID | `UNVERIFIED` |
| Delta version/row count trước | `UNVERIFIED` |
| Trạng thái khi lỗi | `UNVERIFIED` |
| Timestamp khôi phục | `UNVERIFIED` |
| Delta version/row count sau | `UNVERIFIED` |

Không dùng `docker compose down -v` hoặc `lab28 reset --yes` trong demo recovery.
