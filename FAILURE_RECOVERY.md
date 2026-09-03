# Failure and Recovery Record

## Kịch bản

- Dependency: Feast.
- Loại dependency: tùy chọn trên serving path.
- Trạng thái live hiện tại: `UNVERIFIED` — cần chạy trên Docker stack thật.
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
