# Vector Search et Embeddings

## Dense Vector

Stocke des embeddings pour la recherche sémantique.

```bash
PUT /embeddings
{
  "mappings": {
    "properties": {
      "text_vector": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,
        "similarity": "cosine"
      },
      "text": {
        "type": "text"
      }
    }
  }
}
```

| Paramètre | Description |
|-----------|-------------|
| `dims` | Dimension du vecteur (384 pour sentence-transformers, 768 pour BERT, 1536 pour OpenAI) |
| `index` | Indexer pour knn search |
| `similarity` | `cosine`, `dot_product`, `l2_norm` |

## Indexer avec embeddings

```bash
# Indexer un document avec son embedding
PUT /embeddings/_doc/1
{
  "text": "Laptop Dell XPS pour le développement",
  "text_vector": [0.12, -0.34, 0.56, ...]
}

# Via _bulk
POST /_bulk
{ "index": { "_index": "embeddings" } }
{ "text": "iPhone 15 Pro", "text_vector": [0.23, 0.45, -0.12, ...] }
```

## kNN Search

### kNN query

```bash
GET /embeddings/_search
{
  "knn": {
    "field": "text_vector",
    "query_vector": [0.15, -0.23, 0.41, ...],
    "k": 10,
    "num_candidates": 100
  },
  "_source": ["text"]
}
```

| Paramètre | Description |
|-----------|-------------|
| `k` | Nombre de voisins à retourner |
| `num_candidates` | Candidats à considérer (accuracy vs performance) |

### kNN avec filtre

```bash
GET /embeddings/_search
{
  "knn": {
    "field": "text_vector",
    "query_vector": [0.15, -0.23, 0.41, ...],
    "k": 5,
    "filter": {
      "term": { "category": "electronics" }
    }
  }
}
```

### Hybrid search (vector + keyword)

```bash
GET /embeddings/_search
{
  "query": {
    "match": {
      "text": "laptop gaming"
    }
  },
  "knn": {
    "field": "text_vector",
    "query_vector": [0.15, -0.23, 0.41, ...],
    "k": 5
  },
  "rank": {
    "type": "weighted_average",
    "weights": {
      "knn": 0.7,
      "query": 0.3
    }
  }
}
```

## Semantic search

### Comparatif des approches

| Approche | Type | Quand l'utiliser | Performance | Coût |
|----------|------|------------------|-------------|------|
| **BM25** | Keyword | Recherche exacte, termes spécifiques | ⚡ Rapide | € Gratuit |
| **Dense embeddings** | Vectorielle | Sens sémantique, synonymes, paraphrases | 🟡 Moyen | € Modèle (local ou API) |
| **ELSER (Sparse)** | Expansive | Domaine spécifique, termes techniques | 🟡 Moyen | €€ License ML |
| **Hybrid** | Mixte | Meilleur des deux mondes | 🟢 Lent+ | €€€ Modèle + compute |

### BM25 vs Semantic

```bash
# BM25 - classique
GET /products/_search
{
  "query": {
    "match": { "description": "ordinateur portable" }
  }
}
# Résultats : doit contenir "ordinateur" et "portable"
# ❌ Ne trouve pas "laptop" si pas de synonymes

# Dense - sémantique
GET /products/_search
{
  "knn": {
    "field": "description_embedding",
    "query_vector_builder": {
      "text_embedding": {
        "model_id": ".multilingual-e5-small",
        "model_text": "ordinateur portable"
      }
    },
    "k": 5
  }
}
# Résultats : laptop, notebook, PC portable...
# ✅ Trouve les équivalents sémantiques
```

### Hybrid search - Pourquoi ?

BM25 = précision sur les termes exacts
Semantic = compréhension du sens

```bash
# Best des deux mondes
GET /products/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "description": {
              "query": "laptop gaming",
              "boost": 1
            }
          }
        }
      ]
    }
  },
  "knn": {
    "field": "description_embedding",
    "query_vector_builder": {
      "text_embedding": {
        "model_id": ".multilingual-e5-small",
        "model_text": "laptop gaming"
      }
    },
    "k": 5
  },
  "rank": {
    "type": "weighted_average",
    "weights": {
      "knn": 0.7,
      "query": 0.3
    }
  }
}
```

### Choisir son modèle

| Modèle | Type | Dimension | Langage | Usage |
|--------|------|-----------|---------|------|
| `.elser_model_2` | Sparse | 30K+ | Anglais principalement | Domaine technique |
| `.multilingual-e5-small` | Dense | 384 | 100+ langues | Généraliste |
| `.multilingual-e5-base` | Dense | 768 | 100+ langues | Plus précis |
| `sentence-transformers` | Dense | 384-768 | Variable | Open source |
| `text-embedding-3-small` | Dense | 1536 | Multilingue | OpenAI |

## Best practices

### Choix de la similarité

```bash
# Cosine - recommandé pour embeddings
"similarity": "cosine"
# ✅ Robuste aux différences de magnitude
# ✅ Normalisation pas nécessaire

# Dot product - vecteurs normalisés
"similarity": "dot_product"
# ✅ Plus rapide que cosine
# ⚠️ Nécessite des vecteurs normalisés

# L2 norm - distance euclidienne
"similarity": "l2_norm"
# ⚠️ Sensible à la magnitude
```

### Dimension des vecteurs

| Dimension | Usage | Stockage | Vitesse |
|-----------|-------|----------|---------|
| 384 | Général, rapide | + | ⚡⚡⚡ |
| 768 | Précision moyenne | ++ | ⚡⚡ |
| 1536 | Haute précision | +++ | ⚡ |

**Recommandation** : Commencer avec 384, augmenter seulement si nécessaire.

### num_candidates

```bash
"knn": {
  "k": 10,
  "num_candidates": 100  # 10x k est un bon point de départ
}
```

| Taille index | num_candidates |
|--------------|----------------|
| < 100K docs | 50-100 |
| 100K-1M | 100-500 |
| 1M-10M | 500-1000 |
| \> 10M | 1000+ |

### Hybrid search weights

```bash
"rank": {
  "type": "weighted_average",
  "weights": {
    "knn": 0.7,    # Sémantique
    "query": 0.3   # Keywords
  }
}
```

Ajuster selon le contenu :
- **Contenu technique** : plus de keyword (0.5/0.5)
- **Contenu naturel** : plus de sémantique (0.8/0.2)
- **Recherche exacte** : keyword dominant (0.2/0.8)

### Stockage des embeddings

```bash
# Stocker le texte original ET l'embedding
{
  "text": "Laptop Dell XPS",
  "text_embedding": [0.12, -0.34, ...]
}
```

❌ Ne pas stocker que l'embedding - tu perds le texte pour les snippets

### RAG chunking

```python
# Bonnes pratiques
chunk_size = 500        # Caractères par chunk
chunk_overlap = 100     # Chevauchement (20%)
```

- **Chunk trop petit** : pas assez de contexte
- **Chunk trop grand** : bruit dans la recherche
- **Overlap** : évite de couper des informations importantes

### Refresh interval

```bash
PUT /embeddings/_settings
{
  "refresh_interval": "30s"  # Indexation bulk
}

# Remettre après
PUT /embeddings/_settings
{
  "refresh_interval": "1s"
}
```

### Index sizing

Pour la recherche vectorielle, viser des shards plus petits :

```bash
PUT /embeddings
{
  "settings": {
    "number_of_shards": 6,      # Plus que du standard
    "number_of_replicas": 1
  }
}
```

Les vecteurs consomment plus de mémoire lors des recherches kNN.

### Pré-filter vs Post-filter

```bash
# ✅ Pré-filter (recommandé)
"knn": {
  "filter": {
    "term": { "category": "electronics" }
  }
}

# ❌ Post_filter - appliqué APRES le knn
# Perd des résultats pertinents
```

### Normalisation des vecteurs

```python
import numpy as np

def normalize(v):
    norm = np.linalg.norm(v)
    if norm == 0:
        return v
    return v / norm

# Permet d'utiliser dot_product au lieu de cosine
embedding = normalize(model.encode(text))
```

### Monitoring

```bash
# Vérifier l'usage mémoire kNN
GET /_nodes/stats/indices/knn?human

# Stats d'index
GET /embeddings/_stats/knn?human
```

### Quand utiliser quoi

| Scénario | Approche |
|----------|----------|
| ID exact, code produit | BM25 (`term`) |
| Recherche web classique | BM25 (`match`) |
| Synonymes, paraphrases | Dense (384 dims) |
| Questions/Réponses | Dense (768 dims) |
| Domaine médical/juridique | ELSER sparse |
| Meilleur résultats | Hybrid (70/30) |

### Dense vs Sparse

| Type | Exemple | Dimension | Usage |
|------|---------|-----------|-------|
| **Dense** | sentence-transformers, OpenAI | 384-1536 | Sémantique globale, similarité de sens |
| **Sparse** | ELSER, SPLADE | ~30 000 tokens | Mots-clés pondérés, précision termes |

Dense = un vecteur compact par document.
Sparse = expansion de tokens avec scores (type `rank_features`).

### Pattern complet avec Python

```python
from sentence_transformers import SentenceTransformer
from elasticsearch import Elasticsearch

# 1. Charger le modèle
model = SentenceTransformer('all-MiniLM-L6-v2')
es = Elasticsearch('http://localhost:9200')

# 2. Générer l'embedding de la requête
query = "laptop gaming performant"
query_vector = model.encode(query).tolist()

# 3. Chercher les documents similaires
response = es.search(
    index="products",
    body={
        "knn": {
            "field": "embedding",
            "query_vector": query_vector,
            "k": 5,
            "num_candidates": 100
        },
        "_source": ["name", "description"]
    }
)
```

### Avec ingestion automatique

```bash
# 1. Créer le pipeline avec inference
PUT _ingest/pipeline/embeddings
{
  "processors": [
    {
      "inference": {
        "model_id": "sentence-transformers__all-minilm-l6-v2",
        "input_output": [
          {
            "input_field": "text",
            "output_field": "text_embedding"
          }
        ]
      }
    }
  ]
}

# 2. Créer l'index avec le pipeline
PUT /documents
{
  "settings": {
    "default_pipeline": "embeddings"
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text"
      },
      "text_embedding": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,
        "similarity": "cosine"
      }
    }
  }
}

# 3. Indexer (embedding auto-généré)
POST /documents/_doc/1
{
  "text": "Laptop gaming avec RTX 4090"
}

# 4. Rechercher avec inference
GET /documents/_search
{
  "knn": {
    "field": "text_embedding",
    "query_vector_builder": {
      "text_embedding": {
        "model_id": "sentence-transformers__all-minilm-l6-v2",
        "model_text": "ordinateur pour jouer"
      }
    },
    "k": 5,
    "num_candidates": 100
  }
}
```

L'embedding de la requête est généré automatiquement par Elasticsearch avec `query_vector_builder`.

## Approximate kNN (ANN)

Pour des millions de vecteurs, utilise l'index approximatif.

```bash
PUT /embeddings/_settings
{
  "index": {
    "knn": true,
    "knn.algo_param.ef_search": 100
  }
}

PUT /embeddings
{
  "settings": {
    "index": {
      "knn": true,
      "knn.space_type": "cosinesimil",
      "knn.algo_param.ef_search": 100
    }
  },
  "mappings": {
    "properties": {
      "embedding": {
        "type": "knn_vector",
        "dimension": 384
      }
    }
  }
}
```

## Similarity

```bash
# Cosine (recommandé pour embeddings normalisés)
{
  "type": "dense_vector",
  "similarity": "cosine"
}

# Dot product (vecteurs normalisés)
{
  "similarity": "dot_product"
}

# L2 norm (distance euclidienne)
{
  "similarity": "l2_norm"
}
```

## RAG avec Elasticsearch

```bash
# 1. Indexer les chunks de documents avec embeddings
POST /chunks/_bulk
{ "index": { "_index": "chunks" } }
{ "text": "Python est un langage de programmation...", "embedding": [...], "doc_id": "doc_1" }

# 2. Rechercher les chunks pertinents pour la question
GET /chunks/_search
{
  "knn": {
    "field": "embedding",
    "query_vector": [question_embedding],
    "k": 3
  }
}

# 3. Utiliser les résultats pour le prompt LLM
```

## Neural search avec ELSER

ELSER (Elastic Learned Sparse Encoder) est un modèle de retrieval sparse.

```bash
# Créer un index avec le pipeline ELSER
PUT /articles
{
  "settings": {
    "index": {
      "default_pipeline": "elser_embedding"
    }
  },
  "mappings": {
    "properties": {
      "ml.tokens": {
        "type": "rank_features"
      },
      "text": {
        "type": "text"
      }
    }
  }
}

# Ingestion
POST /articles/_doc/1
{
  "text": "Python est un langage interprété..."
}

# Neural search
GET /articles/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "rank_features": {
            "field": "ml.tokens",
            "query": {
              "token": "python", "query": 0.5
            }
          }
        }
      ]
    }
  }
}
```

## Multi-vector search

```bash
# Plusieurs vecteurs par document
PUT /products
{
  "mappings": {
    "properties": {
      "name_vector": {
        "type": "dense_vector",
        "dims": 384
      },
      "description_vector": {
        "type": "dense_vector",
        "dims": 384
      }
    }
  }
}

# Recherche sur plusieurs champs
GET /products/_search
{
  "knn": [
    {
      "field": "name_vector",
      "query_vector": [0.1, ...],
      "k": 5
    },
    {
      "field": "description_vector",
      "query_vector": [0.2, ...],
      "k": 5
    }
  ]
}
```

## Tokenization with embeddings

```bash
# Embeddings au niveau token
PUT /tokens
{
  "mappings": {
    "properties": {
      "token": {
        "type": "text"
      },
      "token_embedding": {
        "type": "dense_vector",
        "dims": 768,
        "index": true
      }
    }
  }
}
```

## Vector field dans une agrégation

```bash
# Pas directement possible, workaround avec script
GET /documents/_search
{
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category"
      },
      "aggs": {
        "avg_similarity": {
          "bucket_selector": {
            "buckets_path": {},
            "script": {
              "source": "return true"
            }
          }
        }
      }
    }
  }
}
```

## Créer un embedding

```bash
# En Python avec sentence-transformers
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')
embedding = model.encode("Laptop Dell XPS")

# Avec OpenAI
import openai

response = openai.Embedding.create(
    input="Laptop Dell XPS",
    model="text-embedding-3-small"
)
embedding = response['data'][0]['embedding']
```

## Index settings pour kNN

```bash
PUT /vectors
{
  "settings": {
    "index": {
      "knn": true,
      "knn.algo_param.ef_search": 100,
      "knn.space_type": "cosinesimil"
    }
  }
}
```

| Espace | Description |
|--------|-------------|
| `l2` | Distance euclidienne |
| `l1` | Distance Manhattan |
| `cosinesimil` | Similarité cosinus |
| `dotproduct` | Produit scalaire |

## Performance kNN

```bash
# Augmenter num_candidates pour plus de précision
# Le réduire pour plus de vitesse

{
  "knn": {
    "k": 10,
    "num_candidates": 100  # 50-1000 selon données
  }
}
```

## Sparse vector (text_expansion)

```bash
# Expansion de requête avec text_expansion
PUT /articles
{
  "mappings": {
    "properties": {
      "text_expansion": {
        "type": "rank_features"
      }
    }
  }
}

# Ingestion avec expansion
POST /_ingest/pipeline/expand-text
{
  "processors": [
    {
      "text_embedding": {
        "model_id": "elser_v2",
        "model_text": "text"
      }
    }
  ]
}
```

## Best practices RAG

1. **Chunk size** : 500-1000 caractères
2. **Overlap** : 20% entre chunks
3. **Embedding model** : choisir selon le langage et la taille
4. **num_candidates** : 100-1000 selon la taille de l'index
5. **Hybrid search** : combiner vector + keyword pour RAG

## Limitations

- `dense_vector` ne supporte pas les scripts
- `knn` ne peut pas être combiné avec `post_filter`
- Les requêtes kNN sont limitées à 10 000 vecteurs par requête
- `num_candidates` ne peut pas dépasser 10 000

## Retrievers (ES 8.10+)

Syntaxe simplifiée pour la recherche sémantique sans gérer les embeddings manuellement.

### Text embedding retriever

```bash
GET /articles/_search
{
  "retriever": {
    "text_embedding": {
      "field": "ml.tokens",
      "model_id": ".elser_model_2",
      "query_text": "machine learning algorithms",
      "k": 5
    }
  }
}
```

### Standard retrieval pipeline

Combine plusieurs retrievers avec une logique de scoring.

```bash
GET /articles/_search
{
  "retriever": {
    "standard": {
      "retrieval_pipeline": {
        "retrievers": [
          {
            "text_embedding": {
              "field": "ml.tokens",
              "model_id": ".elser_model_2",
              "query_text": "laptop gaming"
            }
          },
          {
            "standard": {
              "query": {
                "match": {
                  "title": "laptop gaming"
                }
              }
            }
          }
        ],
        "rank": {
          "type": "weighted_average",
          "weights": [
            { "retriever": 0, "weight": 0.7 },
            { "retriever": 1, "weight": 0.3 }
          ]
        }
      }
    }
  }
}
```

### Retrievers disponibles

| Retriever | Description |
|-----------|-------------|
| `text_embedding` | Embedding avec modèle ML |
| `standard` | Requête classique |
| `knn` | kNN sur dense_vector |
| `rrf` | Reciprocal Rank Fusion |

### RRF (Reciprocal Rank Fusion)

Fusionne les résultats de plusieurs recherches.

```bash
GET /products/_search
{
  "retriever": {
    "rrf": {
      "retrievers": [
        {
          "standard": {
            "query": {
              "match": { "name": "laptop" }
            }
          }
        },
        {
          "knn": {
            "field": "name_vector",
            "query_vector": [0.1, 0.2, ...],
            "k": 5
          }
        }
      ],
      "rank_constant": 60
    }
  }
}
```

### Query expansion retriever

```bash
GET /articles/_search
{
  "retriever": {
    "query_expansion": {
      "retriever": {
        "text_embedding": {
          "field": "ml.tokens",
          "model_id": ".elser_model_2",
          "query_text": "climate change"
        }
      },
      "query_expansion_query": {
        "match": {
          "text": "{{query_expansion_response}}"
        }
      }
    }
  }
}
```

### Avantages des retrievers

- Syntaxe plus clean que le knn manuel
- Chaînage de plusieurs stratégies de retrieval
- RRF intégré pour fusionner les résultats
- Compatible avec les pipelines d'ingestion ML
