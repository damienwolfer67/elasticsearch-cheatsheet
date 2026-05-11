# Search API

## Basique

```bash
GET /products/_search?q=name:laptop

GET /products/_search
{
  "query": { "match": { "name": "laptop" } }
}
```

## Paramètres

```bash
GET /products/_search?from=0&size=10&sort=price:asc
```

| Paramètre | Description |
|-----------|-------------|
| `from/size` | Pagination |
| `sort` | Tri |
| `_source` | Champs à retourner |
| `timeout` | Timeout requête |
| `request_cache` | Cache |
| `routing` | Routing spécifique |
| `preference` | Préférence shard |
| `track_total_hits` | Précision total |

## Structure

```bash
GET /products/_search
{
  "query": { ... },
  "from": 0,
  "size": 10,
  "sort": [{ "price": "asc" }],
  "_source": ["name", "price"],
  "highlight": { ... },
  "aggs": { ... }
}
```

## Pagination

```bash
// search_after (recommandé)
GET /products/_search
{
  "size": 10,
  "sort": [{ "date": "desc" }, { "_id": "asc" }],
  "search_after": [1577836800000, "prod_123"]
}
```

## Highlight

```bash
GET /products/_search
{
  "query": { "match": { "description": "laptop" } },
  "highlight": {
    "fields": {
      "description": {
        "pre_tags": ["<em>"],
        "post_tags": ["</em>"],
        "fragment_size": 150
      }
    }
  }
}
```

## Collapse

```bash
GET /products/_search
{
  "query": { "match": { "name": "laptop" } },
  "collapse": {
    "field": "brand",
    "inner_hits": { "size": 3 }
  }
}
```

## Multi Search

```bash
GET /_msearch
{ "index": "products" }
{ "query": { "match": { "name": "laptop" } }, "size": 2 }
{ "index": "orders" }
{ "query": { "match": { "product": "laptop" } } }
```

## Explain / Profile

```bash
GET /products/_search
{
  "explain": true,
  "profile": true,
  "query": { "match": { "name": "laptop" } }
}
```

## Count

```bash
GET /products/_count
GET /products/_count?q=status:active
```

## Validate

```bash
GET /products/_validate/query
{
  "query": { "match": { "name": "laptop" } }
}
```

## Point in Time

```bash
POST /products/_pit?keep_alive=10m

GET /_search
{
  "pit": {
    "id": "pit_id",
    "keep_alive": "10m"
  },
  "query": { "match": { "name": "laptop" } }
}
```
