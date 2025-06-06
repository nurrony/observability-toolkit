<div align="center">
  <img src="./docs/logo.png" alt="Observability Toolkit"/><BR>
  <img src="https://goto.ceesbos.nl/badge/github/cbos/observability-toolkit" alt="Badge"/>
</div>

# Observability Toolkit for local usage
A set of tools which can be used for local development which help with more insights in Observability.

![](docs/setup.png)

# Background

To get to know more about the background about this setup, please read: https://ceesbos.nl/posts/20240126-observability-toolkit/

# Explanation of the components

## OpenTelemetry Collector
[OpenTelemetry Collector](https://opentelemetry.io/docs/collector/) is the main entry point where all observability signals are collected.
With all kind of configuration you can choose what to do with it.

OpenTelemetry Collector sends the data to:
- [Loki](https://github.com/grafana/loki)   
  Loki is a log storage created by Grafana
- [Tempo](https://github.com/grafana/tempo)   
  Tempo is a trace storage created by Grafana
- [Prometheus](https://github.com/prometheus/prometheus)   
  Prometheus is a metric storage

## Grafana
[Grafana](https://github.com/grafana/grafana) is the visualisation tool to get insights in all observability metrics collected and stored.

# Power of correlation
The power of combining metrics, logs and traces together in 1 setup is that you can correlate these signals.    
![](docs/correlation_between_signals.png)    
The advantage of OpenTelemetry is standardisation on naming, that makes it way easier to hop between the metrics, traces and logs in all directions.
That will help to find the problems.

Configuration in Grafana can help to make it easier to hop from metrics (with exemplars) to traces for example (see [prometheus.yaml](./config/grafana/provisioning/datasources/prometheus.yaml)).    
That is already configured in this setup. Same holds from logs to traces (see [loki.yaml](./config/grafana/provisioning/datasources/loki.yaml)).   
Also the setup of spanmetrics and servicegraph is already configured (see [tempo.yaml](./config/grafana/provisioning/datasources/tempo.yaml)).   

# Start the setup

```shell
git clone https://github.com/cbos/observability-toolkit
cd observability-toolkit
docker compose up -d  

# If you have just command runner installed you can run:
just up
```
Now you can open http://localhost:3000 to open Grafana.

# Run the demo app and generate load
There is a simple demo app, which helps to generate some traces, logs and metrics, but is not intended to show all capabilities.    


```shell
./run-observabilty-demo-app.sh 
```

To generate load:
```shell
./loadgen.sh 
```

## OpenTelemetry Demo
You can use the [demo services of OpenTelemetry](https://opentelemetry.io/docs/demo/) to see what is all available for all languages.
You can change the file: `src/otelcollector/otelcol-config-extras.yml` and add this:

```yaml
exporters:
  otlp/observabilitytoolkit:
   # This assumes that the setup is running at the docker host on port 4317 (which is default of observability-toolkit)
   endpoint: "host.docker.internal:4317"
   tls:
    insecure: true

service:
  pipelines:
   traces:
    receivers: [otlp]
    processors: [batch]
    exporters: [otlp, otlp/observabilitytoolkit]
   metrics:
    receivers: [httpcheck/frontendproxy, redis, otlp]
    processors: [batch]
    exporters: [otlphttp/prometheus, otlp/observabilitytoolkit]
   logs:
    receivers: [otlp]
    processors: [batch]
    exporters: [otlp/observabilitytoolkit]
```

# Default dashboards
- http://localhost:3000/d/observabilitystackOtelCollector/opentelemetry-collector
![](docs/opentelemetry_collector_dashboard.png)

# Settings 

An number of settings can be tweaked by just setting environment variables.

```shell
# Specify the Grafana host port you want, other then the default 3000
export GRAFANA_HOST_PORT=3004

# Start the stack (in the background with -d)
docker compose up -d 

# Now you can open Grafana at http://localhost:3004
```
See [.env](.env) for actual values, below are the descriptions

| Variable name                        | Description                                                                                                             |
|--------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| LOKI_IMAGE_NAME                      | Loki docker image                                                                                                       |
| TEMPO_IMAGE_NAME                     | Tempo docker image                                                                                                      |
| PROMETHEUS_IMAGE_NAME                | Prometheus docker image                                                                                                 |
| GRAFANA_IMAGE_NAME                   | Grafana docker image                                                                                                    |
| GRAFANA_HOST_PORT                    | Port on host on which Grafana will be available                                                                         |
| OTEL_COLLECTOR_IMAGE_NAME            | OpenTelemetry Collector docker image                                                                                    |
| OTEL_COLLECTOR_HOST_PORT_GRPC        | Port on host on which OpenTelemetry Collector will be available for OTLP format with GRPC                               |
| OTEL_COLLECTOR_HOST_PORT_HTTP        | Port on host on which OpenTelemetry Collector will be available for OTLP format with HTTP                               |
| OTEL_COLLECTOR_HOST_PORT_PROMETHEUS  | Port on host on which OpenTelemetry Collector will listen to expose prometheus data, like http://localhost:8889/metrics |
| PROMTAIL_IMAGE_NAME                  | Promtail docker image                                                                                                   |
| PYROSCOPE_IMAGE_NAME                 | Pyroscope docker image                                                                                                  |
| PYROSCOPE_PORT                       | Port on which Pyroscope is available                                                                                    |


# Inspired by 

This setup is created based on what I already used locally.    
The idea of adding a simple application for showcasing the setup comes fromL [grafana/docker-otel-lgtm](https://github.com/grafana/docker-otel-lgtm)    
The script for generating random load with a simple curl command comes from the [Grafana Beyla project](https://github.com/grafana/beyla/blob/main/examples/greeting-apps/loadgen.sh)

The OpenTelemetry Collector Grafana Dashboard is coming from
https://grafana.com/grafana/dashboards/15983-opentelemetry-collector/    
But it required some changes to get it working in this setup.