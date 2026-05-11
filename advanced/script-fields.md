# Scripting Painless

## Script Fields

```bash
GET /products/_search
{
  "query": { "match_all": {} },
  "script_fields": {
    "with_tax": {
      "script": {
        "source": "doc.price.value * 1.2"
      }
    }
  }
}
```

## Script Score

```bash
GET /products/_search
{
  "query": {
    "script_score": {
      "query": { "match": { "name": "laptop" } },
      "script": {
        "source": "doc.popularity.value * params.factor",
        "params": { "factor": 2 }
      }
    }
  }
}
```

## Update Script

```bash
POST /products/_update/1
{
  "script": {
    "source": "ctx._source.price += params.inc",
    "params": { "inc": 50 }
  }
}
```

## Stored Script

```bash
POST _scripts/calc-tax
{
  "script": {
    "lang": "painless",
    "source": "doc.price.value * params.tax"
  }
}

POST /products/_update/1
{
  "script": {
    "id": "calc-tax",
    "params": { "tax": 1.2 }
  }
}
```

## Syntaxe

```bash
// Accès champs
doc.price.value
doc['name.keyword'].value

// Test existence
doc.containsKey('price')

// Fonctions
Math.random(), Math.sqrt(), Math.abs()
s.length(), s.toLowerCase(), s.startsWith(x)

// Conditionnelles
if (condition) { ... } else { ... }
value > 100 ? "expensive" : "cheap"

// Boucles
for (int i = 0; i < 10; i++) { ... }
for (def item : list) { ... }
```

## Update Ops

```bash
ctx._source.field = value
ctx._source.remove('field')
ctx.op = 'noop'      // Ne rien faire
ctx.op = 'delete'    // Supprimer
```

## Runtime Fields

```bash
GET /products/_search
{
  "runtime_mappings": {
    "price_with_tax": {
      "type": "double",
      "script": { "source": "emit(doc['price'].value * 1.2)" }
    }
  },
  "query": {
    "range": { "price_with_tax": { "lte": 1000 } }
  }
}
```

## Best Practices

- ✅ Utiliser `params` pour performance
- ✅ Préférer `doc.field.value` à `ctx._source.field`
- ❌ Éviter scripts complexes dans la requête
- ⚠️ Pas d'accès réseau/IO
