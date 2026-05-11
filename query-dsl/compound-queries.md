# Compound Queries

## Bool

```bash
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "laptop" } }
      ],
      "filter": [
        { "term": { "in_stock": true } },
        { "range": { "price": { "lte": 2000 } } }
      ],
      "should": [
        { "term": { "brand": "Apple" } },
        { "term": { "brand": "Dell" } }
      ],
      "must_not": [
        { "term": { "refurbished": true } }
      ]
    }
  }
}
```

| Clause | Score | Cache |
|--------|-------|-------|
| `must` | Oui | Non |
| `filter` | Non | Oui |
| `should` | Oui | Non |
| `must_not` | Non | Oui |

## Minimum Should Match

```bash
{
  "bool": {
    "should": [
      { "match": { "name": "laptop" } },
      { "match": { "description": "computer" } },
      { "match": { "tags": "pc" } }
    ],
    "minimum_should_match": 2  // ou "50%"
  }
}
```

## Constant Score

```bash
GET /products/_search
{
  "query": {
    "constant_score": {
      "filter": { "term": { "status": "active" } },
      "boost": 1.2
    }
  }
}
```

## Function Score

```bash
GET /products/_search
{
  "query": {
    "function_score": {
      "query": { "match": { "name": "laptop" } },
      "functions": [
        {
          "filter": { "term": { "brand": "Apple" } },
          "weight": 10
        },
        {
          "filter": { "range": { "price": { "lte": 1000 } } },
          "weight": 5
        }
      ],
      "score_mode": "multiply",
      "boost_mode": "multiply"
    }
  }
}
```

## Boosting

```bash
GET /products/_search
{
  "query": {
    "boosting": {
      "positive": { "match": { "name": "laptop" } },
      "negative": { "term": { "refurbished": true } },
      "negative_boost": 0.2
    }
  }
}
```

## Dis Max

```bash
GET /products/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "name": "laptop" } },
        { "match": { "description": "gaming laptop" } }
      ],
      "tie_breaker": 0.3
    }
  }
}
```

## Récapitulatif

| Query | Usage |
|-------|-------|
| `bool` | Combiner clauses |
| `constant_score` | Score constant |
| `function_score` | Modifier score |
| `boosting` | Pénaliser résultats |
| `dis_max` | Meilleure clause |
