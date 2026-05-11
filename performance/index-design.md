# Index Design

## Sharding

| Métrique | Valeur |
|----------|--------|
| Taille shard | 20-50 GB |
| Shards/GB heap | ~20 |
| Shards/nœud | < 1000 |

```bash
PUT /logs-2024-01
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}
```

## ILM Policy

```bash
PUT _ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_age": "1d",
            "max_size": "50gb"
          }
        }
      },
      "warm": {
        "min_age": "3d",
        "actions": {
          "forcemerge": { "max_num_segments": 1 },
          "shrink": { "number_of_shards": 1 }
        }
      },
      "cold": {
        "min_age": "7d",
        "actions": { "freeze": {} }
      },
      "delete": {
        "min_age": "30d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

## Template + ILM

```bash
PUT _index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "logs_policy",
      "index.lifecycle.rollover_alias": "logs"
    }
  }
}
```

## Rollover

```bash
POST /logs/_rollover
{
  "conditions": {
    "max_age": "1d",
    "max_size": "50gb",
    "max_docs": 10000000
  }
}
```

## Mapping

```bash
PUT /products
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "name": {
        "type": "text",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      },
      "price": {
        "type": "scaled_float",
        "scaling_factor": 100
      }
    }
  }
}
```

## Dynamic Templates

```bash
"dynamic_templates": [
  {
    "strings_as_keywords": {
      "match_mapping_type": "string",
      "match": "*_id",
      "mapping": { "type": "keyword" }
    }
  }
]
```

## Refresh / Translog

```bash
PUT /products/_settings
{
  "index.refresh_interval": "30s",
  "index.translog.durability": "async"
}
```

## Disk Watermarks

```bash
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.disk.watermark.low": "85%",
    "cluster.routing.allocation.disk.watermark.high": "90%",
    "cluster.routing.allocation.disk.watermark.flood_stage": "95%"
  }
}
```

## Allocation

```bash
// Par zone
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.awareness.attributes": "zone"
  }
}

// Par index
PUT /products/_settings
{
  "index.routing.allocation.require.zone": "zone1"
}
```
