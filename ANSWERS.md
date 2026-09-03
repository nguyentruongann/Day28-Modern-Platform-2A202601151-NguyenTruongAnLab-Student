# ANSWERS — Day 28 Track 2

## Thông tin cá nhân

- Họ và tên: Nguyễn Trường An
- Mã sinh viên: 2A202601151
- Hình thức: Cá nhân

## Phần thực hiện và đóng góp

Tôi thực hiện lần lượt các vai trò ingestion/orchestration, data/ML,
serving/retrieval và platform/observability. Phần mã sinh viên sở hữu gồm:

- IP01/IP10: truyền `idempotency-key` và `traceparent` qua Kafka dưới dạng
  bytes; không phát `traceparent` rỗng.
- IP03: deduplicate mỗi `idempotency_key`, giữ event có cặp
  `(occurred_at, event_id)` lớn nhất và sắp xếp đầu ra theo khóa.
- IP04: tạo Feast online request bằng `FEATURE_REFS` từ contracts.
- IP07/IP08: phân loại đúng `ready`, `degraded` và `not_ready` theo mức độ bắt
  buộc của probe.

Các IP còn lại được đối chiếu qua integration matrix, cấu hình Compose,
telemetry, metrics và manifest. Evidence runtime phải được thu từ stack thật;
IP07 chỉ được xác nhận khi endpoint vLLM GPU thật trả `/version`, `/v1/models`
và metrics có tiền tố `vllm:`.

## Điều khó nhất

Điều khó nhất là làm cho nguồn Delta vừa replay-safe vừa không phụ thuộc thứ tự
Kafka giao bản tin. Chỉ loại khóa trùng chưa đủ vì có thể giữ nhầm bản cũ. So
sánh `(occurred_at, event_id)` tạo quy tắc chọn bản mới nhất có tie-break ổn
định; sắp xếp theo `idempotency_key` làm đầu ra tất định.

## Trade-off

`dedupe_latest` chỉ duyệt đầu vào một lần nhưng dùng bộ nhớ tỷ lệ với số khóa
duy nhất trong batch. Cách này phù hợp cho micro-batch trước Delta MERGE. Với
batch lớn trong production, cần giới hạn batch hoặc đưa phép dedupe sang Spark.

Readiness cho phép hệ thống tiếp tục phục vụ khi dependency tùy chọn lỗi bằng
trạng thái `degraded`; đổi lại cần metric và alert rõ ràng để tránh che giấu sự
suy giảm chất lượng.

## Khoảng trống trước production

- Chạy đủ J1–J5 trên hạ tầng thật và lưu bằng chứng có thể đối chiếu run ID,
  trace ID, Delta version và MLflow version.
- Bổ sung TLS/mTLS, quản lý secret tập trung, xác thực và phân quyền.
- Bổ sung replication, backup/restore, quota và kiểm thử disaster recovery cho
  Kafka, Delta, Qdrant, MLflow cùng hệ thống telemetry.
- Xây dựng SLO, error budget, kiểm thử tải dài hạn và capacity planning.
- Xác minh IP07 với endpoint vLLM GPU thật; không dùng server giả lập API.

## Cải tiến tiếp theo

Tôi sẽ bổ sung transactional outbox cho API → Kafka, chaos test cho
Kafka/Airflow/Delta và tự động đối chiếu cùng trace ID giữa Jaeger, Prometheus,
Delta và MLflow trong evidence bundle.
