# Bucket Aggregations

## Terms

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "by_category": {
      "terms": {
        "field": "category",
        "size": 10
      }
    }
  }
}
```

## Range

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 50 },
          { "from": 50, "to": 200 },
          { "from": 200 }
        ]
      }
    }
  }
}
```

## Date Range

```bash
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "date_ranges": {
      "date_range": {
        "field": "created_at",
        "ranges": [
          { "to": "now-7d" },
          { "from": "now-7d" }
        ]
      }
    }
  }
}
```

## Histogram

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 50
      }
    }
  }
}
```

## Date Histogram

```bash
GET /logs/_search
{
  "size": 0,
  "aggs": {
    "by_day": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "day"
      }
    }
  }
}
```

Intervalle : `minute`, `hour`, `day`, `week`, `month`, `quarter`, `year`

## Filter

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "active": {
      "filter": { "term": { "in_stock": true } },
      "aggs": {
        "avg_price": { "avg": { "field": "price" } }
      }
    }
  }
}
```

## Filters

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "statuses": {
      "filters": {
        "filters": {
          "active": { "term": { "status": "active" } },
          "pending": { "term": { "status": "pending" } }
        }
      }
    }
  }
}
```

## Nested

```bash
GET /invoices/_search
{
  "size": 0,
  "aggs": {
    "items": {
      "nested": { "path": "items" },
      "aggs": {
        "by_sku": {
          "terms": { "field": "items.sku" }
        }
      }
    }
  }
}
```

## Composite (pagination)

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "buckets": {
      "composite": {
        "size": 10,
        "sources": [
          { "category": { "terms": { "field": "category" } } },
          { "brand": { "terms": { "field": "brand" } } }
        ]
      }
    }
  }
}
```

## Récapitulatif

| Agg | Usage |
|-----|-------|
| `terms` | Valeurs uniques |
| `range` | Intervalles |
| `date_range` | Intervalles dates |
| `histogram` | Histogramme numérique |
| `date_histogram` | Histogramme temporel |
| `filter` | Un filtre |
| `filters` | Plusieurs filtres |
| `nested` | Objets imbriqués |
| `composite` | Pagination buckets |
