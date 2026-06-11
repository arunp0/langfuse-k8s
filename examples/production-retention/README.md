# Production retention profile (7-day window)

This example enables the chart's built-in data retention with an aggressive 7-day
window for all data stores, plus the optional MinIO sweep job on a daily schedule.

## Why

Without retention, a Langfuse deployment grows disk unboundedly:

- Raw event uploads accumulate in S3/MinIO forever (`{projectId}/{type}/...`, `otel/...`).
- ClickHouse application tables have no TTL.
- ClickHouse internal `system.*_log` tables have no TTL and typically outgrow the
  application data itself (one production deployment: 40 GiB of system logs next to
  2.5 GiB of data — 107 GB total before cleanup, ~30 GB after).

## Usage

```bash
helm install langfuse langfuse/langfuse \
  -f your-secrets-values.yaml \
  -f values.yaml
```

Trigger any job immediately instead of waiting for the schedule:

```bash
kubectl create job --from=cronjob/langfuse-data-retention-clickhouse manual-1
kubectl create job --from=cronjob/langfuse-data-retention-minio-lifecycle manual-2
```

## Tuning

- Keep visible trace history longer than the raw archive: set
  `dataRetention.days: 7` (raw events) and `dataRetention.clickhouse.days: 90`
  (queryable history) — the per-component value wins.
- Media files are never deleted by these jobs, regardless of settings.
- `dataRetention.suspend: true` pauses all retention jobs without uninstalling them.
