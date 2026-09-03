# Kubernetes and GitOps Validation

## Kết quả static

```text
$ uv run python scripts/validate_manifests.py
Kubernetes and GitOps manifest contracts passed
```

Các contract đã được kiểm tra:

- Deployment chạy non-root và bỏ Linux capabilities.
- Có liveness, readiness, startup probes và resource requests/limits.
- Image dùng tag bất biến `3.0.0`, không dùng `latest`.
- Gateway và HTTPRoute dùng Gateway API `v1`.
- Argo CD trỏ đến revision cố định `refs/tags/v3.0.0`.
- Automated sync bật `prune` và `selfHeal`; revision history giữ 5 bản.

## Quy trình drift và rollback

1. Build image bất biến và cập nhật tag trong Git.
2. Review diff rồi Argo CD sync desired state.
3. Kiểm tra health, readiness, gateway và smoke test.
4. Thay đổi thủ công một field an toàn để tạo drift.
5. Quan sát Argo CD phát hiện drift và self-heal về desired state.
6. Revert Git revision/image tag về bản trước.
7. Xác minh replicas, gateway, trace và release behavior sau rollback.

Live drift/self-heal evidence: `UNVERIFIED` khi chưa có Kubernetes/Argo CD thật.
