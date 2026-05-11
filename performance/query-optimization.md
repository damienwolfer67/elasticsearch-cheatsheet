# Query Optimization

## Filter vs Query

```bash
// ✅ Filter : pas de score, cacheable
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "category": "laptop" } },
        { "range": { "price": { "lte": 1000 } } }
      ]
    }
  }
}

// ❌ Query : scoring inutile pour filtres
{
  "query": {
    "term": { "category": "laptop" }
  }
}
```

| Context | Score | Cache | Usage |
|---------|-------|-------|-------|
| query | Oui | Non | Full-text |
| filter | Non | Oui | Filtres exacts |

## Profiling

```bash
GET /products/_search
{
  "profile": true,
  "query": { "match": { "name": "laptop" } }
}
```

## Pagination

```bash
// ✅ search_after (recommandé)
GET /products/_search
{
  "size": 10,
  "sort": [{ "date": "asc" }, { "_id": "asc" }],
  "search_after": [1577836800000, "prod_123"]
}

// ❌ from/size (deep pagination lent)
{
  "from": 10000,
  "size": 10
}
```

## Source Filtering

```bash
GET /products/_search
{
  "_source": ["name", "price"],
  "query": { "match_all": {} }
}
```

## Doc Values

```bash
// Désactiver si pas d'aggs/tris
PUT /products
{
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "doc_values": false
      }
    }
  }
}
```

## Index Sorting

```bash
PUT /products
{
  "settings": {
    "index": {
      "sort.field": ["date", "name"],
      "sort.order": ["desc", "asc"]
    }
  }
}
```

## Routing

```bash
// Index
PUT /products/_doc/1?routing=user123
{ "name": "Product", "user_id": "user123" }

// Search
GET /products/_search?routing=user123
```

## Cache

```bash
// Activer request cache
PUT /products/_settings
{
  "index.requests.cache.enable": true
}

GET /products/_search?request_cache=true
{
  "size": 0,
  "query": {
    "bool": {
      "filter": { "term": { "status": "active" } }
    }
  }
}
```

## Timeout

```bash
GET /products/_search?timeout=2s
```

## Best Practices

- ✅ `filter` pour clauses sans scoring
- ✅ `search_after` pour pagination
- ✅ `_source` filtering
- ✅ `doc_values` pour agrégations
- ✅ `constant_score` pour filtres simples
- ❌ Éviter `wildcard` en début de pattern
- ❌ Éviter `script_fields` si possible
- ❌ Éviter `fielddata` sur text fields
