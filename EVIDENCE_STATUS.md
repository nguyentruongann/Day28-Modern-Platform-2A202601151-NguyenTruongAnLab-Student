# Evidence Status — Individual Submission

Quy ước: `PARTIAL` chỉ ghi nhận checkpoint thật đã quan sát, không thay thế file
JSON bắt buộc do integration test tạo. `OFFLINE_VERIFIED` là contract đã qua test
nhưng chưa có bằng chứng hạ tầng live.

| IP | File bắt buộc | Bằng chứng hiện có | Trạng thái |
|---|---|---|---|
| IP01 | `evidence/ip01-kafka-consume.json` | Bốn topic đã tạo; seed trả event ID, idempotency key và trace ID | `PARTIAL` — chưa consume Kafka thành JSON |
| IP02 | `evidence/ip02-airflow-run.json` | Airflow image build thành công và từng healthy | `PARTIAL` — chưa có DAG run/task states |
| IP03 | `evidence/ip03-delta-history.json` | Dedupe/merge contract nằm trong fast suite 87 test | `OFFLINE_VERIFIED` — chưa có Delta history live |
| IP04 | `evidence/ip04-feast-online.json` | Feast container healthy và readiness ready | `PARTIAL` — chưa materialize/đọc entity sau DAG |
| IP05 | `evidence/ip05-qdrant-search.json` | Qdrant live có 13 points | `PARTIAL` — chưa sinh JSON chuẩn từ `lab28 evidence` |
| IP06 | `evidence/ip06-mlflow-release.json` | MLflow live có release v1, alias champion | `PARTIAL` — chưa sinh JSON chuẩn |
| IP07 | `evidence/ip07-vllm-identity.json` | Readiness xác nhận endpoint unreachable | `UNVERIFIED` — chưa có vLLM thật |
| IP08 | `evidence/ip08-gateway.json` | Gateway container up; basic seed qua gateway đã đi được một phần trước rate limit | `PARTIAL` — chưa có cặp response 200/429 kèm request ID |
| IP09 | `evidence/ip09-prometheus-targets.json` và dashboard | Prometheus/Grafana containers up | `PARTIAL` — chưa chụp targets/dashboard |
| IP10 | `evidence/ip10-trace.json` | Response ingestion có trace ID | `PARTIAL` — chưa query được trace xuyên luồng trong Jaeger |

## Ảnh bằng chứng đã thu

| Ảnh | Điều được chứng minh |
|---|---|
| [01-preflight-compose.png](docs/submission-screenshots/01-preflight-compose.png) | Máy, Docker và hai Compose config hợp lệ |
| [02-code-static-gates.png](docs/submission-screenshots/02-code-static-gates.png) | 87 test, Ruff, matrix 245, portability, manifests |
| [03-basic-containers.png](docs/submission-screenshots/03-basic-containers.png) | Basic stack up/healthy |
| [04-kafka-topics.png](docs/submission-screenshots/04-kafka-topics.png) | Bốn Kafka topic được tạo |
| [05-qdrant-index.png](docs/submission-screenshots/05-qdrant-index.png) | 13 Qdrant points |
| [06-mlflow-champion.png](docs/submission-screenshots/06-mlflow-champion.png) | Release v1 và alias champion |
| [07-seed-complete.png](docs/submission-screenshots/07-seed-complete.png) | 13 documents + 12 feedback, 0 rejected |
| [08-readiness-degraded.png](docs/submission-screenshots/08-readiness-degraded.png) | Degraded đúng vì thiếu vLLM; dependency còn lại ready |
| [09-spark-oom-failure.png](docs/submission-screenshots/09-spark-oom-failure.png) | Sự cố Spark exit 137 trong full profile |
| [10-host-memory-pressure.png](docs/submission-screenshots/10-host-memory-pressure.png) | Áp lực RAM trên máy chạy |

Lệnh thu phần evidence do CLI hỗ trợ:

```text
uv run lab28 evidence
uv run lab28 integration > integration-report.json
```

Không tạo file JSON rỗng và không chuyển `UNVERIFIED`/`PARTIAL` thành `ready` nếu
chưa có đầu ra từ hạ tầng thật. Evidence hoàn chỉnh phải có timestamp,
run/trace/data/model ID và nguồn tạo.
