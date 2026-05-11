# Debug

## Explain API

Comprendre pourquoi un document matche (ou pas).

```bash
# Pour un document
GET /products/_explain/1
{
  "query": {
    "match": { "name": "laptop" }
  }
}

# Dans une requête
GET /products/_search
{
  "explain": true,
  "query": {
    "match": { "name": "laptop" }
  }
}
```

La réponse montre le calcul de score détaillé.

## Profile API

Analyser les performances d'une requête.

```bash
GET /products/_search
{
  "profile": true,
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "laptop" } }
      ],
      "filter": [
        { "term": { "status": "active" } }
      ]
    }
  }
}
```

Retourne :
- Temps d'exécution par shard
- Détail de chaque partie de la requête
- Temps passé dans chaque collector

## Validate API

Vérifier si une requête est valide.

```bash
# Basique
GET /products/_validate/query
{
  "query": {
    "match": { "name": "laptop" }
  }
}

# Avec explications
GET /products/_validate/query?explain
{
  "query": {
    "bool": {
      "must": {
        "match": { "name": "laptop" }
      }
    }
  }
}

# Avec rewrite
GET /products/_validate/query?rewrite
{
  "query": {
    "match": { "name": "laptop" } }
  }
}
```

## Analyze API

Comprendre comment le texte est analysé.

```bash
# Analyzer simple
GET /_analyze
{
  "analyzer": "standard",
  "text": "Café au lait"
}

# Avec explain
GET /_analyze
{
  "analyzer": "standard",
  "text": "Café au lait",
  "explain": true
}

# Pour un champ
GET /products/_analyze
{
  "field": "name",
  "text": "iPhone 15 Pro"
}

# Avec attributes
GET /_analyze
{
  "analyzer": "french",
  "text": "Le chat mange",
  "attributes": ["keyword", "offset", "position"]
}
```

## Search shards

Voir quels shards seront utilisés.

```bash
GET /products/_search_shards

# Avec routing
GET /products/_search_shards?routing=user123
```

## Count API

Compter sans retourner les documents.

```bash
GET /products/_count
{
  "query": {
    "term": { "status": "active" }
  }
}
```

## Exists API

Vérifier si des documents existent.

```bash
# Par champ
GET /products/_search/exists
{
  "query": {
    "exists": {
      "field": "email"
    }
  }
}

# Équivalent
GET /products/_search
{
  "query": { "exists": { "field": "email" } },
  "size": 0
}
```

## Field caps

Capacités des champs d'un index.

```bash
GET /products/_field_caps

# Champs spécifiques
GET /products/_field_caps?fields=price,name

# Indices spécifiques
GET /products,orders/_field_caps
```

Retourne les types, si searchable, aggregatable, etc.

## Ping

Vérifier que le cluster répond.

```bash
HEAD /

# 200 OK = cluster up
```

## Info

Infos sur le cluster.

```bash
GET /

# Retourne : name, version, cluster_name, etc.
```

## Nodes info

```bash
GET /_nodes

# Parties spécifiques
GET /_nodes/jvm
GET /_nodes/os
GET /_nodes/process
GET /_nodes/settings
```

## Cluster state

État du cluster.

```bash
GET /_cluster/state

# Parties
GET /_cluster/state/metadata
GET /_cluster/state/blocks
GET /_cluster/state/routing_table
```

## Pending tasks

Tâches en attente dans le cluster.

```bash
GET /_cluster/pending_tasks
```

## Allocation explain

Pourquoi un shard n'est pas assigné.

```bash
GET /_cluster/allocation/explain

# Shard spécifique
GET /_cluster/allocation/explain?shard=0&index=products
```

## Hot threads

Threads consommant le plus de CPU.

```bash
GET /_nodes/hot_threads

# Sur intervalles
GET /_nodes/hot_threads?threads=5&interval=500ms
```

## Tasks

Lister les tâches en cours.

```bash
GET /_tasks

# Par groupe
GET /_tasks?actions=*search*

# Détail
GET /_tasks/task_id
```

## Ingest simulate

Tester un ingest pipeline.

```bash
POST _ingest/pipeline/my_pipeline/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "2024-01-15 INFO Test log"
      }
    }
  ]
}
```

## Search slowlog

Logger les requêtes lentes.

```bash
PUT /products/_settings
{
  "index.search.slowlog.threshold.query.warn": "10s",
  "index.search.slowlog.threshold.query.info": "5s",
  "index.search.slowlog.threshold.fetch.warn": "1s",
  "index.search.slowlog.level": "info"
}
```

## Indexing slowlog

Logger les indexations lentes.

```bash
PUT /products/_settings
{
  "index.indexing.slowlog.threshold.index.warn": "10s",
  "index.indexing.slowlog.threshold.index.info": "5s",
  "index.indexing.slowlog.level": "info"
}
```

## Audit trail

Tracer les événements de sécurité.

```bash
PUT _cluster/settings
{
  "xpack.security.audit.logfile.events": ["access_denied", "authentication_failed"]
}
```

## Common problèmes

### Requête vide

```bash
# Si ta requête ne retourne rien
GET /products/_validate/query?explain
{
  "query": { ... }
}
```

### Score inattendu

```bash
# Utilise explain pour voir le calcul
GET /products/_explain/doc_id
{
  "query": { ... }
}
```

### Mapping erreur

```bash
# Vérifier le mapping
GET /products/_mapping

# Tester l'analyse
GET /products/_analyze
{
  "field": "mon_champ",
  "text": "test"
}
```

### Shard non assigné

```bash
GET /_cluster/allocation/explain
```

### Requête lente

```bash
# 1. Profile la requête
GET /_search
{
  "profile": true,
  "query": { ... }
}

# 2. Vérifie le slowlog
GET /_cat/thread_pool?v&h=node_name,name,active,queue,rejected

# 3. Vérifie les stats
GET /_nodes/stats/indices
```
