# Grafana Stack on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/template/8TLSQD?referralCode=IFlm92)

## What is this template

This template deploys a complete Grafana observability stack on Railway with just one click! The stack includes four integrated services:

- **Grafana**: The leading open-source analytics and monitoring solution
- **Loki**: A horizontally-scalable, highly-available log aggregation system
- **Prometheus**: A powerful metrics collection and alerting system
- **Tempo**: A high-scale distributed tracing backend

This template is perfect for teams who need a comprehensive observability solution for their railway project without the hassle of manual configuration and infrastructure management.

### Key Features

- **Pre-configured Integration**: _All services come pre-connected_, so Grafana is ready to query your data immediately.
- **Persistent Storage**: All four services use Railway volumes to ensure your data, dashboards, and configurations persist between updates and deploys.
- **Version Control**: Pin specific Docker image versions for each service using environment variables.
- **Customizable**: Fork the repository to customize configuration files for any service. You can take full control and edit anything you'd need to as you scale.
- **One-Click Deploy**: Get a complete Grafana-based observability stack running in minutes.

## Quick Start Guide

1. Click the "Deploy on Railway" button at the top of this page
2. Enter your desired Grafana admin username in the `GF_SECURITY_ADMIN_USER` variable
3. Leave all other variables at their defaults (or customize as needed)
4. Wait for your stack to deploy (this typically takes 3-5 minutes)
5. Navigate to the Grafana URL provided by Railway
6. Log in with your admin username and the auto-generated password found in the `GF_SECURITY_ADMIN_PASSWORD` environment variable
7. Hook up your applications to the datasources.
8. Create dashboards, alerts, and explore your data in Grafana!

## Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `GF_SECURITY_ADMIN_USER` | Username for the Grafana admin account | Required input |
| `GF_SECURITY_ADMIN_PASSWORD` | Password for the Grafana admin account | Auto-generated secure string |
| `GF_DEFAULT_INSTANCE_NAME` | Name of your Grafana instance | `Grafana on Railway` |
| `GF_INSTALL_PLUGINS` | Comma-separated list of Grafana plugins to install | `marcusolsson-json-datasource,grafana-clock-panel` |
| `POLYBOT_JSON_URL` | Base URL for the Polybot JSON API datasource | `http://polybot:3000/api/status` |
| `POLYBOT_METRICS_TOKEN` | Bearer token sent as Authorization header to Polybot JSON API | `changeme` |
| `POLYBOT_SLACK_WEBHOOK_URL` | Incoming webhook URL used by the bundled Slack contact point | `https://hooks.slack.com/services/your/webhook/here` |
| `POLYBOT_SLACK_CHANNEL` | Slack destination (for example `#polybot-alerts`) for provisioned alerts | `#polybot-alerts` |

> Grafana 12+ no longer loads legacy Angular plugins, so the default `GF_INSTALL_PLUGINS` list ships only React-based plugins. If you still need Angular plugins (for example `grafana-worldmap-panel`), enable Grafana's Angular compatibility flag manually and add them back to the variable.

### Internal Service URLs

The Grafana service exposes these environment variables that you can reference in your other Railway applications to easily send data to your observability stack:

| Variable | Description | Usage |
|----------|-------------|-------|
| `LOKI_INTERNAL_URL` | Internal URL for the Loki service | Use in your applications to send logs to and query Loki |
| `PROMETHEUS_INTERNAL_URL` | Internal URL for the Prometheus service | Use in your applications to send metrics to and query Prometheus |
| `TEMPO_INTERNAL_URL` | Internal URL for the Tempo service | Use in your applications to query Tempo |

These variables make it easy to configure your other Railway services to send telemetry data to your observability stack.

Tempo also exposes a few variables to make it easier to push tracing information to the service using either HTTP or GRPC

| Variable | Description | Usage |
|----------|-------------|-------|
| `INTERNAL_HTTP_INGEST` | Internal HTTP ingest server URL for Tempo | Use in your applications to send traces to tempo via HTTP |
| `INTERNAL_GRPC_INGEST` | Internal GRPC ingest server URL for Tempo | Use in your applications to send traces to tempo via GRPC |

### Version Control

Each service has its own `VERSION` environment variable that can be set independently in each service's settings in the Railway dashboard:

- **Grafana Service**: Set `VERSION` to control the Grafana Docker image tag
- **Loki Service**: Set `VERSION` to control the Loki Docker image tag
- **Prometheus Service**: Set `VERSION` to control the Prometheus Docker image tag
- **Tempo Service**: Set `VERSION` to control the Tempo Docker image tag

By default, all services use the `latest` tag, but you can pin specific versions for stability:

Examples:
- Grafana: `VERSION=11.5.2`
- Loki: `VERSION=3.4.2`
- Prometheus: `VERSION=v3.2.1`
- Tempo: `VERSION=v2.7.1`

This allows you to update each component independently as needed.

## Project Structure & Services

This template deploys four interconnected services:

### PostgreSQL Datasource for Grafana
- Grafana now provisions a PostgreSQL datasource automatically. Provide credentials via the following environment variables on the Grafana service (Railway variables or `docker compose` overrides):
  - `POSTGRES_DS_HOST` (default `postgres:5432`)
  - `POSTGRES_DS_DATABASE` (default `postgres`)
  - `POSTGRES_DS_USER` (default `postgres`)
  - `POSTGRES_DS_PASSWORD` (default `postgres`)
  - `POSTGRES_DS_SSLMODE` (default `disable`)
- When running locally, copy `.env.example` to `.env` and adjust these values before starting the stack so `docker compose` passes them through to Grafana.
- Grafana uses these values during startup; update them before deploying so dashboards can query your database.

### Grafana
- The central visualization and dashboarding platform
- Pre-configured with connections to all other services
- Persistent volume for storing dashboards, users, and configurations
- Comes with useful plugins pre-installed
- Exposes internal URLs for other Railway services to connect to Loki, Prometheus, and Tempo

#### Embedded Alerting
Grafana now provisions unified alerting resources from `grafana/alerting`. Update `alerts.yml`, `contact-points.yml`, and `notification-policies.yml`, then rebuild the Grafana image (`docker compose up --build grafana`) to version alert rules, contact points, and routing with the rest of the stack. The bundled `Polybot Flow Error Detected` rule evaluates every two minutes, looks for the most recent `level = ERROR` row in the `FlowEvent` table within that window, and fires a single notification per interval. The emitted series encodes the flow, stage, `context.class`, and truncated error message inside the label payload so Slack posts include useful debugging hints without jumping into Grafana. A second `Polybot Trade Executed` rule runs on the same cadence, joins `TradeExecution` with `User` metadata, and emits a single data point when a trade lands in the last two minutes so you see celebratory alerts with user, handle, market, side, and notional amount baked into the Slack message.
All provisioned alerts route to the `polybot-slack` contact point, which pulls its channel + webhook from `${POLYBOT_SLACK_CHANNEL}` and `${POLYBOT_SLACK_WEBHOOK_URL}`. Override those variables (via `.env` locally or Railway variables in production) before redeploying to point alerts at the correct workspace destination.

### Prometheus
- Time-series database for metrics collection
- Configured with sensible defaults for monitoring
- Persistent volume for metrics data

### Loki
- Log aggregation system designed to be cost-effective
- Horizontally scalable architecture
- Persistent volume for log storage

### Tempo
- Distributed tracing system for tracking requests across services
- High-performance trace storage
- Persistent volume for trace data

All services are deployed using official Docker images and configured to work together seamlessly.

## Connecting Your Applications

### Using [Locomotive](https://railway.com/template/jP9r-f) for Loki

You can easily ingest *all* of your railway logs into Loki from *any* service using [Locomotive](https://railway.com/template/jP9r-f). Just spin up their template, drop in your Railway API key, the ID of the services you want to monitor, and a link to your new Loki instance and logs will start flowing! no code changes needed anywhere!

### Using OpenTelemetry libraries for Tempo 

Tempo is a bit different than both Prometheus and Loki in that exposes separate GRPC and HTTP servers on ports `:4317` and `:4318` respectively specifically for ingesting your tracing data or "spans".

When configuring your application to send traces to Tempo, please use one of the preconfigured variables in the Tempo service: `INTERNAL_HTTP_INGEST` or `INTERNAL_GRPC_INGEST`.

Another thing to note is that the ingest API endpoint for the HTTP server is `/v1/traces`. For a working example of this in a node.js express API, see `/examples/api/tracer.js` in our GitHub repository.

### Using otherwise standard observability tooling

To send data from your other Railway applications to this observability stack:

1. In your application's Railway service, add environment variables that reference the internal URLs:
   ```
   LOKI_URL=${{Grafana.LOKI_INTERNAL_URL}}
   PROMETHEUS_URL=${{Grafana.PROMETHEUS_INTERNAL_URL}}
   TEMPO_URL=${{Grafana.TEMPO_INTERNAL_URL}}
   ```
2. Configure your application's logging, metrics, or tracing libraries to use these URLs
3. Your application data will automatically appear in your Grafana dashboards

## Customizing Your Stack

To customize the configuration of Loki, Prometheus, or Tempo:

1. Fork the [GitHub repository](https://github.com/yourusername/grafana-railway-template)
2. Modify the configuration files in their respective directories
3. In Railway, disconnect the service you want to customize
4. Reconnect the service to your forked repository
5. Deploy the updated service

The pre-configured Grafana connections will continue to work with your customized services.

## Additional Resources

- [Locomotive: a loki transport for railway services](https://railway.com/template/jP9r-f)
- [Grafana Documentation](https://grafana.com/docs/grafana/latest/)
- [Loki Documentation](https://grafana.com/docs/loki/latest/)
- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Tempo Documentation](https://grafana.com/docs/tempo/latest/)
- [Grafana Community Forums](https://community.grafana.com/)
- [Grafana Plugins Directory](https://grafana.com/grafana/plugins/)

---

Developed and maintained by [Mykal](https://mykal.codes). For issues or suggestions, please open an issue on the [GitHub repository](https://github.com/MykalMachon/grafana-stack-railway).
