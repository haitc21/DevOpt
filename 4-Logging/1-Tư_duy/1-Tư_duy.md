# Tư duy về Logging

## 1. Tư duy chọn giải pháp

![](./images/1.png)

![](./images/2.png)

![](./images/3.png)

![](./images/4.png)

## 2. Các loại log

![](./images/5.png)

![](./images/6.png)

![](./images/7.png)

![](./images/8.png)

![](./images/10.png)

![](./images/11.png)

![](./images/12.png)

## 3. Tư duy thiết kế hệ thống Logging

- Chuyển các Log dạng thô (Chuỗi) sang có cấu trục (JSON, CSV...)

![](./images/13.png)

![](./images/14.png)

![](./images/15.png)

- Lgo phải gắn với Context cụ thể. Ví dụ có trace_id sẽ truy vấn được luồng hoạt động của 1 request.

![](./images/16.png)

![](./images/17.png)

- Log Level

![](./images/18.png)

![](./images/19.png)

![](./images/20.png)

- Không lưu log dữ liệu nhạy cảm

![](./images/21.png)

![](./images/22.png)

- Log phải dễ truy vấn và lưu trữ lâu dài

![](./images/23.png)

![](./images/24.png)

## 4. Các thành phần hệ thống Logging tập trung

![](./images/26.png)

![](./images/27.png)

![](./images/28.png)

![](./images/29.png)

![](./images/30.png)

![](./images/31.png)

![](./images/32.png)

![](./images/33.png)

![](./images/34.png)

## 5. Lưu chọn công nghệ

### 5.1. Các Stack công nghệ phổ biến

![](./images/35.png)

![](./images/36.png)

- Promtail: Là 1 Agent thu thập log từ các nguồn, sau đó gửi đến Loki.
- Loki: Hệ thống tổng hợp, lưu trữ Log.
- Grafana: Truy vấn, trực quan hóa.

![](./images/37.png)

- Fluentd/FluentBit: Agent thu thập log. Fluentd viết bằng Ruby, FluentBit viết bằng C nhẹ hơn Fluentd.
- Elastic Search: Lưu trữ Log.
- Kibana: Trực quan hóa.

![](./images/38.png)

- beats: Agent thu thập log.
- Logstash: Tổng hợp, làm giàu, xử lý log. Bộ công cụ này rất phổ biến, tuy nhiên công cụ này khá tốn tài nguyên.

![](./images/39.png)

- Vector: Agent vừa có thể thu thập vừa có thể tổng hợp, làm giàu, chuẩn hóa log. Vector là công cụ mạnh mẽ, viết bằng Rust nên perf rất cao, làm được 3 việc:
  - Source: Thu thập log.
  - Tranfrom: Tổng hợp, làm giàu, chuẩn hóa v.v.
  - Sink: Gửi log đến cá nơi lưu trữ như Elastic Search, S3 ...

![](./images/40.png)

Ngoài ra các hệ thống log còn thêm 1 lớp đệm (buffer) như **Kafka**, Redis, RabitMQ.

### 5.2. Phân tích lựa chọn Stack công nghệ phù hợp

#### 5.2.1. Logging Storage

| **Tiêu chí**              | **Ưu tiên Elasticsearch**                                                       | **Ưu tiên Loki**                                                                     | **Ưu tiên ClickHouse**                                                                                              |
| ------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| **Đặc tính dữ liệu log**  | Log văn bản tự do kèm fields có cấu trúc; cần full-text search; schema-on-write | Log theo thời gian; metadata chủ yếu qua labels; payload giữ nguyên (schema-on-read) | Log dạng structured (columns) tối ưu analytics; không phù hợp full-text search nâng cao; schema-on-write            |
| **Mô hình truy vấn**      | Query DSL: full-text + filter/aggregation đa chiều                              | LogQL: filter theo labels, regex, query theo thời gian                               | SQL đầy đủ + aggregation cực nhanh; hỗ trợ vector index, TTL; phân tích log lớn theo thời gian/labels/fields        |
| **Tốc độ ghi & lưu giữ**  | Write trung bình → cao; index tốn tài nguyên; retention theo tiered storage     | Write rất cao, retention lâu với object storage (S3/GCS)                             | Write cực cao (hàng triệu sự kiện/s); lưu lượng TB–PB/tháng; TTL, compression mạnh; schema thay đổi khó hơn         |
| **Hệ sinh thái tích hợp** | Kibana, Beats, APM, SIEM… phong phú                                             | Grafana + Tempo/OTel; correlation logs-metrics-traces                                | Tích hợp tốt với Vector, Grafana, Kafka, OTel; nhiều hệ thống observability dùng làm backend log/metric             |
| **Vận hành & chi phí**    | Tốn chi phí index, yêu cầu IOPS cao, JVM phức tạp                               | Rẻ hơn nhờ kiến trúc stateless + object storage                                      | Khả năng mở rộng tuyến tính, chi phí/GB thấp; quản lý cluster cần chuyên môn; tối ưu phân vùng/partition quan trọng |

| **Công cụ**    | **Bản chất/kiến trúc**                                                                                | **Điểm mạnh**                                                                                                                    | **Hạn chế**                                                                                                   | **Khi dùng**                                                                                                                     |
| -------------- | ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Elasticsearch  | Document store dùng Lucene (inverted index) + doc_values; mapping/schema-on-write                     | Tìm kiếm full-text mạnh; aggregations đa chiều; Query DSL giàu; hệ sinh thái Elastic/Kibana/ES                                   | Chi phí ghi & lưu trữ cao (tokenize, build index, segment merge); JVM/GC; quản trị shard/ILM phức tạp         | Điều tra chuyên sâu, audit/security; phân tích nhiều chiều; cần truy vấn kết hợp text + field                                    |
| Loki           | Chỉ index labels (metadata); nội dung log nén thành chunks trên object storage                        | Chi phí rẻ; lưu trữ lâu dài; scale ngang đơn giản; tích hợp Grafana/metrics/traces                                               | Tìm kiếm theo nội dung phải quét chunk (full-text chậm); nhạy cảm khi label cardinality cao                   | Khối lượng log lớn, retention dài; truy vấn theo nhãn/service/env và lọc theo dòng                                               |
| **ClickHouse** | Column-oriented DB; storage phân vùng theo thời gian; schema-on-write; nén mạnh; vectorized execution | Ghi & truy vấn cực nhanh ở quy mô TB–PB; analytics thời gian thực; chi phí/GB thấp; TTL/partition giúp quản lý retention tự động | Không tối ưu full-text search phức tạp; phải thiết kế schema/partition chuẩn; thay đổi schema có thể phức tạp | Observability analytics lớn, dashboard real-time; log structured từ Kafka/OTel; truy vấn thống kê theo thời gian/fields hiệu quả |

Bảng chú thích khái niệm

| Thuật ngữ                                    | Giải thích                                                                       |
| -------------------------------------------- | -------------------------------------------------------------------------------- |
| **Document store**                           | Mô hình lưu trữ dạng tài liệu (như JSON), mỗi tài liệu có cấu trúc linh hoạt     |
| **Inverted index**                           | Chỉ mục đảo: dùng trong full-text search để ánh xạ từ khóa → tài liệu chứa từ đó |
| **doc_values**                               | Cách Elasticsearch lưu trữ dữ liệu dạng field để phục vụ aggregation nhanh       |
| **Schema-on-write**                          | Phải định nghĩa schema trước khi ghi dữ liệu (mapping cứng)                      |
| **Schema-on-read**                           | Dữ liệu ghi vào không ép buộc schema, chỉ parse khi đọc                          |
| **Column-oriented DB**                       | Cơ sở dữ liệu lưu theo cột thay vì hàng, tối ưu cho analytics                    |
| **Chunks**                                   | Các khối log được gom và nén lại trước khi lưu trên storage                      |
| **Labels (metadata)**                        | Metadata để định danh log: service, env, instance…                               |
| **High label cardinality**                   | Số lượng giá trị nhãn quá nhiều gây nổ index, hiệu năng giảm                     |
| **Full-text search**                         | Tìm kiếm theo nội dung văn bản (tokenization, phân tích ngôn ngữ)                |
| **Aggregation**                              | Tổng hợp dữ liệu đa chiều (count/avg/sum theo time/service…)                     |
| **Query DSL**                                | Ngôn ngữ truy vấn riêng của Elasticsearch                                        |
| **Vectorized execution**                     | Cách thực thi truy vấn xử lý theo batch giúp tốc độ cao (ClickHouse)             |
| **Compactor**                                | Thành phần nén/gom dữ liệu (Loki)                                                |
| **ILM (Index Lifecycle Management**          | Quản lý vòng đời index (hot-warm-cold-delete)                                    |
| **Segment merge**                            | Quá trình hợp nhất chỉ mục trong Elasticsearch, tốn CPU/IO                       |
| **TTL (Time-to-Live)**                       | Tự động xóa dữ liệu sau thời gian cấu hình                                       |
| **Retention**                                | Thời gian giữ log trước khi xóa hoặc archive                                     |
| **Hot/Warm/Cold tiers**                      | Phân tầng storage theo tốc độ/chi phí truy cập                                   |
| **APM (Application Performance Monitoring)** | Theo dõi hiệu năng ứng dụng (latency, transactions…)                             |
| **SIEM (Security Info & Event Management)**  | Phân tích bảo mật và log audit tập trung                                         |
| **OTel / OpenTelemetry**                     | Open standard cho logs / metrics / traces                                        |
| **Correlated logs-metrics-traces**           | Khả năng kết hợp dữ liệu từ 3 loại quan sát để phân tích root cause              |
| **JVM / GC**                                 | Java Virtual Machine & Garbage Collector – ảnh hưởng lớn hiệu năng ES            |
| **Shard**                                    | Đơn vị phân mảnh dữ liệu trong Elasticsearch để scale                            |
| **Object storage (S3/GCS)**                  | Lưu log chi phí thấp với cloud storage mở rộng tuyến tính                        |
| **IOPS**                                     | Số lượng thao tác I/O mỗi giây – ảnh hưởng tốc độ query/indexing                 |

#### 5.2.2. Agent/processor

![](./images/41.png)

#### 5.2.3. uSE cASE SỬ DỤNG

| **Use case**                                                       | **Đặc điểm chính**                                                                                                            | **Stack khuyến nghị**                                          | **Lưu ý triển khai**                                                                                |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| **E-commerce microservices**                                       | Truy vấn lỗi theo order_id/trace_id; cần full-text search đa trường; correlation chuỗi sự kiện; dashboard phân tích nghiệp vụ | **EFK/ELK (Elasticsearch)** + Vector/Fluent Bit                | Chuẩn hóa field (trace_id, order_id, user_id, service, env, region…); ILM hot-warm-cold; bảo vệ PII |
| **Nền tảng nội bộ (TB/ngày)**                                      | Hàng trăm service tạo log lớn mỗi ngày; chủ yếu truy vấn theo labels/time; retention 3–6 tháng; alert theo mẫu bất thường     | **Loki** + Promtail/Vector/Fluent Bit                          | Quy ước labels (cluster, namespace, app, env…); tránh high cardinality (request_id)                 |
| **Tuân thủ / Audit / Security**                                    | Truy vấn phức tạp; yêu cầu timeline chi tiết; SIEM; anomaly detection                                                         | **Elasticsearch / OpenSearch** + Vector/Fluent Bit             | Mask PII; ILM chặt; chi phí cao → cần quota/archival hợp lý                                         |
| **Log phân tích kinh doanh (Analytics)**           | Tính toán churn, errors theo user/device/version; truy vấn aggregation lớn theo time-series                                   | **ClickHouse** + Vector/Kafka + Grafana                        | Thiết kế schema/partition theo thời gian; JSON phải parse trước; index sparse                       |
| **Observability hợp nhất Logs + Metrics + Traces** | Root cause analysis theo thời gian thực; correlation đa chiều                                                                 | **Loki + Prometheus/Mimir + Tempo** hoặc **ClickHouse + OTel** | Đồng bộ trace_id trong cả logs/metrics/traces; storage S3 để tối ưu chi phí                         |
| **Ứng dụng AI/ML tạo log khổng lồ** ⭐ *bổ sung mới*                | Training/inference tạo PB log; cần query theo feature/experiment                                                              | **ClickHouse**                                                 | Batch ingest Kafka; TTL linh hoạt theo lifecycle của model                                          |
| **ETL / Data pipeline logs** ⭐ *bổ sung mới*                       | Log nhiều field structured; cần truy vấn performance, thời gian xử lý                                                         | **Elasticsearch** hoặc **ClickHouse**                          | Dễ bị high-cardinality → cần chuẩn hóa schema log pipeline                                          |

TỔNG KẾT

| Mô hình                                     | Ưu tiên              |
| ------------------------------------------- | -------------------- |
| Full-text search + security + investigation | Elasticsearch        |
| Logs volume lớn + chi phí thấp              | Loki                 |
| Analytics log + dashboard real-time         | ClickHouse           |
| Observability 3-in-1 (logs-metrics-traces)  | Loki hoặc ClickHouse |
| Compliance + SIEM                           | Elasticsearch        |
