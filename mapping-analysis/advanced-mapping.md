# Mapping Avancé

## Dynamic mapping

Comportement quand Elasticsearch rencontre un champ inconnu.

```bash
# Au niveau index
PUT /products
{
  "mappings": {
    "dynamic": "strict"
  }
}

| Valeur | Comportement |
|--------|--------------|
| true | Crée les champs automatiquement (défaut) |
| false | Ignore les champs inconnus |
| strict | Erreur si champ inconnu |

# Au niveau champ
{
  "mappings": {
    "properties": {
      "metadata": {
        "dynamic": "runtime",
        "type": "object"
      }
    }
  }
}
```

## Dynamic templates

```bash
PUT /products
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_as_keywords": {
          "match_mapping_type": "string",
          "match": "*_id",
          "mapping": {
            "type": "keyword"
          }
        }
      },
      {
        "english_text": {
          "match_mapping_type": "string",
          "match": "*_text",
          "unmatch": "*_id",
          "mapping": {
            "type": "text",
            "analyzer": "english",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      },
      {
        "longs_as_dates": {
          "match_mapping_type": "long",
          "match": "*_timestamp",
          "mapping": {
            "type": "date"
          }
        }
      },
      {
        "dates": {
          "match_mapping_type": "string",
          "match": "*_date",
          "mapping": {
            "type": "date"
          }
        }
      }
    ]
  }
}
```

### Patterns

| Pattern | Description |
|---------|-------------|
| `match_mapping_type` | Type de données : string, long, double, boolean |
| `match` | Pattern du nom (ex: `*_id`, `*_text`) |
| `unmatch` | Exclure si match |
| `match_pattern` | regex : `0`, `1` (0 = entire, 1 = substring) |

## Multi-fields

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          },
          "french": {
            "type": "text",
            "analyzer": "french"
          },
          "english": {
            "type": "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}

# Recherche
GET /products/_search
{
  "query": {
    "match": { "name": "laptop" }           # standard
  }
}

GET /products/_search
{
  "query": {
    "term": { "name.keyword": "Laptop" }    # exact
  }
}
```

## Runtime fields

Champs calculés à la lecture, pas stockés.

```bash
# Dans le mapping
PUT /products
{
  "mappings": {
    "runtime": {
      "price_with_tax": {
        "type": "double",
        "script": {
          "source": "emit(doc['price'].value * 1.2)"
        }
      }
    }
  }
}

# Dans une requête
GET /products/_search
{
  "runtime_mappings": {
    "profit": {
      "type": "double",
      "script": {
        "source": "emit(doc['price'].value - doc['cost'].value)"
      }
    }
  },
  "query": {
    "range": {
      "profit": {
        "gte": 10
      }
    }
  }
}
```

## _source et doc_values

```bash
PUT /products
{
  "mappings": {
    "_source": {
      "enabled": true,
      "excludes": ["metadata"],
      "includes": ["name", "price"]
    },
    "properties": {
      "price": {
        "type": "scaled_float",
        "doc_values": true
      },
      "description": {
        "type": "text",
        "doc_values": false,
        "norms": false
      }
    }
  }
}
```

| Setting | Usage |
|---------|-------|
| `_source.enabled` | Stocker le doc original (true = oui, false = non) |
| `_source.excludes` | Champs à ne pas stocker dans _source |
| `_source.includes` | Seulement ces champs dans _source |
| `doc_values` | Pour agrégations/tris (par défaut true sur la plupart des types) |
| `norms` | Pour scoring (true par défaut sur text) |

## eager_global_ordinals

Pour les champs fréquemment utilisés dans les agrégations :

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "category": {
        "type": "keyword",
        "eager_global_ordinals": true
      }
    }
  }
}
```

## index_prefixes

Pour optimiser les recherches prefix :

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "product_id": {
        "type": "keyword",
        "index_prefixes": {
          "min_chars": 1,
          "max_chars": 10
        }
      }
    }
  }
}
```

## similarity

Algorithme de scoring.

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "similarity": "BM25"
      }
    }
  }
}
```

Types : `BM25` (défaut), `classic` (TF/IDF), `boolean` (juste présence)

## norms

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "tags": {
        "type": "text",
        "norms": false
      }
    }
  }
}
```

`norms: false` si tu n'as pas besoin de scoring sur ce champ (économise de la mémoire et du disque).

## copy_to

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "first_name": {
        "type": "text",
        "copy_to": "full_name"
      },
      "last_name": {
        "type": "text",
        "copy_to": "full_name"
      },
      "full_name": {
        "type": "text"
      }
    }
  }
}

# Recherche sur full_name cherche dans les deux champs
```

## ignore_above

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "product_id": {
        "type": "keyword",
        "ignore_above": 256
      }
    }
  }
}
```

Les valeurs plus longues que 256 caractères ne seront pas indexées (utile pour éviter les longs strings uniques).

## null_value

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "status": {
        "type": "keyword",
        "null_value": "unknown"
      }
    }
  }
}
```

Permet de traiter les documents sans le champ comme s'ils avaient la valeur "unknown".

## coerce

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "price": {
        "type": "float",
        "coerce": true
      }
    }
  }
}
```

`coerce: true` tente de convertir les valeurs (ex: "10" -> 10.0).

## fields avec analyzers

```bash
PUT /articles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "french",
        "search_analyzer": "french",
        "search_quote_analyzer": "french",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          },
          "stemmed": {
            "type": "text",
            "analyzer": "french_light"
          }
        }
      }
    }
  }
}
```

## update_mapping

Ajouter un champ à un mapping existant.

```bash
# Attention : ne peut pas modifier un champ existant
PUT /products/_mapping
{
  "properties": {
    "new_field": {
      "type": "keyword"
    }
  }
}
```

## Mettre à jour le mapping

```bash
# Pour modifier un mapping, il faut :
# 1. Créer un nouvel index avec le bon mapping
# 2. Reindexer les données
# 3. Basculer l'alias

POST _reindex
{
  "source": { "index": "products_v1" },
  "dest": { "index": "products_v2" }
}
```
