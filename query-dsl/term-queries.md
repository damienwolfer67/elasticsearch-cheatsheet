# Term Level Queries

## Term

```bash
GET /products/_search
{
  "query": {
    "term": { "status": "active" }
  }
}
```

## Terms

```bash
GET /products/_search
{
  "query": {
    "terms": {
      "status": ["active", "pending"]
    }
  }
}
```

## Range

```bash
GET /products/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 100,
        "lte": 500
      }
    }
  }
}

# gt, gte, lt, lte
```

## Exists

```bash
GET /products/_search
{
  "query": {
    "exists": { "field": "email" }
  }
}
```

## Prefix

```bash
GET /products/_search
{
  "query": {
    "prefix": {
      "category": "elect"
    }
  }
}
```

## Wildcard

```bash
GET /products/_search
{
  "query": {
    "wildcard": {
      "name": "lapt*p"
    }
  }
}
```

## Regexp

```bash
GET /products/_search
{
  "query": {
    "regexp": {
      "product_id": "PROD-[0-9]{4}"
    }
  }
}
```

## Fuzzy

```bash
GET /products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "iphone",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

## Ids

```bash
GET /_search
{
  "query": {
    "ids": {
      "values": ["1", "2", "3"]
    }
  }
}
```

## Tableau

| Query | Usage |
|-------|-------|
| `term` | Valeur exacte |
| `terms` | Plusieurs valeurs |
| `range` | Intervalles |
| `exists` | Champ non vide |
| `prefix` | Commence par |
| `wildcard` | Joker simple |
| `regexp` | Regex |
| `fuzzy` | Fautes de frappe |
| `ids` | Par IDs |
