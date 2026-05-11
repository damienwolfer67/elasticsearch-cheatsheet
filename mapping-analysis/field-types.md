# Field Types

## Core Types

```bash
PUT /products
{
  "mappings": {
    "properties": {
      "id": { "type": "keyword" },
      "name": {
        "type": "text",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      },
      "price": {
        "type": "scaled_float",
        "scaling_factor": 100
      },
      "created": { "type": "date" },
      "active": { "type": "boolean" }
    }
  }
}
```

## String

| Type | Usage | Searchable | Aggregatable |
|------|-------|------------|--------------|
| `keyword` | Valeur exacte | Non | Oui |
| `text` | Full-text | Oui | Non |

## Numeric

| Type | Taille | Usage |
|------|--------|-------|
| `long` | 8 bytes | Compteurs |
| `integer` | 4 bytes | Quantités |
| `short` | 2 bytes | Petits nombres |
| `byte` | 1 byte | 0-255 |
| `scaled_float` | Variable | Prix précis |

## Date

```bash
{ "type": "date" }
// Formats : yyyy-MM-dd, epoch_millis, epoch_second
```

## Object / Nested

```bash
// Object (aplati)
{ "type": "object" }

// Nested (indépendants)
{
  "items": {
    "type": "nested",
    "properties": { ... }
  }
}
```

## Geo

```bash
// Geo point
{ "location": { "type": "geo_point" } }

// Geo shape
{ "area": { "type": "geo_shape" } }
```

## Specialized

```bash
{ "ip": { "type": "ip" } }
{ "suggest": { "type": "completion" } }
{ "join_field": { "type": "join", "relations": { "parent": ["child"] } } }
```

## Field Parameters

```bash
{
  "field": {
    "type": "keyword",
    "doc_values": true,     // Agrégations/tris
    "index": true,          // Searchable
    "null_value": "null",   // Valeur pour null
    "ignore_above": 256,    // Ignorer si >
    "store": false,         // Stocker séparément
    "copy_to": "all"        // Copier vers champ
  }
}
```

## Récapitulatif

| Type | Usage |
|------|-------|
| `keyword` | Filtres, tris, agrégations |
| `text` | Recherche full-text |
| `numeric` | Nombres |
| `date` | Dates |
| `boolean` | Vrai/faux |
| `binary` | Données binaires |
| `object` | Objet simple |
| `nested` | Tableau d'objets |
| `geo_point` | Lat/lon |
| `geo_shape` | Formes géo |
| `ip` | Adresses IP |
| `completion` | Suggestions |
| `join` | Parent-enfant |
