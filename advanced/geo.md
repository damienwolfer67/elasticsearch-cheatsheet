# Géospatial

## Geo Point

```bash
PUT /places
{
  "mappings": {
    "properties": {
      "location": { "type": "geo_point" }
    }
  }
}

// Indexer
PUT /places/_doc/1
{
  "location": { "lat": 48.85, "lon": 2.35 }
}
```

## Distance Query

```bash
GET /places/_search
{
  "query": {
    "geo_distance": {
      "distance": "1km",
      "location": { "lat": 48.85, "lon": 2.35 }
    }
  }
}
```

## Bounding Box

```bash
GET /places/_search
{
  "query": {
    "geo_bounding_box": {
      "location": {
        "top_left": { "lat": 48.88, "lon": 2.28 },
        "bottom_right": { "lat": 48.84, "lon": 2.36 }
      }
    }
  }
}
```

## Geo Distance Sort

```bash
GET /places/_search
{
  "query": { "match_all": {} },
  "sort": [
    {
      "_geo_distance": {
        "location": { "lat": 48.85, "lon": 2.35 },
        "order": "asc",
        "unit": "km"
      }
    }
  ]
}
```

## Geo Distance Aggregation

```bash
GET /places/_search
{
  "size": 0,
  "aggs": {
    "by_distance": {
      "geo_distance": {
        "field": "location",
        "origin": { "lat": 48.85, "lon": 2.35 },
        "ranges": [
          { "to": 1 },
          { "from": 1, "to": 5 },
          { "from": 5 }
        ]
      }
    }
  }
}
```

## Geo Shape

```bash
PUT /maps
{
  "mappings": {
    "properties": {
      "area": { "type": "geo_shape" }
    }
  }
}

// Polygon
PUT /maps/_doc/1
{
  "area": {
    "type": "polygon",
    "coordinates": [[
      [2.28, 48.88], [2.36, 48.88],
      [2.36, 48.84], [2.28, 48.84],
      [2.28, 48.88]
    ]]
  }
}

// Query
GET /maps/_search
{
  "query": {
    "geo_shape": {
      "area": {
        "shape": {
          "type": "circle",
          "radius": "1km",
          "coordinates": [2.35, 48.85]
        },
        "relation": "intersects"
      }
    }
  }
}
```

## Units

`km`, `m`, `cm`, `mm`, `mi` (miles), `yd` (yards), `ft` (feet), `in` (inches), `nmi` (nautical)
