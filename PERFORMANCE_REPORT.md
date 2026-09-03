# Performance Report

## Trạng thái

Load profile live: `UNVERIFIED` — cần API/Docker stack đang chạy. Không điền số
P50/P95/P99 giả định vì kết quả phụ thuộc phần cứng, model, dataset, warm-up và
degraded policy.

## Tài nguyên đã đo trên máy chạy

| Chỉ số | Giá trị quan sát |
|---|---:|
| RAM vật lý | 11.8 GiB |
| RAM Docker nhìn thấy | 4.8 GiB |
| Mức dùng RAM tại thời điểm chụp | khoảng 81% |
| Kết quả full profile | Spark Connect từng bị OOM, exit code 137 |

Ảnh: [10-host-memory-pressure.png](docs/submission-screenshots/10-host-memory-pressure.png).
Số liệu này chỉ chứng minh bottleneck tài nguyên; không thay thế P50/P95/P99.

## Cấu hình đo bắt buộc

```text
uv run python load-tests/run_profile.py --requests 200 --workers 8
uv run python load-tests/run_profile.py --requests 200 --workers 16
```

Ghi lại CPU, RAM, model ID, GPU, corpus, `top_k`, số request warm-up, error rate,
Kafka lag và trạng thái vLLM cho từng lần chạy.

## Bảng kết quả

| Workers | Requests | P50 | P95 | P99 | Error rate | Trạng thái |
|---:|---:|---:|---:|---:|---:|---|
| 8 | 200 | `UNVERIFIED` | `UNVERIFIED` | `UNVERIFIED` | `UNVERIFIED` | Chờ stack thật |
| 16 | 200 | `UNVERIFIED` | `UNVERIFIED` | `UNVERIFIED` | `UNVERIFIED` | Chờ stack thật |

## Phân tích bottleneck

- Quan sát thật: memory pressure là bottleneck đầu tiên; full integration chưa
  đủ ổn định để đo throughput/latency có ý nghĩa.
- Gateway: rate limit hoặc connection saturation tạo 429/queueing trước API.
- API: sync I/O hoặc worker thiếu làm tăng request duration và saturation.
- Kafka/Airflow: consumer lag tăng khi ingestion nhanh hơn tốc độ xử lý batch.
- Delta: nhiều file nhỏ và MERGE lớn làm tăng thời gian commit.
- Feast/Qdrant: lookup/search latency tăng theo network, collection và cache.
- vLLM: queue time, token generation, model size và GPU memory thường chi phối
  P95/P99 của `/api/v1/ask`.

Không suy luận production capacity từ kết quả laptop. Capacity conclusion chỉ
hợp lệ khi kèm phần cứng, workload và SLO cụ thể.
