# Test Results

## Phạm vi đã xác minh — Nguyễn Trường An

- Mã sinh viên: `2A202601151`
- Hình thức: cá nhân
- Nền tảng chạy: Windows PowerShell, Python 3.11.16, Docker Desktop/WSL 2

Source của bốn boundary có SHA-256:

```text
61c9c67a1db1c648409ff6eb263ca7ca41564d67637470a346179ac398ef8585
```

### Fast suite

```text
$ uv run pytest starter-tests tests -q
........................................................................ [ 82%]
...............                                                          [100%]
87 passed in 22.72s
```

Ảnh chạy thật: [02-code-static-gates.png](docs/submission-screenshots/02-code-static-gates.png).

### Static gates

```text
$ uv run ruff check .
All checks passed!

$ uv run python scripts/verify_matrix.py
OK    245 checks passed: contracts/integration-matrix.yaml matches the repository

$ uv run python scripts/check_portability.py
OK    supported workflow is host-path and shell independent

$ uv run python scripts/validate_manifests.py
Kubernetes and GitOps manifest contracts passed
```

Các gate tĩnh nằm chung trong ảnh chạy thật phía trên.

### Docker basic profile

| Checkpoint | Kết quả quan sát |
|---|---|
| Preflight | `local-standard`, `local_ready=true`, Docker CLI/daemon sẵn sàng, 8 CPU, 110.8 GiB trống |
| Compose config | Basic và full profile đều trả exit code 0 |
| Containers cơ bản | API, Feast, Kafka, MLflow, Qdrant healthy; gateway và monitoring up |
| Kafka | Bốn topic `data.raw`, `data.processed`, `model.events`, `data.raw.dlq` được tạo |
| Qdrant | `13 points_upserted`, `13 points_total` |
| MLflow | `lab28-rag-release` version 1, alias `champion` |
| Seed trực tiếp | 13 documents và 12 feedback accepted, 0 rejected |
| Readiness | `degraded`; chỉ vLLM unreachable, bốn dependency còn lại ready |

Bằng chứng ảnh: [01-preflight-compose.png](docs/submission-screenshots/01-preflight-compose.png),
[03-basic-containers.png](docs/submission-screenshots/03-basic-containers.png),
[04-kafka-topics.png](docs/submission-screenshots/04-kafka-topics.png),
[05-qdrant-index.png](docs/submission-screenshots/05-qdrant-index.png),
[06-mlflow-champion.png](docs/submission-screenshots/06-mlflow-champion.png),
[07-seed-complete.png](docs/submission-screenshots/07-seed-complete.png) và
[08-readiness-degraded.png](docs/submission-screenshots/08-readiness-degraded.png).

### Docker full profile — chưa hoàn tất

- Airflow image build thành công sau khi dùng đúng Spark Connect client
  `pyspark-client==4.2.0` thay cho gói PySpark đầy đủ 450 MB.
- Một lần chạy ghi nhận Airflow healthy nhưng Spark Connect bị `Exited (137)`;
  `docker inspect` xác nhận `OOMKilled=true`.
- Máy có 11.8 GiB RAM, Docker chỉ nhận 4.8 GiB; Task Manager ghi nhận khoảng
  81% RAM đã sử dụng.
- Lần chạy cuối ghi nhận Spark Connect healthy nhưng Airflow vẫn ở trạng thái
  `Waiting` sau 1422.6 giây. Chưa có một thời điểm cả hai cùng healthy đủ lâu để
  chạy integration suite.

Do đó J1–J5, Delta live history, Feast materialization, full trace và load test
không được ghi là đạt.

### Kiểm tra bổ sung

- Bốn starter test được chạy trực tiếp trên bản đóng gói: đạt 4/4.
- Toàn bộ file Python compile thành công.
- `uv lock --check --offline`: thành công, 146 package được resolve từ lock.
- ZIP không chứa `.git`, `.venv`, `.env`, `.lab28`, cache, database hoặc model
  weights.

## Gate còn thiếu hạ tầng đủ RAM/GPU

`integration-tests -m "not gpu and not langsmith"`, J1–J5, load test và evidence
runtime có trạng thái `UNVERIFIED`. IP07 tiếp tục `UNVERIFIED` nếu chưa có
endpoint vLLM GPU thật. Không dùng mock để thay bằng chứng live.
