# Test Results

## Phạm vi đã xác minh

Source của bốn boundary có SHA-256:

```text
61c9c67a1db1c648409ff6eb263ca7ca41564d67637470a346179ac398ef8585
```

### Fast suite

```text
$ uv run pytest starter-tests tests -q
........................................................................ [ 82%]
...............                                                          [100%]
87 passed in 2.27s
```

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

### Kiểm tra bổ sung

- Bốn starter test được chạy trực tiếp trên bản đóng gói: đạt 4/4.
- Toàn bộ file Python compile thành công.
- `uv lock --check --offline`: thành công, 146 package được resolve từ lock.
- ZIP không chứa `.git`, `.venv`, `.env`, `.lab28`, cache, database hoặc model
  weights.

## Gate cần hạ tầng thật

`integration-tests -m "not gpu and not langsmith"`, J1–J5, load test và evidence
runtime có trạng thái `UNVERIFIED` khi không có Docker stack. IP07 tiếp tục
`UNVERIFIED` nếu chưa có endpoint vLLM GPU thật. Không dùng mock để thay bằng
chứng live.
