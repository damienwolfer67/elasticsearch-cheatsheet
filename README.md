# Elasticsearch Cheat Sheet

Guide complet pour Elasticsearch 8.x+ : de la prise en main aux optimisations avancées.

## Contenu

**Fondamentaux**
- Concepts clés : inverted index, sharding, TF-IDF
- Opérations CRUD de base

**APIs**
- Document APIs : index, get, update, delete, bulk, reindex
- Search API : pagination, tri, highlighting, suggestions

**Query DSL**
- Full-text : match, match_phrase, multi_match, query_string
- Term queries : term, range, exists, prefix, wildcard, fuzzy
- Compound : bool, function_score, boosting, dis_max

**Agrégations**
- Bucket : terms, range, histogram, filters, composite
- Metric : avg, sum, stats, percentiles, top_hits, pipeline

**Mapping et Analyse**
- Types de champs : keyword, text, numeric, date, nested, geo_point, join
- Analyzers : tokenizer, char filters, token filters
- Mapping avancé : dynamic templates, runtime fields, multi-fields

**Avancé**
- Nested objects et relations parent-enfant
- Recherche géospatiale
- Scripting Painless
- Ingest pipelines
- **Vector search et embeddings** : kNN, RAG, ELSER, retrievers

**Performance**
- Optimisation de requêtes : filter vs query, profiling
- Design d'index : shard sizing, ILM, rollover

**Opérations**
- Cluster health et monitoring
- Kibana : Dev Tools, Console, Discover
- Alias et reindex : migration zero-downtime
- Snapshots et restore
- Debug : explain, profile, validate, analyze

**Snippets**
- Patterns courants : autocomplete, bool queries, geo search

## Quick Start

```bash
# Health check
GET /

# Créer un index
PUT /products
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "price": { "type": "integer" }
    }
  }
}

# Indexer un document
PUT /products/_doc/1
{
  "name": "Laptop Dell XPS",
  "price": 1299
}

# Rechercher
GET /products/_search
{
  "query": {
    "match": { "name": "laptop" }
  }
}
```

## Installation

```bash
# Docker
docker run -p 9200:9200 -e "discovery.type=single-node" \
  docker.elastic.co/elasticsearch/elasticsearch:8.x.x

# Homebrew
brew tap elastic/tap
brew install elastic/tap/elasticsearch-full

# Démarrer
./bin/elasticsearch
```

## Structure

```
elasticsearch-cheatsheet/
├── index.md                           # Page d'accueil
├── fundamentals/                      # Concepts de base
├── api/                               # APIs document et search
├── query-dsl/                         # Requêtes
├── aggregations/                      # Agrégations
├── mapping-analysis/                  # Mapping et analyseurs
├── advanced/                          # Features avancées
├── performance/                       # Optimisation
├── operations/                        # Cluster et ops
└── snippets/                          # Patterns réutilisables
```

## Contribution

Contributions welcome : corriger une erreur, ajouter un exemple, améliorer une explication.

## Ressources

- [Documentation officielle Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/guide/current/index.html)
