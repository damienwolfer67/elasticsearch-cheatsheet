# Document APIs

## Index

```bash
PUT /products/_doc/1
{ "name": "Laptop", "price": 999 }

POST /products/_doc
{ "name": "iPhone" }

PUT /products/_create/1
{ "name": "New" }
```

## Get

```bash
GET /products/_doc/1
GET /products/_doc/1?_source=name,price

GET /products/_mget
{ "ids": ["1", "2", "3"] }
```

## Update

```bash
POST /products/_update/1
{
  "doc": { "price": 899 }
}

POST /products/_update/1
{
  "script": {
    "source": "ctx._source.price += params.inc",
    "params": { "inc": 50 }
  }
}

POST /products/_update/1
{
  "doc": { "price": 899 },
  "upsert": { "name": "New", "price": 0 }
}
```

## Delete

```bash
DELETE /products/_doc/1

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

## Update By Query

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

POST _reindex?wait_for_completion=false
```
