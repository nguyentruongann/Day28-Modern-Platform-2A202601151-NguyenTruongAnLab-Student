# Submission Index — Nguyễn Trường An

- Mã sinh viên: 2A202601151
- Hình thức: Cá nhân

## Thành phần trong repository

| Yêu cầu | Tệp |
|---|---|
| Source hoàn thiện bốn boundary | `src/lab28_platform/integration_tasks.py` |
| Kết quả fast suite và static gates | `TEST_RESULTS.md` |
| Kiến trúc và ownership cá nhân | `ARCHITECTURE_OWNERSHIP.md` |
| Giải thích IP01–IP10 và reflection | `ANSWERS.md` |
| Kịch bản sự cố và tiêu chí no-data-loss | `FAILURE_RECOVERY.md` |
| Phương pháp đo và bottleneck analysis | `PERFORMANCE_REPORT.md` |
| Kubernetes/GitOps validation và rollback | `GITOPS_VALIDATION.md` |
| Ma trận evidence bắt buộc | `EVIDENCE_STATUS.md` |

## Trạng thái gate

| Gate | Trạng thái |
|---|---|
| Starter tests | Đạt 4/4 |
| Fast suite | Đạt 87 test |
| Ruff | Đạt |
| Integration matrix | Đạt 245 checks |
| Portability | Đạt |
| Kubernetes/GitOps manifests | Đạt |
| J1–J5 trên Docker stack | `UNVERIFIED` — cần hạ tầng thật |
| vLLM GPU | `UNVERIFIED` — cần endpoint thật |
| P50/P95/P99 | `UNVERIFIED` — cần stack thật |

Các mục `UNVERIFIED` không được thay bằng dữ liệu giả. Sau khi chạy stack, dùng
`uv run lab28 evidence`, lưu output `uv run lab28 integration` thành
`integration-report.json`, rồi cập nhật `EVIDENCE_STATUS.md` bằng ID/version
thật trước khi nộp evidence bundle.
