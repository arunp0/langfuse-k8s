# Data Retention Examples

These examples enable the chart's built-in data retention CronJobs, which keep disk
usage flat by expiring raw S3/MinIO event archives, applying ClickHouse TTLs (including
the otherwise-unbounded `system.*_log` tables) and optionally cleaning old Postgres rows.

Media files are never deleted by these jobs, and credentials are reused from the
regular `s3.*` / `clickhouse.*` / `postgresql.*` values - no extra secrets needed.

## Profiles

| File | Profile |
| --- | --- |
| `values.yaml` | Defaults: 30-day window for everything, daily jobs |
| `with-long-history.yaml` | 7-day raw S3 archive, 90-day queryable ClickHouse history |
| `with-external-services.yaml` | External S3 + ClickHouse (TLS) + Postgres |
| [`../production-retention`](../production-retention) | Aggressive 7-day window everywhere, daily sweep, Postgres cleanup |

## Installation

Combine a profile with your secrets values (see [minimal-installation](../minimal-installation)):

```bash
helm repo add langfuse https://cbeneke.github.io/langfuse-k8s
helm repo update

helm install langfuse langfuse/langfuse \
  -f your-secrets-values.yaml \
  -f values.yaml
```

## Operations

Run any job immediately instead of waiting for its schedule:

```bash
kubectl create job --from=cronjob/langfuse-data-retention-minio-lifecycle manual-1
kubectl create job --from=cronjob/langfuse-data-retention-clickhouse manual-2
```

Pause all retention jobs without uninstalling them:

```yaml
dataRetention:
  suspend: true
```

Inspect what a job did via its logs:

```bash
kubectl logs job/manual-1
```
