# Concepts

## Architecture

```
Cluster
  └── Nodes (serveurs)
       └── Indices (comme des BDD)
            └── Shards (morceaux d'index)
                 └── Documents (JSON)
```

| Concept | C'est quoi |
|---------|------------|
| Cluster | Groupe de nœuds qui travaillent ensemble |
| Node | Un serveur Elasticsearch (master, data, etc.) |
| Index | Ensemble de documents (comme une table SQL) |
| Shard | Partie d'un index, stockée sur un nœud |
| Replica | Copie d'un shard pour la redondance |
| Document | Une unité de données en JSON |
| Mapping | Schéma qui définit les champs et leurs types |

## Pourquoi des shards ?

Un index est découpé en shards qui sont répartis sur les nœuds. Ça permet :

- La parallélisation : chaque shard est cherché indépendamment
- La scalabilité : on ajoute des nœuds pour plus de capacité
- La redondance : si un nœud tombe, les replicas prennent le relais

**En pratique** : visez 20-50 GB par shard, et environ 20 shards par GB de heap mémoire.

## Near Real-Time

Quand tu indexes un document, il n'est pas tout de suite searchable. Elasticsearch attend par défaut 1 seconde (configurable avec `refresh_interval`).

```
Ecriture -> Translog (journal) -> Refresh (1s) -> Searchable
                                          ->
                                      Flush -> Disque
```

Le translog garantit qu'on ne perd pas de données en cas de crash.

## Comparaison avec SQL

| Elasticsearch | MySQL/PostgreSQL |
|---------------|------------------|
| Index | Database |
| Document | Row |
| Field | Column |
| Mapping | Schema |
| Query DSL | WHERE clause |
| Aggregation | GROUP BY + COUNT/SUM... |

## L'inverted index

Le secret d'Elasticsearch pour être rapide sur la recherche full-text :

```
Documents :
Doc 1 : "The quick brown fox"
Doc 2 : "The lazy dog"
Doc 3 : "The quick cat"

Index inversé :
quick  -> [Doc 1, Doc 3]
brown  -> [Doc 1]
fox    -> [Doc 1]
lazy   -> [Doc 2]
dog    -> [Doc 2]
cat    -> [Doc 3]
```

Quand tu cherches "quick", Elasticsearch sait immédiatement quels documents contiennent ce mot, sans les scanner un par un.

## Comment fonctionne le score (TF-IDF)

```
Score = TF x IDF x Normes

TF (Term Frequency)     : Fréquence du terme dans le document
IDF (Inverse Doc Freq)  : Rareté du terme dans l'index (plus il est rare, plus il compte)
Normes                  : Normalisation selon la longueur du champ
```

Un document qui contient 5 fois le mot "laptop" aura un meilleur TF qu'un document qui ne le contient qu'une fois. Mais si "laptop" apparaît dans tous les documents, l'IDF sera bas et le mot aura moins d'importance.

## Types de nœuds

| Type | Rôle |
|------|------|
| master-eligible | Gère le cluster (élection, décisions) |
| data | Stocke et recherche les données |
| data-hot | Données récentes sur SSD (performance) |
| data-warm | Données anciennes sur HDD (coût réduit) |
| data-cold | Archive |
| ingest | Pré-traite les documents avant indexation |
| coordinating | Route les requêtes vers les bons shards |

## Cat API vs Core API

```bash
# Cat API : lisible pour humains
GET _cat/indices?v
GET _cat/shards?v
GET _cat/nodes?v

# Core API : JSON structuré
GET /_cluster/health
GET /_nodes/stats
```

## Config de base (elasticsearch.yml)

```yaml
cluster.name: mon-cluster
node.name: node-1
node.roles: [data, master]

network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["node-1", "node-2"]
cluster.initial_master_nodes: ["node-1", "node-2"]
```

## Bonnes pratiques

1. **Toujours définir tes mappings** avant d'indexer (évite les surprises)
2. **Utilise des alias** pour ne jamais manipuler les index directement
3. **Surveille la taille des shards** (20-50 GB idéal)
4. **keyword pour les filtres/tris/agrégations**, **text pour la recherche full-text**
5. **Mode `strict`** en production pour éviter les champs imprévus
