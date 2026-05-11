# CRUD

## Index

```bash
# Avec ID
PUT /products/_doc/1
{ "name": "Laptop", "price": 999 }

# ID auto
POST /products/_doc
{ "name": "iPhone" }

# Créer uniquement (erreur si existe)
PUT /products/_create/1
{ "name": "New" }
```

## Get

```bash
GET /products/_doc/1

# Certains champs
GET /products/_doc/1?_source=name,price

# Multi-get
GET /products/_mget
{ "ids": ["1", "2", "3"] }
```

## Update

```bash
# Partiel
POST /products/_update/1
{ "doc": { "price": 899 } }

# Script
POST /products/_update/1
{
  "script": {
    "source": "ctx._source.price += params.inc",
    "params": { "inc": 50 }
  }
}

# Upsert
POST /products/_update/1
{
  "doc": { "price": 899 },
  "upsert": { "name": "New", "price": 0 }
}
```

## Delete

```bash
DELETE /products/_doc/1

# Par requête
POST /products/_delete_by_query
{
  "query": { "term": { "status": "deleted" } }
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

## Opérations

| Méthode | Endpoint | Action |
|---------|----------|--------|
| PUT/POST | `/{index}/_doc/{id}` | Index |
| GET | `/{index}/_doc/{id}` | Get |
| POST | `/{index}/_update/{id}` | Update |
| DELETE | `/{index}/_doc/{id}` | Delete |
| POST | `/_bulk` | Bulk |
| POST | `/_reindex` | Reindex |
