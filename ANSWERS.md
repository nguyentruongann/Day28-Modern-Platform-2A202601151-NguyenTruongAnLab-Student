# ANSWERS — Day 28 Track 2

## Thông tin cá nhân

- Họ và tên: Nguyễn Trường An
- Mã sinh viên: 2A202601151
- Hình thức: Cá nhân

## Phần thực hiện và đóng góp cá nhân

Bài làm cá nhân bao quát lần lượt các vai trò ingestion/orchestration, data/ML,
serving/retrieval và platform/observability. Phần mã được hoàn thiện gồm:

- IP01/IP10: truyền `idempotency-key` và `traceparent` qua Kafka dưới dạng
  bytes; không phát `traceparent` rỗng.
- IP03: deduplicate mỗi `idempotency_key`, giữ event có cặp
  `(occurred_at, event_id)` lớn nhất và sắp xếp đầu ra theo khóa.
- IP04: tạo Feast online request bằng `FEATURE_REFS` từ contracts.
- IP07/IP08: phân loại đúng `ready`, `degraded` và `not_ready` theo mức độ bắt
  buộc của probe.

Toàn bộ vai trò trong bài cá nhân được quy về Nguyễn Trường An:

| Vai trò | Phạm vi |
|---|---|
| Ingestion & Orchestration | IP01–IP02, Kafka, retry, DLQ và replay |
| Data & ML | IP03–IP04–IP06, Delta, Feast và MLflow |
| Serving & Retrieval | IP05–IP07, Qdrant, grounding và vLLM |
| Platform & Observability | IP08–IP10, Envoy, metrics, traces và readiness |
| Presenter / Incident Commander | Evidence, sự cố, khôi phục, rollback và Q&A |

## Giải thích 10 điểm kết nối

| IP | Kết nối | Điều phải chứng minh |
|---|---|---|
| IP01 | FastAPI → Kafka | Event hợp lệ mang `idempotency-key` và `traceparent` |
| IP02 | Kafka → Airflow | Consumer đọc bản tin, retry có giới hạn và đưa lỗi vào DLQ |
| IP03 | Airflow/Spark → Delta | Replay không tăng số dòng; Delta history và time travel đọc được |
| IP04 | Delta → Feast | Snapshot được materialize và đọc đúng entity `asker_id` |
| IP05 | Delta → Qdrant | Document dùng ID ổn định, collection có điểm và search có kết quả |
| IP06 | Delta/evaluation → MLflow | Release có provenance, signature và alias `champion` |
| IP07 | FastAPI → vLLM | Endpoint thật trả version, model ID và metrics `vllm:` |
| IP08 | Client → Envoy → API | Route đúng, có request ID, health/readiness và phản hồi 429 |
| IP09 | Services → Prometheus/Grafana | Target được scrape, golden signals và alert có hành động xử lý |
| IP10 | Services → OTel/Jaeger | Một trace ID chứa đầy đủ span bắt buộc xuyên hệ thống |

## Các khái niệm chính

### Vì sao phải tách ingestion, storage, feature store, vector store và registry?

Mỗi vùng có contract, vòng đời và cách scale khác nhau. Kafka hấp thụ burst và
tách producer khỏi consumer; Delta giữ lịch sử giao dịch và hỗ trợ replay;
Feast phục vụ feature theo entity với độ trễ thấp; Qdrant tối ưu tìm kiếm vector;
MLflow quản lý phiên bản, provenance, promotion và rollback. Việc tách rời làm
giảm coupling, xác định rõ owner và cho phép khôi phục từng boundary.

### Idempotency và replay-safe

`idempotency_key` biểu diễn một thao tác logic. Khi Kafka giao lại cùng dữ liệu,
`dedupe_latest` chỉ giữ một event cho mỗi khóa. Cặp `(occurred_at, event_id)`
chọn bản mới nhất và xử lý ổn định trường hợp trùng timestamp. Delta MERGE dùng
khóa này để update hoặc insert thay vì append vô điều kiện.

### Health và readiness

Health chỉ xác nhận tiến trình còn sống. Readiness xác nhận tiến trình có đủ
dependency để nhận traffic. Lỗi dependency bắt buộc tạo `not_ready`; lỗi
dependency tùy chọn tạo `degraded`; mọi probe đạt tạo `ready`.

### Metrics và traces

Metrics cho biết phạm vi và xu hướng sự cố qua rate, error, duration, saturation
và Kafka lag. Trace cho biết một request cụ thể chậm hoặc lỗi tại span nào.
Metrics dùng để phát hiện và khoanh vùng thời gian; trace dùng để truy nguyên
đường đi chi tiết trong khoảng thời gian đó.

## Năm hành trình tích hợp

1. J1 chứng minh happy path từ gateway đến Kafka, Airflow, Delta, Feast/Qdrant,
   MLflow, vLLM và observability.
2. J2 gửi lại cùng batch và xác nhận số dòng Delta cùng số điểm Qdrant không tăng.
3. J3 tạo release mới, đổi alias `champion`, kiểm tra hành vi rồi rollback alias.
4. J4 dừng dependency tùy chọn, quan sát `degraded`, khởi động lại và xác nhận
   không mất dữ liệu.
5. J5 đối chiếu request ID, trace ID, metrics và readiness trên cùng một lần chạy.

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

Kafka và Airflow tăng độ bền cùng khả năng replay nhưng làm tăng độ trễ và số
trạng thái cần vận hành. Delta MERGE bảo đảm idempotency tốt hơn append nhưng có
chi phí đọc transaction log và compact file. Alias MLflow giúp rollback không
cần sửa mã nhưng yêu cầu provenance đầy đủ để tránh trỏ nhầm release.

## Sự cố và khôi phục được chọn

Kịch bản demo chọn Feast vì đây là dependency tùy chọn của serving path:

1. Trước sự cố: ghi readiness, trace ID, số dòng Delta và kết quả Feast.
2. Dừng Feast bằng `docker compose stop feast`.
3. Dự đoán: API vẫn sống, readiness chuyển `degraded`, feature lookup lỗi nhưng
   đường trả lời còn hoạt động theo degraded policy.
4. Quan sát log, metric lỗi/độ trễ và trace span Feast.
5. Khôi phục bằng `docker compose start feast`, chờ health đạt rồi materialize
   lại nếu cần.
6. Chứng minh `ready`, entity đọc lại được và số dòng Delta không giảm.

Không dùng `down -v` vì thao tác đó xóa state và không còn là recovery.

## Promotion và rollback

Release MLflow phải gắn prompt version, model ID, embedding model ID, Qdrant
collection, feature service, `top_k`, Delta version và Git SHA. Alias
`champion` được chuyển sang release mới để thử nghiệm. Rollback chỉ chuyển alias
về version trước, sau đó xác minh API resolve đúng version mà không sửa source.

## Khoảng trống trước production

- Chạy đủ J1–J5 trên hạ tầng thật và lưu bằng chứng có thể đối chiếu run ID,
  trace ID, Delta version và MLflow version.
- Bổ sung TLS/mTLS, quản lý secret tập trung, xác thực và phân quyền.
- Bổ sung replication, backup/restore, quota và kiểm thử disaster recovery cho
  Kafka, Delta, Qdrant, MLflow cùng hệ thống telemetry.
- Xây dựng SLO, error budget, kiểm thử tải dài hạn và capacity planning.
- Xác minh IP07 với endpoint vLLM GPU thật; không dùng server giả lập API.

## Cải tiến tiếp theo

Hướng cải tiến tiếp theo là bổ sung transactional outbox cho API → Kafka,
chaos test cho Kafka/Airflow/Delta và tự động đối chiếu cùng trace ID giữa
Jaeger, Prometheus, Delta và MLflow trong evidence bundle.

## Trạng thái xác minh

- Fast suite: `87 passed` trên source có SHA-256
  `61c9c67a1db1c648409ff6eb263ca7ca41564d67637470a346179ac398ef8585`.
- Ruff, integration matrix, portability và manifest validation: đạt.
- Docker basic profile: đã xác minh topics, seed, Qdrant index, MLflow champion
  và readiness `degraded` hợp lệ do thiếu vLLM.
- Docker full profile: `PARTIAL`; Spark Connect từng bị OOM với Docker RAM
  4.8 GiB và lần cuối Airflow chưa qua trạng thái `Waiting`.
- J1–J5, load profile, Delta/Feast live evidence và vLLM GPU: `UNVERIFIED`;
  không tạo evidence giả.
