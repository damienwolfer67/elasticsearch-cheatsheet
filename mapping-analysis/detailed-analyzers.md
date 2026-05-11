# Analyzers Détaillés

## Pipeline d'analyse

```
Texte > Char Filter > Tokenizer > Token Filter > Tokens
```

- **Char Filter** : nettoie le texte avant tokenization (HTML, remplacements)
- **Tokenizer** : découpe en tokens
- **Token Filter** : transforme les tokens (minuscule, stop words, synonymes...)

## Tokenizers

### Standard (défaut)

```bash
POST /_analyze
{
  "tokenizer": "standard",
  "text": "The 3 QUICK Brown-Foxes."
}
# ["The", "3", "QUICK", "Brown", "Foxes"]
```

### Whitespace

```bash
{
  "tokenizer": "whitespace"
}
# Découpe sur les espaces, pas de suppression
```

### Letter

```bash
{
  "tokenizer": "letter"
}
# Uniquement les lettres
```

### Lowercase

```bash
{
  "tokenizer": "lowercase"
}
# + minuscule
```

### Pattern (regex)

```bash
{
  "tokenizer": {
    "my_pattern": {
      "type": "pattern",
      "pattern": "\\s+"
    }
  }
}
```

### Ngram

```bash
{
  "tokenizer": {
    "my_ngram": {
      "type": "ngram",
      "min_gram": 3,
      "max_gram": 3,
      "token_chars": ["letter", "digit"]
    }
  }
}
# "laptop" > ["lap", "apt", "pto"]
```

### Edge ngram

```bash
{
  "tokenizer": {
    "my_edge_ngram": {
      "type": "edge_ngram",
      "min_gram": 2,
      "max_gram": 10,
      "side": "front"
    }
  }
}
# "laptop" > ["la", "lap", "lapt", "lapto", "laptop"]
```

### Path hierarchy

```bash
{
  "tokenizer": {
    "my_path": {
      "type": "path_hierarchy",
      "delimiter": "/"
    }
  }
}
# "/usr/local/bin" > ["/usr", "/usr/local", "/usr/local/bin"]
```

## Char Filters

### HTML strip

```bash
{
  "char_filter": {
    "my_html": {
      "type": "html_strip"
    }
  }
}
```

### Mapping

```bash
{
  "char_filter": {
    "my_mapping": {
      "type": "mapping",
      "mappings": [
        "٠ => 0",
        "١ => 1",
        "& => and"
      ]
    }
  }
}
```

### Pattern replace

```bash
{
  "char_filter": {
    "my_pattern": {
      "type": "pattern_replace",
      "pattern": "(\\d+)-(?=\\d)",
      "replacement": "$1_"
    }
  }
}
```

## Token Filters

### Lowercase / Uppercase

```bash
{
  "filter": {
    "lowercase": { "type": "lowercase" },
    "uppercase": { "type": "uppercase" }
  }
}
```

### Stop words

```bash
{
  "filter": {
    "french_stop": {
      "type": "stop",
      "stopwords": "_french_"
    },
    "custom_stop": {
      "type": "stop",
      "stopwords": ["le", "la", "les", "un", "une", "des"]
    }
  }
}
```

### Stemmer

```bash
{
  "filter": {
    "french_stemmer": {
      "type": "stemmer",
      "language": "french"
    },
    "english_stemmer": {
      "type": "stemmer",
      "language": "english"
    }
  }
}
```

### Snowball

```bash
{
  "filter": {
    "snowball_french": {
      "type": "snowball",
      "language": "French"
    }
  }
}
```

### Synonymes

```bash
{
  "filter": {
    "my_synonym": {
      "type": "synonym",
      "synonyms": [
        "ordinateur, pc, computer",
        "portable => laptop"
      ]
    }
  }
}

# Syntaxe :
# a, b, c          : équivalents
# a => b           : a devient b
# a, b => c        : a et b deviennent c
```

### Synonym graph

```bash
{
  "filter": {
    "my_synonym_graph": {
      "type": "synonym_graph",
      "synonyms": [
        "disque dur, hdd, hard disk"
      ]
    }
  }
}
```

Pour les synonymes multi-mots (obligatoire pour `match_phrase`).

### Synonymes depuis fichier

```bash
{
  "filter": {
    "my_synonym": {
      "type": "synonym",
      "synonyms_path": "analysis/synonyms.txt",
      "updateable": true
    }
  }
}

# synonyms.txt :
ordinateur, pc, computer
portable => laptop
```

### Ascii folding

```bash
{
  "filter": {
    "ascii": {
      "type": "asciifolding"
    }
  }
}
# "Café au lait" > "Cafe au lait"
```

### Shingle

```bash
{
  "filter": {
    "my_shingle": {
      "type": "shingle",
      "min_shingle_size": 2,
      "max_shingle_size": 3,
      "output_unigrams": false
    }
  }
}
# "the quick brown" > ["the quick", "quick brown", "the quick brown"]
```

### Word delimiter

```bash
{
  "filter": {
    "my_delimiter": {
      "type": "word_delimiter",
      "split_on_numerics": true
    }
  }
}
# "wi-fi" > ["wi", "fi"]
```

### Length

```bash
{
  "filter": {
    "my_length": {
      "type": "length",
      "min": 3,
      "max": 20
    }
  }
}
```

### Reverse

```bash
{
  "filter": {
    "reverse": {
      "type": "reverse"
    }
  }
}
```

### Trim

```bash
{
  "filter": {
    "trim": {
      "type": "trim"
    }
  }
}
```

## Analyzer complet

```bash
PUT /products
{
  "settings": {
    "analysis": {
      "char_filter": {
        "html_strip": { "type": "html_strip" }
      },
      "tokenizer": {
        "my_standard": { "type": "standard" }
      },
      "filter": {
        "my_lowercase": { "type": "lowercase" },
        "my_ascii": { "type": "asciifolding" },
        "french_stop": {
          "type": "stop",
          "stopwords": "_french_"
        },
        "french_stemmer": {
          "type": "stemmer",
          "language": "french"
        }
      },
      "analyzer": {
        "french_custom": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "my_standard",
          "filter": [
            "my_lowercase",
            "my_ascii",
            "french_stop",
            "french_stemmer"
          ]
        }
      }
    }
  }
}
```

## Analyzers built-in

| Analyzer | Usage |
|----------|-------|
| `standard` | Général, mots séparés par ponctuation |
| `simple` | Minuscule + ponctuation comme séparateur |
| `whitespace` | Sépare sur whitespace uniquement |
| `stop` | Standard + stop words |
| `keyword` | Pas de tokenization |
| `pattern` | Regex comme séparateur |
| `fingerprint` | Déduplique + trie |
| `language` | `french`, `english`, `spanish`... |

## Analyzer autocomplete

```bash
PUT /autocomplete
{
  "settings": {
    "analysis": {
      "filter": {
        "my_edge_ngram": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 20
        }
      },
      "analyzer": {
        "autocomplete_index": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_edge_ngram"
          ]
        },
        "autocomplete_search": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "autocomplete_index",
        "search_analyzer": "autocomplete_search"
      }
    }
  }
}
```

## Phonetic analyzer

```bash
PUT /names
{
  "settings": {
    "analysis": {
      "analyzer": {
        "phonetic": {
          "tokenizer": "standard",
          "filter": ["phonetic"]
        }
      },
      "filter": {
        "phonetic": {
          "type": "phonetic",
          "encoder": "metaphone"
        }
      }
    }
  }
}
```

Encoders : `metaphone`, `double_metaphone`, `soundex`, `caverphone`

## Tester un analyzer

```bash
# Analyser du texte
GET /_analyze
{
  "analyzer": "standard",
  "text": "Café au lait"
}

# Pour un champ spécifique
GET /products/_analyze
{
  "field": "name",
  "text": "iPhone 15 Pro"
}

# Analyzer custom
GET /_analyze
{
  "tokenizer": "standard",
  "filter": ["lowercase", "stop"],
  "text": "The Quick Brown Fox"
}

# Avec explain
GET /_analyze
{
  "analyzer": "french",
  "text": "Le chat mange",
  "explain": true
}
```

## Normalizer (pour keyword)

```bash
PUT /products
{
  "settings": {
    "analysis": {
      "normalizer": {
        "my_normalizer": {
          "type": "custom",
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "category": {
        "type": "keyword",
        "normalizer": "my_normalizer"
      }
    }
  }
}

# "Électronique" > "electronique"
```

## Built-in normalizers

| Normalizer | Description |
|------------|-------------|
| `lowercase` | Minuscules |
| `uppercase` | Majuscules |
| `asciifolding` | Accents vers ASCII |
| `trim` | Enlève les espaces |
