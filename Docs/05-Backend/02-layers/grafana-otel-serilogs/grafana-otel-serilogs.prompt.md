# Observability Stack – Implementation Prompt

> **SYSTEM INSTRUCTION**: This prompt contains the exact instructions to implement the Observability Stack (Serilog, OpenTelemetry, Grafana, Loki, Tempo, Prometheus). Execute these steps sequentially.

## Context
We are implementing a full observability stack for the SnakeAid Backend. The goal is to collect Logs (via Serilog → Loki), Traces (OpenTelemetry → Tempo), and Metrics (OpenTelemetry → Prometheus), all visualized in Grafana. The backend is an ASP.NET Core API running in Docker Compose.

---

## Step 1: Infrastructure Setup (Docker Compose)

### 1. Create Configuration Files
Create a new folder `observability/` at the solution root (`d:\SourceCode\Snake_AID\SnakeAid.Backend\observability\`) and populate it with the following configuration files:

#### `observability/otel-collector-config.yaml`
```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  batch:

exporters:
  otlp:
    endpoint: tempo:4317
    tls:
      insecure: true
  prometheus:
    endpoint: "0.0.0.0:8889"
    namespace: "snakeaid_api"

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp] # To Tempo (via OTLP)
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus] # Expose via Prometheus scrape endpoint
```

#### `observability/prometheus.yaml`
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'otel-collector'
    scrape_interval: 10s
    static_configs:
      - targets: ['otel-collector:8889']
```

#### `observability/loki-config.yaml`
```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093
```

#### `observability/tempo-config.yaml`
```yaml
server:
  http_listen_port: 3200

distributor:
  receivers:
    otlp:
      protocols:
        grpc:
        http:

ingester:
  trace_idle_period: 10s
  max_block_bytes: 1_000_000
  max_block_duration: 5m

compactor:
  compaction:
    compaction_window: 1h
    max_block_bytes: 100_000_000
    block_retention: 1h
    compacted_block_retention: 10m

storage:
  trace:
    backend: local
    wal:
      path: /var/tempo/wal
    local:
      path: /var/tempo/blocks

overrides:
  metrics_generator_processors: [service-graphs, span-metrics]
```

#### `observability/grafana-datasources.yaml`
(Provision Loki, Tempo, Prometheus automatically)
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: false
    jsonData:
      graphiteVersion: "1.1"

  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: true # Make logs default for exploration

  - name: Tempo
    type: tempo
    access: proxy
    url: http://tempo:3200
    jsonData:
      tracesToLogs:
        datasourceUid: 'Loki'
        tags: ['job', 'instance', 'pod', 'namespace']
        mappedTags: [{ key: 'service.name', value: 'app' }]
        mapTagNamesEnabled: false
        spanStartTimeShift: '0'
        spanEndTimeShift: '0'
        filterByTraceID: true
        filterBySpanID: true
      tracesToMetrics:
        datasourceUid: 'Prometheus'
        tags: [{ key: 'service.name', value: 'service' }, { key: 'job' }]
        queries:
          - name: 'Sample query'
            query: 'sum(rate(traces_spanmetrics_latency_bucket{$$__tags}[5m]))'
```

### 2. Update `docker-compose.yml`
Add the following services to `d:\SourceCode\Snake_AID\SnakeAid.Backend\docker-compose.yml`:

```yaml
  # ... existing snakeaid-api service ...

  loki:
    image: grafana/loki:latest
    command: -config.file=/etc/loki/local-config.yaml
    ports:
      - "3100:3100"
    volumes:
      - ./observability/loki-config.yaml:/etc/loki/local-config.yaml

  tempo:
    image: grafana/tempo:latest
    command: [ "-config.file=/etc/tempo.yaml" ]
    volumes:
      - ./observability/tempo-config.yaml:/etc/tempo.yaml
      - ./observability/tempo-data:/var/tempo
    ports:
      - "14268"  # jaeger ingest
      - "3200"   # tempo
      - "4317"  # otlp grpc
      - "4318"  # otlp http
      - "9411"   # zipkin

  prometheus:
    image: prom/prometheus:latest
    command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --web.enable-lifecycle
    volumes:
      - ./observability/prometheus.yaml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - ./observability/grafana-datasources.yaml:/etc/grafana/provisioning/datasources/datasources.yaml
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
      - GF_AUTH_DISABLE_LOGIN_FORM=true

  otel-collector:
    image: otel/opentelemetry-collector:latest
    command: ["--config=/etc/otel-collector-config.yaml"]
    volumes:
      - ./observability/otel-collector-config.yaml:/etc/otel-collector-config.yaml
    ports:
      - "1888:1888"   # pprof extension
      - "8888:8888"   # Prometheus metrics exposed by the collector
      - "8889:8889"   # Prometheus exporter metrics
      - "13133:13133" # health_check extension
      - "4317:4317"   # OTLP gRPC receiver
      - "4318:4318"   # OTLP http receiver
      - "55679:55679" # zpages extension
    depends_on:
      - loki
      - tempo
      - prometheus
```

Also, update `snakeaid-api` to use the `otel-collector` endpoint (or configure it via env vars later).

---

## Step 2: Implementation (ASP.NET Core)

### 1. Install NuGet Packages
Run the following commands in `d:\SourceCode\Snake_AID\SnakeAid.Backend\SnakeAid.Api`:
```bash
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Instrumentation.Http
dotnet add package OpenTelemetry.Instrumentation.SqlClient # Or Npgsql if using Postgres
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
dotnet add package Serilog.Sinks.Grafana.Loki
```

### 2. Update `DependencyInjection.cs`
In `d:\SourceCode\Snake_AID\SnakeAid.Backend\SnakeAid.Api\DI\DependencyInjection.cs`:

Add a new method `AddOpenTelemetryServices`:
```csharp
public static IServiceCollection AddOpenTelemetryServices(this IServiceCollection services, IConfiguration configuration)
{
    var otlpEndpoint = configuration["OpenTelemetry:Endpoint"] ?? "http://localhost:4317"; // fallback for local without docker

    services.AddOpenTelemetry()
        .WithTracing(tracerProviderBuilder =>
        {
            tracerProviderBuilder
                .AddSource("SnakeAid.Api")
                .SetResourceBuilder(
                    OpenTelemetry.Resources.ResourceBuilder.CreateDefault()
                        .AddService("SnakeAid.Api"))
                .AddAspNetCoreInstrumentation()
                .AddHttpClientInstrumentation()
                .AddSqlClientInstrumentation(options => options.SetDbStatementForText = true) // or AddNpgsql()
                .AddOtlpExporter(opts => opts.Endpoint = new Uri(otlpEndpoint));
        })
        .WithMetrics(metricsProviderBuilder =>
        {
            metricsProviderBuilder
                .AddMeter("SnakeAid.Api")
                .AddAspNetCoreInstrumentation()
                .AddRuntimeInstrumentation()
                .AddOtlpExporter(opts => opts.Endpoint = new Uri(otlpEndpoint));
        });

    return services;
}
```

### 3. Update `Program.cs`
In `d:\SourceCode\Snake_AID\SnakeAid.Backend\SnakeAid.Api\Program.cs`:

1.  Call `builder.Services.AddOpenTelemetryServices(builder.Configuration);` inside the service registration block.
2.  Update Serilog configuration to include Loki sink IF configured:
    ```csharp
    var lokiUrl = builder.Configuration["Serilog:WriteTo:Loki:Args:requestUri"];
    if (!string.IsNullOrEmpty(lokiUrl)) 
    {
         // Add Loki sink via code OR rely on appsettings.json "WriteTo" section if using Serilog.Settings.Configuration
    }
    ```
    *Better approach*: Since `ReadFrom.Configuration` is used, we just need to update `appsettings.json`.

### 4. Update `appsettings.json`
Add the following configuration:

```json
  "OpenTelemetry": {
    "Endpoint": "http://otel-collector:4317"
  },
  "Serilog": {
    "WriteTo": [
      { "Name": "Console" },
      { 
        "Name": "GrafanaLoki", 
        "Args": { 
            "requestUri": "http://loki:3100" ,
            "labels": [
                { "key": "app", "value": "SnakeAid.Api" }
            ]
        } 
      }
    ]
  }
```

---

## Step 3: Verification
1.  Run `docker-compose up -d --build`.
2.  Check logs: `docker-compose logs -f snakeaid-api`.
3.  Open Grafana: `http://localhost:3000`.
4.  Navigate to **Explore**, select **Loki**, run query `{app="SnakeAid.Api"}`.
5.  Select **Tempo**, search for traces.
6.  Select **Prometheus**, query `http_requests_total`.
