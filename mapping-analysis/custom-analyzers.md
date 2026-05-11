# Analyzers

## Pipeline

```
Text → Char Filter → Tokenizer → Token Filter → Tokens
```

## Built-in

```bash
// Test
GET /_analyze
{
  "analyzer": "standard",
  "text": "Café au lait"
}

// Types : standard, simple, whitespace, stop, keyword, pattern, fingerprint, language (french, english...)
```

## Custom

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
        "french_stop": { "type": "stop", "stopwords": "_french_" },
        "french_stemmer": { "type": "stemmer", "language": "french" },
        "my_synonym": {
          "type": "synonym_graph",
          "synonyms": ["portable, laptop", "téléphone => phone"]
        }
      },
      "analyzer": {
        "french_custom": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "my_standard",
          "filter": ["my_lowercase", "french_stop", "french_stemmer"]
        }
      }
    }
  }
}
```

## Tokenizers

```bash
"standard"      // Mot par mot
"whitespace"    // Espaces
"letter"        // Lettres
"lowercase"     // Minuscules
"classic"       // Anglais
"ngram"         // n-grams (autocomplete)
"edge_ngram"    // Edge n-grams
"pattern"       // Regex
```

## Token Filters

```bash
"lowercase"         // Minuscules
"uppercase"         // Majuscules
"stop"              // Stop words
"stemmer"           // Lemmatization
"snowball"          // Stemming agressif
"synonym"           // Synonymes
"synonym_graph"     // Multi-word synonyms
"truncate"          // Tronquer
"word_delimiter"    // Découper mots composés
"shingle"           // Combinaisons de mots
"asciifolding"      // accents → ascii
"length"            // Limiter longueur
```

## Normalizer (keyword)

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
```

## Synonyms

```bash
// Inline
{ "synonyms": ["pc, ordinateur", "portable => laptop"] }

// Fichier
{ "synonyms_path": "synonyms.txt", "updateable": true }
```

## Phonetic

```bash
{
  "filter": {
    "phonetic": {
      "type": "phonetic",
      "encoder": "metaphone"  // metaphone, double_metaphone, soundex
    }
  }
}
```

## Récapitulatif

| Composant | Role |
|-----------|-------|
| Char Filter | Pré-traitement caractères |
| Tokenizer | Découpage en tokens |
| Token Filter | Transformation tokens |
| Analyzer | Pipeline complet |
| Normalizer | Pour keyword |
