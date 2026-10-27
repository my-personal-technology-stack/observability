# Observability Stack

A Docker-based observability stack for monitoring container metrics and centralizing container logs.

## Architecture

| Component | Responsibility | Local endpoint |
|---|---|---|
| Grafana | Dashboards and data visualization | http://localhost:3000 |
| Prometheus | Metrics collection and storage | http://localhost:9090 |
| cAdvisor | Container resource metrics | http://localhost:8080 |
| Loki | Log aggregation and storage | http://localhost:3100 |
| Grafana Alloy | Docker log discovery and collection | http://localhost:12345 |

Prometheus and cAdvisor provide metrics. Alloy collects container logs and forwards them to Loki. Grafana provides a unified interface for querying and visualizing both.

## Repository structure

- `docker-compose.yml`: service definitions, networking, ports and persistent volumes.
- `.env.example`: template for required environment variables and image versions.
- `.env`: local environment configuration; never commit secrets.
- `config/alloy/`: Docker log collection pipeline.
- `config/grafana/dashboards/`: version-controlled dashboard definitions.
- `config/grafana/provisioning/`: automatic datasource and dashboard provisioning.
- `config/loki/`: Loki storage and retention settings.
- `config/prometheus/`: Prometheus scrape configuration.

## Prerequisites

- Docker Desktop with Linux containers enabled.
- Docker Compose v2.
- A local environment file containing the required variables.

## Start the stack

Validate the Compose configuration:

```powershell
docker compose config --quiet
```

Start all services:

```powershell
docker compose up -d
```

Check the service status:

```powershell
docker compose ps
```

## Validate the installation

Check container logs:

```powershell
docker compose logs --tail 100
```

Check Loki readiness:

```powershell
Invoke-RestMethod http://localhost:3100/ready
```

Open Grafana at http://localhost:3000 and verify that Prometheus and Loki are configured as datasources.

In Grafana Explore, use the following LogQL query to inspect Docker logs:

```logql
{job="docker"}
```

## Maintenance

Restart the stack:

```powershell
docker compose restart
```

Stop the stack without deleting persistent data:

```powershell
docker compose down
```

Update services after changing configuration or image versions:

```powershell
docker compose pull
docker compose up -d
```

Review logs when troubleshooting:

```powershell
docker compose logs -f alloy loki
```

## Data persistence

Named Docker volumes store application data independently of container lifecycles.

The `docker compose down` command preserves named volumes by default. Do not use `docker compose down -v` unless you intentionally want to delete the persistent data.

## Security

- Keep local service ports bound to `127.0.0.1` unless remote access is explicitly required.
- Never commit passwords, tokens or other secrets.
- Restrict access to the Docker socket because it provides significant control over the Docker daemon.
- Review privileged container settings and mounted host paths.
- Configure and monitor data retention to prevent unbounded disk usage.

## Configuration management

Configuration files are stored in Git so that changes can be reviewed, reproduced and rolled back.

After modifying a configuration file, validate the Compose configuration and restart only the affected service when appropriate.
