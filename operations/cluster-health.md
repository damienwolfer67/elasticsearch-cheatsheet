# Cluster Health & Operations

## Health

```bash
GET /_cluster/health
GET /_cluster/health?wait_for_status=yellow&timeout=50s
```

| Status | Signification |
|--------|---------------|
| green | Tous shards assignés |
| yellow | Primaires OK, replicas manquants |
| red | Primaires manquants |

## Cat APIs

```bash
GET _cat/indices?v
GET _cat/shards?v
GET _cat/nodes?v
GET _cat/allocation?v
GET _cat/health?v
GET _cat/pending_tasks?v
GET _cat/recovery?v
```

## Headers

```bash
GET _cat/indices?v&h=index,health,store.size&s=store.size:desc
```

## Stats

```bash
GET /_nodes/stats
GET /_nodes/stats/indices,jvm
GET /products/_stats
```

## Settings

```bash
GET /_cluster/settings

PUT /_cluster/settings
{
  "persistent": {
    "search.default_search_timeout": "30s"
  }
}
```

## Cache

```bash
POST /products/_cache/clear
POST /_cache/clear
```

## Refresh / Flush / Merge

```bash
POST /products/_refresh
POST /products/_flush
POST /products/_forcemerge?max_num_segments=1
```

## Allocation

```bash
GET /_cluster/allocation/explain

POST /_cluster/reroute?retry_failed=true

PUT /_cluster/settings
{
  "transient": {
    "cluster.routing.allocation.enable": "none"
  }
}
```

## Tasks

```bash
GET /_tasks
POST /_tasks/_cancel?actions=*search*
```
