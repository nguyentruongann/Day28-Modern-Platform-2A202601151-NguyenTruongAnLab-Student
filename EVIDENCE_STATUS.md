# Evidence Status — Individual Submission

| IP | File bắt buộc | Cách tạo hợp lệ | Trạng thái |
|---|---|---|---|
| IP01 | `evidence/ip01-kafka-consume.json` | Consume `data.raw`, giữ event/key/header | `UNVERIFIED` |
| IP02 | `evidence/ip02-airflow-run.json` | Airflow run ID, task states, asset event | `UNVERIFIED` |
| IP03 | `evidence/ip03-delta-history.json` | `uv run lab28 evidence` sau J1/J2 | `UNVERIFIED` |
| IP04 | `evidence/ip04-feast-online.json` | Đọc online row sau materialization | `UNVERIFIED` |
| IP05 | `evidence/ip05-qdrant-search.json` | `uv run lab28 evidence` sau index | `UNVERIFIED` |
| IP06 | `evidence/ip06-mlflow-release.json` | `uv run lab28 evidence` sau release | `UNVERIFIED` |
| IP07 | `evidence/ip07-vllm-identity.json` | vLLM thật: version/models/metrics | `UNVERIFIED` |
| IP08 | `evidence/ip08-gateway.json` | Lưu response 200 và 429 có request ID | `UNVERIFIED` |
| IP09 | `evidence/ip09-prometheus-targets.json` và dashboard | Prometheus targets + Grafana/alert | `UNVERIFIED` |
| IP10 | `evidence/ip10-trace.json` | Query trace có đủ required spans | `UNVERIFIED` |

Lệnh thu phần evidence do CLI hỗ trợ:

```text
uv run lab28 evidence
uv run lab28 integration > integration-report.json
```

Không tạo file JSON rỗng và không chuyển `UNVERIFIED` thành `ready` nếu chưa có
đầu ra từ hạ tầng thật. Evidence hợp lệ phải có timestamp, run/trace/data/model
ID và nguồn tạo.
