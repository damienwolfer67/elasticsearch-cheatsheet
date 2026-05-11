# Elasticsearch Cheat Sheet

Guide complet pour Elasticsearch 8.x+

## Sommaire

**Fondamentaux**
- [Concepts](./fundamentals/concepts.md)
- [Opérations CRUD](./fundamentals/basic-operations.md)

**APIs**
- [Document APIs](./api/document-apis.md)
- [Search API](./api/search-api.md)

**Requêtes**
- [Full-text](./query-dsl/full-text-queries.md)
- [Term queries](./query-dsl/term-queries.md)
- [Compound](./query-dsl/compound-queries.md)

**Agrégations**
- [Bucket](./aggregations/bucket-aggs.md)
- [Metric](./aggregations/metric-aggs.md)

**Mapping**
- [Types de champs](./mapping-analysis/field-types.md)
- [Analyseurs](./mapping-analysis/custom-analyzers.md)
- [Analyseurs détaillés](./mapping-analysis/detailed-analyzers.md)
- [Mapping avancé](./mapping-analysis/advanced-mapping.md)

**Avancé**
- [Nested et Join](./advanced/nested-objects.md)
- [Géospatial](./advanced/geo.md)
- [Scripting](./advanced/script-fields.md)
- [Ingest Pipelines](./advanced/ingest-pipelines.md)
- [Vector Search et Embeddings](./advanced/vector-search.md)

**Performance**
- [Optimisation requêtes](./performance/query-optimization.md)
- [Design d'index](./performance/index-design.md)

**Opérations**
- [Cluster health](./operations/cluster-health.md)
- [Kibana](./operations/kibana.md)
- [Alias et Reindex](./operations/aliases-reindex.md)
- [Sauvegardes](./operations/snapshots.md)
- [Debug](./operations/debug.md)

**Snippets**
- [Patterns courants](./snippets/common-patterns.md)

## Quick Start

```bash
# Health check
GET /

# Créer et indexer
PUT /products/_doc/1
{
  "name": "Laptop",
  "price": 999
}

# Rechercher
GET /products/_search
{
  "query": {
    "match": { "name": "laptop" }
  }
}
```
