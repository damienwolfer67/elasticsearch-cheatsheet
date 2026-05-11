# Recherches textuelles

Ces requêtes analysent ton texte avant de chercher, ce qui permet d'être plus flexible.

## Match — la plus utilisée

```bash
GET /products/_search
{
  "query": {
    "match": {
      "description": "wireless headphones"
    }
  }
}
```

Elasticsearch analyse "wireless headphones" et cherche les deux termes.

### Avec AND

```bash
{
  "match": {
    "description": {
      "query": "wireless bluetooth",
      "operator": "and"
    }
  }
}
```

Ici les deux termes doivent être présents.

### Avec tolérance aux fautes (fuzziness)

```bash
{
  "match": {
    "name": {
      "query": "iphone",
      "fuzziness": "AUTO"
    }
  }
}
```

`AUTO` veut dire : 0 ou 1 caractère de différence selon la longueur du terme.

### Minimum de termes requis

```bash
{
  "match": {
    "description": {
      "query": "laptop gaming computer",
      "minimum_should_match": 2
    }
  }
}
```

Au moins 2 des 3 termes doivent être présents.

## Match Phrase — phrase exacte

```bash
GET /products/_search
{
  "query": {
    "match_phrase": {
      "description": "wireless headphones"
    }
  }
}
```

Là il faut vraiment la séquence exacte.

### Avec slop (tolérance d'écart)

```bash
{
  "match_phrase": {
    "description": {
      "query": "wireless headphones",
      "slop": 2
    }
  }
}
```

Permet jusqu'à 2 mots entre "wireless" et "headphones".

## Match Phrase Prefix — autocomplete

```bash
GET /products/_search
{
  "query": {
    "match_phrase_prefix": {
      "name": {
        "query": "wireless hea",
        "max_expansions": 10
      }
    }
  }
}
```

Complète la fin du dernier terme.

⚠️ Attention avec `max_expansions` sur des préfixes courts.

## Multi Match — plusieurs champs à la fois

```bash
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "laptop",
      "fields": ["name^3", "description", "tags^2"]
    }
  }
}
```

Le `^3` et `^2` boostent ces champs (ils comptent plus dans le score).

### Les types de multi_match

| Type | Comportement |
|------|--------------|
| `best_fields` (défaut) | Meilleur champ compte |
| `most_fields` | Cumule les scores |
| `cross_fields` | Tous champs comme un seul |
| `phrase` | Mode phrase sur tous champs |
| `phrase_prefix` | Mode autocomplete |

```bash
{
  "multi_match": {
    "query": "laptop gaming",
    "type": "best_fields",
    "tie_breaker": 0.3
  }
}
```

`tie_breaker` : 0 = seul le meilleur compte, 1 = tous comptent pareil.

## Query String — syntaxe complète

```bash
GET /products/_search
{
  "query": {
    "query_string": {
      "query": "(laptop OR computer) AND (apple OR dell) NOT refurbished"
    }
  }
}
```

### Syntaxe

| Syntaxe | Exemple |
|---------|---------|
| `AND`, `OR` | `laptop AND computer` |
| `NOT` | `laptop NOT refurbished` |
| `+`, `-` | `+laptop -refurbished` |
| `""` | `"laptop computer"` (phrase) |
| `*` | `lapt*` (wildcard) |
| `()` | `(laptop OR computer) AND apple` |
| `[]`, `{}` | `price:[100 TO 500]` |

⚠️ Attention : cette API peut retourner des erreurs de syntaxe.

## Simple Query String — sans erreurs

```bash
GET /products/_search
{
  "query": {
    "simple_query_string": {
      "query": "laptop +computer -refurbished",
      "flags": "OR|AND|NOT"
    }
  }
}
```

Même syntaxe que `query_string` mais ne plante jamais sur une erreur de syntaxe.

## Quelle requête utiliser ?

| Tu veux... | Utilise... |
|------------|-----------|
| Rechercher du texte normalement | `match` |
| Phrase exacte | `match_phrase` |
| Autocomplete | `match_phrase_prefix` |
| Chercher sur plusieurs champs | `multi_match` |
| Utiliser des opérateurs complexes | `query_string` |
| Champs utilisateurs sans risque d'erreur | `simple_query_string` |
