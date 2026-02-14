---
doc_role: baseline
module: grafana-otel-serilogs
kind: layer
status: active
last_updated: 2026-02-15
owners: [backend-team]
---

# Observability Stack – Introduction

## 1. Mục tiêu

Tài liệu này mô tả tổng quan về hệ thống Observability cho backend ASP.NET trong dự án, nhằm tạo "bộ nhớ ngoài" cho AI agent và lập trình viên khi tiếp cận hệ thống.

Hệ thống được thiết kế để theo dõi đầy đủ:

* Logs (Nhật ký hệ thống)
* Traces (Luồng xử lý request)
* Metrics (Chỉ số hiệu năng)

Mục tiêu chính:

* Hỗ trợ debug nhanh
* Phân tích hiệu năng
* Phát hiện lỗi sớm
* Demo và bảo vệ đồ án
* Xây dựng tư duy production-ready

---

## 2. Phạm vi áp dụng

Stack observability này áp dụng cho:

* Backend ASP.NET Core (.NET 6/7/8)
* Chạy bằng Docker / Docker Compose
* Môi trường: Dev, Demo, Staging

Không tập trung vào triển khai enterprise-scale production.

---

## 3. Tổng quan kiến trúc

Kiến trúc observability theo mô hình cloud-native:

```
ASP.NET API
 ├─ Serilog ─────────▶ Loki
 │      └─ SerilogUI (Dev)
 │
 └─ OpenTelemetry ─▶ OTel Collector
                     ├─▶ Tempo (Traces)
                     └─▶ Prometheus (Metrics)

Grafana ◀──────────── Loki / Tempo / Prometheus
```

Mỗi thành phần đảm nhiệm một vai trò riêng biệt, không chồng chéo trách nhiệm.

---

## 4. Danh sách công nghệ

### 4.1 Logging

* Serilog: Structured logging engine
* SerilogUI: Web UI xem log trong môi trường dev
* Loki: Centralized log storage

### 4.2 Tracing

* OpenTelemetry SDK (NuGet Package)
* OpenTelemetry Collector
* Tempo

### 4.3 Metrics

* OpenTelemetry SDK (NuGet Package)
* Prometheus

### 4.4 Visualization

* Grafana

---

## 5. Vai trò từng thành phần

### 5.1 Serilog

* Sinh log có cấu trúc (structured logs)
* Gắn metadata (TraceId, MachineName, ThreadId,...)
* Gửi log tới nhiều đích (Console, Loki, File)

### 5.2 SerilogUI

* Công cụ debug nội bộ
* Chỉ dùng trong môi trường dev
* Không sử dụng cho production

### 5.3 OpenTelemetry NuGet Package

* Instrument ASP.NET Core
* Sinh trace và metrics tự động
* Gửi dữ liệu qua OTLP protocol

### 5.4 OpenTelemetry Collector

* Trung gian thu thập dữ liệu
* Gom batch, retry, transform
* Forward dữ liệu tới backend lưu trữ

### 5.5 Loki

* Lưu trữ log theo mô hình label-based
* Hỗ trợ truy vấn bằng LogQL

### 5.6 Tempo

* Lưu trữ distributed traces
* Phục vụ phân tích latency và dependency

### 5.7 Prometheus

* Lưu trữ time-series metrics
* Hỗ trợ alerting

### 5.8 Grafana

* Dashboard trung tâm
* Hiển thị Logs, Traces, Metrics
* Hỗ trợ correlation giữa các nguồn dữ liệu

---

## 6. Nguyên tắc thiết kế

1. Separation of Concerns

   * Logging, Tracing, Metrics độc lập
   * Không dùng một công cụ cho nhiều vai trò

2. Vendor Neutral

   * Dựa trên OpenTelemetry
   * Dễ thay thế backend trong tương lai

3. Centralized Observability

   * Dữ liệu tập trung tại Loki / Tempo / Prometheus

4. Production Mindset

   * Thiết kế theo mô hình thực tế trong ngành
   * Không chỉ phục vụ mục đích học tập

---

## 7. Danh sách container khi triển khai Docker Compose

Khi triển khai hệ thống observability bằng Docker Compose, hệ thống tối thiểu sẽ bao gồm các container sau:

| Tên container  | Vai trò                 | Ghi chú                      |
| -------------- | ----------------------- | ---------------------------- |
| api            | Backend ASP.NET         | Chạy ứng dụng chính          |
| otel-collector | OpenTelemetry Collector | Nhận và forward trace/metric |
| loki           | Log storage             | Lưu trữ log tập trung        |
| tempo          | Trace storage           | Lưu distributed traces       |
| prometheus     | Metrics storage         | Lưu time-series metrics      |
| grafana        | Visualization           | Dashboard trung tâm          |

Tổng số container bắt buộc: **6 + 1 (API)**.

---

## 8. Phạm vi không bao gồm

Stack này không tập trung vào:

* Log analytics nâng cao (SIEM)
* Enterprise APM độc quyền
* Big data pipeline
* Long-term cold storage

---

## 9. Tài liệu liên quan trong Bộ 5 Docs

* observability.plan.md
* observability.prompt.md
* observability.sourcecode.md
* observability.usageguide.md

Các tài liệu trên phải được cập nhật đồng bộ khi hệ thống thay đổi.

---

## 10. Nguyên tắc bảo trì tài liệu

* Khi thay đổi kiến trúc → cập nhật file này
* Khi thay đổi code → cập nhật sourcecode.md
* Khi thay đổi API → cập nhật usageguide.md
* Không để tài liệu lệch so với thực tế

---

## 11. Tóm tắt nhanh (Load Context)

Hệ thống observability sử dụng:

* Logging: Serilog → Loki
* Tracing: OpenTelemetry → Tempo
* Metrics: OpenTelemetry → Prometheus
* Visualization: Grafana
* Dev tool: SerilogUI

Mục tiêu: xây dựng hệ thống backend ASP.NET có khả năng quan sát đầy đủ theo chuẩn cloud-native.