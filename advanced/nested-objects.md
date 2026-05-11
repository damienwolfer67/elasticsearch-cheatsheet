# Nested & Join

## Nested

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "reviews": {
        "type": "nested",
        "properties": {
          "user": { "type": "keyword" },
          "rating": { "type": "integer" }
        }
      }
    }
  }
}
```

### Query

```bash
GET /products/_search
{
  "query": {
    "nested": {
      "path": "reviews",
      "query": {
        "bool": {
          "must": [
            { "match": { "reviews.comment": "excellent" } },
            { "range": { "reviews.rating": { "gte": 4 } } }
          ]
        }
      }
    }
  }
}
```

### Aggregation

```bash
GET /products/_search
{
  "size": 0,
  "aggs": {
    "reviews": {
      "nested": { "path": "reviews" },
      "aggs": {
        "by_user": {
          "terms": { "field": "reviews.user" }
        }
      }
    }
  }
}
```

### Reverse Nested

```bash
GET /invoices/_search
{
  "size": 0,
  "aggs": {
    "items": {
      "nested": { "path": "items" },
      "aggs": {
        "customers": {
          "reverse_nested": {},
          "aggs": {
            "by_name": {
              "terms": { "field": "customer.name" }
            }
          }
        }
      }
    }
  }
}
```

## Join (Parent-Enfant)

```bash
PUT /company
{
  "mappings": {
    "properties": {
      "relation": {
        "type": "join",
        "relations": {
          "question": ["answer", "comment"]
        }
      }
    }
  }
}
```

### Index

```bash
// Parent
PUT /company/_doc/1
{
  "text": "Question ?",
  "relation": { "name": "question" }
}

// Enfant (routing obligatoire)
PUT /company/_doc/2?routing=1
{
  "text": "Answer",
  "relation": { "name": "answer", "parent": "1" }
}
```

### Query

```bash
// Has child
GET /company/_search
{
  "query": {
    "has_child": {
      "type": "answer",
      "query": { "match": { "text": "elasticsearch" } }
    }
  }
}

// Has parent
GET /company/_search
{
  "query": {
    "has_parent": {
      "parent_type": "question",
      "query": { "match": { "text": "question" } }
    }
  }
}
```

## Comparaison

| | Nested | Join |
|---|--------|------|
| Performance | Bon < 1000 objets | Meilleur grosses collections |
| Latence | Plus rapide | Plus lent |
| Relations | Enfant direct | Multi-niveaux |
| Routing | Auto | Manuel requis |
