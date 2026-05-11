# Alias & Reindex

## Alias

```bash
// Créer
POST /_aliases
{
  "actions": [
    { "add": { "index": "products_2024_01", "alias": "products" } }
  ]
}

// Avec filtre
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "products",
        "alias": "active",
        "filter": { "term": { "status": "active" } }
      }
    }
  ]
}

// Supprimer
DELETE /products/_alias/products_current

// Voir
GET /products/_alias
GET /_alias/*
```

## Reindex

```bash
// Basique
POST _reindex
{
  "source": { "index": "products_v1" },
  "dest": { "index": "products_v2" }
}

// Avec transformation
POST _reindex
{
  "source": { "index": "products_v1" },
  "dest": { "index": "products_v2" },
  "script": {
    "source": "ctx._source.version = 2"
  }
}

// Avec query
POST _reindex
{
  "source": {
    "index": "products_v1",
    "query": { "range": { "created_at": { "gte": "now-30d" } } }
  },
  "dest": { "index": "products_v2" }
}

// Async
POST _reindex?wait_for_completion=false
```

## Migration Zero-Downtime

```bash
// 1. Créer nouvel index
PUT /products_v2 { ... }

// 2. Reindex async
POST _reindex?wait_for_completion=false
{ "source": { "index": "products_v1" }, "dest": { "index": "products_v2" } }

// 3. Suivre
GET /_tasks/task_id

// 4. Rattraper nouvelles données
POST _reindex
{
  "source": {
    "index": "products_v1",
    "query": { "range": { "timestamp": { "gte": "start_time" } } }
  },
  "dest": { "index": "products_v2" }
}

// 5. Basculer alias
POST /_aliases
{
  "actions": [
    { "remove": { "index": "products_v1", "alias": "products" } },
    { "add": { "index": "products_v2", "alias": "products" } }
  ]
}
```

## Shrink / Split

```bash
// Shrink (read-only, shards sur un nœud)
PUT /source/_shrink/target
{
  "settings": {
    "index.number_of_shards": 1
  }
}

// Split (multiple de 2)
PUT /source/_split/target
{
  "settings": {
    "index.number_of_shards": 4
  }
}
```
