# Snippets

## Autocomplete

```bash
GET /products/_search
{
  "suggest": {
    "product-suggest": {
      "prefix": "lapt",
      "completion": {
        "field": "name.suggest"
      }
    }
  }
}
```

## Booléen complet

```bash
GET /products/_search
{
  "query": {
    "bool": {
      "must": {
        "multi_match": {
          "query": "laptop gaming",
          "fields": ["name^3", "description"],
          "type": "best_fields"
        }
      },
      "filter": [
        { "term": { "in_stock": true } },
        { "range": { "price": { "lte": 2000 } } }
      ],
      "should": [
        { "term": { "brand": "Apple" } }
      ]
    }
  }
}
```

## Facettes

```bash
GET /products/_search
{
  "query": { "match": { "name": "laptop" } },
  "aggs": {
    "categories": { "terms": { "field": "category" } },
    "brands": { "terms": { "field": "brand" } },
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 500, "key": "budget" },
          { "from": 500, "to": 1000, "key": "mid" },
          { "from": 1000, "key": "high" }
        ]
      }
    }
  }
}
```

## Recherche géo

```bash
GET /places/_search
{
  "query": {
    "bool": {
      "must": { "match": { "name": "restaurant" } },
      "filter": {
        "geo_distance": {
          "distance": "1km",
          "location": { "lat": 48.85, "lon": 2.35 }
        }
      }
    }
  },
  "sort": [
    { "_geo_distance": { "location": { "lat": 48.85, "lon": 2.35 }, "order": "asc" } }
  ]
}
```

## Date histogram

```bash
GET /sales/_search
{
  "size": 0,
  "aggs": {
    "over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "day"
      },
      "aggs": {
        "revenue": { "sum": { "field": "amount" } },
        "moving_avg": {
          "moving_avg": { "buckets_path": "revenue", "window": 7 }
        }
      }
    }
  }
}
```

## Bulk

```bash
POST /_bulk
{ "index": { "_index": "products", "_id": "1" } }
{ "name": "P1", "price": 10 }
{ "create": { "_index": "products", "_id": "2" } }
{ "name": "P2", "price": 20 }
{ "update": { "_index": "products", "_id": "1" } }
{ "doc": { "price": 15 } }
{ "delete": { "_index": "products", "_id": "2" } }
```

## Update by query

```bash
POST /products/_update_by_query
{
  "query": { "range": { "created_at": { "lte": "now-30d" } } },
  "script": { "source": "ctx._source.old = true" }
}
```

## Reindex

```bash
POST _reindex
{
  "source": { "index": "products_v1" },
  "dest": { "index": "products_v2" },
  "script": { "source": "ctx._source.version = 2" }
}
```

## Rollover

```bash
POST /logs/_rollover
{
  "conditions": {
    "max_age": "7d",
    "max_size": "50gb"
  }
}
```

## Alias switch

```bash
POST /_aliases
{
  "actions": [
    { "remove": { "index": "products_v1", "alias": "products" } },
    { "add": { "index": "products_v2", "alias": "products" } }
  ]
}
```
