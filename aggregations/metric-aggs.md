# Metric Aggregations

## Avg, Sum, Min, Max

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "avg_price": { "avg": { "field": "price" } },
    "total_sales": { "sum": { "field": "amount" } },
    "min_price": { "min": { "field": "price" } },
    "max_price": { "max": { "field": "price" } }
  }
}
```

## Stats

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "stats": { "stats": { "field": "price" } },
    "extended": { "extended_stats": { "field": "price" } }
  }
}
```

## Cardinality

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "unique_categories": {
      "cardinality": {
        "field": "category",
        "precision_threshold": 100
      }
    }
  }
}
```

## Percentiles

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "percentiles": {
      "percentiles": {
        "field": "price",
        "percents": [1, 5, 25, 50, 75, 95, 99]
      }
    }
  }
}
```

## Top Hits

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "by_category": {
      "terms": { "field": "category" },
      "aggs": {
        "top": {
          "top_hits": {
            "size": 3,
            "sort": [{ "price": "desc" }]
          }
        }
      }
    }
  }
}
```

## Geo Bounds / Centroid

```bash
GET /locations/_search
{
  "size": 0,
  "aggs": {
    "bounds": { "geo_bounds": { "field": "location" } },
    "center": { "geo_centroid": { "field": "location" } }
  }
}
```

## Pipeline (bucket stats)

```bash
GET /sales/_search
{
  "size": 0,
  "aggs": {
    "by_month": {
      "date_histogram": { "field": "date", "calendar_interval": "month" },
      "aggs": {
        "sales": { "sum": { "field": "amount" } }
      }
    },
    "max_month": {
      "max_bucket": {
        "buckets_path": "by_month>sales"
      }
    },
    "avg_monthly": {
      "avg_bucket": {
        "buckets_path": "by_month>sales"
      }
    }
  }
}
```

## Moving Functions

```bash
GET /sales/_search
{
  "size": 0,
  "aggs": {
    "by_date": {
      "date_histogram": { "field": "date", "fixed_interval": "1d" },
      "aggs": {
        "sales": { "sum": { "field": "amount" } },
        "moving_avg": {
          "moving_avg": {
            "buckets_path": "sales",
            "window": 7
          }
        },
        "cumulative": {
          "cumulative_sum": {
            "buckets_path": "sales"
          }
        },
        "derivative": {
          "derivative": {
            "buckets_path": "sales"
          }
        }
      }
    }
  }
}
```

## Récapitulatif

| Agg | Usage |
|-----|-------|
| `avg/sum/min/max` | Métriques de base |
| `stats` | Toutes stats |
| `extended_stats` | + variance, std_dev |
| `cardinality` | Valeurs uniques |
| `percentiles` | Distribution |
| `top_hits` | Top documents |
| `max_bucket` | Bucket max |
| `avg_bucket` | Moyenne buckets |
| `moving_avg` | Moyenne mobile |
| `cumulative_sum` | Somme cumulée |
| `derivative` | Taux changement |
