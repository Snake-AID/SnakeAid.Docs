# Observability Stack – Implementation Plan

## 1. Goal Description

This plan outlines the steps to implement a full observability stack for the SnakeAid Backend. The goal is to enable centralized logging, distributed tracing, and metrics collection using OpenTelemetry, Serilog, and the Grafana ecosystem (Loki, Tempo, Prometheus).

## 2. Proposed Changes

### 2.1. Infrastructure (Docker Compose)
We need to add the observability services to `docker-compose.yml` and provide their configuration files.

#### [NEW] `docker-compose.yml` (Update)
- Add services: `loki`, `tempo`, `prometheus`, `grafana`, `otel-collector`.
- Configure volumes and networks.

#### [NEW] `observability/otel-collector-config.yaml`
- Configure receivers (OTLP).
- Configure exporters (Loki, Tempo, Prometheus).
- Configure processors (batch).

#### [NEW] `observability/prometheus.yaml`
- Configure scrape configs (if pulling from OTel) or rely on OTel pushing to Prometheus (remote write) or Prometheus scraping OTel.
- *Decision*: We will use OTel Collector to export metrics to Prometheus (via `prometheus` exporter on a specific port).

#### [NEW] `observability/loki-config.yaml`
- Basic Loki configuration.

#### [NEW] `observability/tempo-config.yaml`
- Basic Tempo configuration.

#### [NEW] `observability/grafana-datasources.yaml`
- Pre-configure Loki, Tempo, and Prometheus as data sources in Grafana.

### 2.2. Backend Logic (ASP.NET Core)

#### [MODIFY] `SnakeAid.Api/SnakeAid.Api.csproj`
- Add NuGet packages:
    - `OpenTelemetry.Extensions.Hosting`
    - `OpenTelemetry.Instrumentation.AspNetCore`
    - `OpenTelemetry.Instrumentation.Http`
    - `OpenTelemetry.Instrumentation.SqlClient` (or `Npgsql`)
    - `OpenTelemetry.Exporter.OpenTelemetryProtocol`
    - `Serilog.Sinks.Grafana.Loki`

#### [MODIFY] `SnakeAid.Api/DI/DependencyInjection.cs`
- Create `AddOpenTelemetryServices` extension method.
- Configure Tracing:
    - Add ASP.NET Core instrumentation.
    - Add HttpClient instrumentation.
    - Add EF Core / Npgsql instrumentation.
    - Add OTLP Exporter (endpoint from config).
- Configure Metrics:
    - Add Runtime metrics.
    - Add ASP.NET Core metrics.
    - Add OTLP Exporter.

#### [MODIFY] `SnakeAid.Api/Program.cs`
- Register `AddOpenTelemetryServices`.
- Configure Serilog to write to Loki (via `appsettings` or code). *Decision*: Use code configuration in `Program.cs` or `AddAppsettings` but read values from config.

#### [NEW] `SnakeAid.Api/appsettings.json` (and `appsettings.Development.json`)
- Add `OpenTelemetry` section (Endpoint).
- Add `Loki` section (Endpoint).

## 3. Verification Plan

### 3.1. Automated Verification
- **Build Check**: Run `dotnet build` to ensure no compile errors.
- **Container Check**: Run `docker-compose up -d` and verify all containers (`otel-collector`, `loki`, `tempo`, `prometheus`, `grafana`) are healthy.

### 3.2. Manual Verification
1.  **Generate Traffic**: Call Swagger endpoints (`GET /weatherforecast` or similar).
2.  **Check Grafana**:
    -   Access Grafana at `http://localhost:3000`.
    -   **Logs**: Explore Loki data source. Filter by `{app="SnakeAid.Api"}`. Verify logs appear.
    -   **Traces**: Explore Tempo data source. Find a trace ID from logs (if correlated) or search recent traces. Verify span visualization.
    -   **Metrics**: Explore Prometheus data source. Query `http_requests_total`. Verify metrics are increasing.
3.  **Check Correlation**:
    -   Verify logs contain `TraceId` and `SpanId`.
    -   Verify clicking a TraceId in logs opens Tempo view.
