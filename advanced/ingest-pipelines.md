# Ingest Pipelines

Les pipelines transforment les documents avant leur indexation.

## Créer un pipeline

```bash
PUT _ingest/pipeline/compute-timestamp
{
  "processors": [
    {
      "set": {
        "field": "@timestamp",
        "value": "{{_ingest.timestamp}}"
      }
    }
  ]
}
```

## Processors courants

### Set / Remove

```bash
# Set un champ
PUT _ingest/pipeline/add-timestamp
{
  "processors": [
    {
      "set": {
        "field": "created_at",
        "value": "{{_ingest.timestamp}}"
      }
    }
  ]
}

# Supprimer un champ
{
  "processors": [
    {
      "remove": {
        "field": "temporary_field"
      }
    }
  ]
}
```

### Append / Rename

```bash
# Ajouter à un tableau
{
  "processors": [
    {
      "append": {
        "field": "tags",
        "value": "processed"
      }
    }
  ]
}

# Renommer
{
  "processors": [
    {
      "rename": {
        "field": "old_name",
        "target_field": "new_name"
      }
    }
  ]
}
```

### Date

```bash
# Parser une date
{
  "processors": [
    {
      "date": {
        "field": "timestamp",
        "target_field": "@timestamp",
        "formats": ["ISO8601", "UNIX_MS"]
      }
    }
  ]
}
```

### Grok

```bash
# Extraire des patterns avec grok
{
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:msg}"]
      }
    }
  ]
}

# Logs Apache
{
  "grok": {
    "field": "message",
    "patterns": ["%{COMMONAPACHELOG}"]
  }
}
```

### User Agent

```bash
# Parser user-agent
{
  "processors": [
    {
      "user_agent": {
        "field": "user_agent"
      }
    }
  ]
}
```

### GeoIP

```bash
# Enrichir avec geoIP
{
  "processors": [
    {
      "geoip": {
        "field": "ip_address",
        "target_field": "geo"
      }
    }
  ]
}
```

### Convert

```bash
# Convertir un type
{
  "processors": [
    {
      "convert": {
        "field": "price",
        "type": "float"
      }
    }
  ]
}
```

### Lowercase / Uppercase

```bash
{
  "processors": [
    {
      "lowercase": {
        "field": "email"
      }
    },
    {
      "uppercase": {
        "field": "status_code"
      }
    }
  ]
}
```

### Split

```bash
# Découper une chaîne
{
  "processors": [
    {
      "split": {
        "field": "tags",
        "separator": ","
      }
    }
  ]
}
```

### Join

```bash
# Joindre un tableau en string
{
  "processors": [
    {
      "join": {
        "field": "tags",
        "separator": "; "
      }
    }
  ]
}
```

### Script

```bash
# Script custom
{
  "processors": [
    {
      "script": {
        "source": "ctx.processed = true; ctx.timestamp = new Date().toString()"
      }
    }
  ]
}
```

### Fail / Conditional

```bash
# Échouer si condition
{
  "processors": [
    {
      "fail": {
        "if": "ctx.status == 'error'",
        "message": "Status is error"
      }
    }
  ]
}

# Processor conditionnel
{
  "processors": [
    {
      "set": {
        "if": "ctx.price > 1000",
        "field": "category",
        "value": "expensive"
      }
    }
  ]
}
```

## Pipeline avec plusieurs processors

```bash
PUT _ingest/pipeline/normalize-log
{
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:msg}"]
      }
    },
    {
      "date": {
        "field": "timestamp",
        "target_field": "@timestamp"
      }
    },
    {
      "convert": {
        "field": "level",
        "target_field": "level_code",
        "type": "integer"
      }
    },
    {
      "remove": {
        "field": "message"
      }
    }
  ]
}
```

## Utiliser un pipeline

```bash
# Avec index
POST /logs/_doc?pipeline=normalize-log
{
  "message": "2024-01-15T10:30:00Z INFO Application started"
}

# Avec bulk
POST /_bulk
{ "index": { "_index": "logs", "pipeline": "normalize-log" } }
{ "message": "2024-01-15T10:30:00Z INFO Test message" }

# Par défaut sur un index
PUT /logs/_settings
{
  "index.default_pipeline": "normalize-log"
}
```

## Pipeline dans reindex

```bash
POST _reindex
{
  "source": { "index": "logs_raw" },
  "dest": {
    "index": "logs",
    "pipeline": "normalize-log"
  }
}
```

## On failure

```bash
# Gérer les erreurs
PUT _ingest/pipeline/safe-pipeline
{
  "processors": [
    {
      "convert": {
        "field": "price",
        "type": "float",
        "ignore_failure": true,
        "on_failure": [
          {
            "set": {
              "field": "error",
              "value": "Could not convert price"
            }
          }
        ]
      }
    }
  ]
}
```

## Enrichment

```bash
# Enrichir avec une policy
PUT _enrich/policy/users-policy
{
  "match": {
    "indices": "users",
    "match_field": "user_id",
    "enrich_fields": ["email", "department"]
  }
}

PUT _enrich/policy/users-policy/_execute

# Dans un pipeline
{
  "processors": [
    {
      "enrich": {
        "policy_name": "users-policy",
        "field": "user_id",
        "target_field": "user_info"
      }
    }
  ]
}
```

## Pipeline avancé

```bash
# Pipeline complexe pour logs web
PUT _ingest/pipeline/web-logs
{
  "description": "Parse web server logs",
  "processors": [
    {
      "grok": {
        "field": "message",
        "patterns": ["%{COMMONAPACHELOG}"],
        "ignore_failure": true
      }
    },
    {
      "user_agent": {
        "field": "agent",
        "ignore_failure": true
      }
    },
    {
      "geoip": {
        "field": "clientip",
        "target_field": "geo",
        "ignore_failure": true
      }
    },
    {
      "date": {
        "field": "timestamp",
        "target_field": "@timestamp"
      }
    },
    {
      "convert": {
        "field": "response",
        "type": "integer",
        "target_field": "response_code"
      }
    },
    {
      "script": {
        "if": "ctx.response_code >= 400",
        "source": "ctx.error = true"
      }
    }
  ]
}
```

## Gérer les pipelines

```bash
# Lister
GET _ingest/pipeline

# Voir un pipeline
GET _ingest/pipeline/normalize-log

# Simuler un pipeline
POST _ingest/pipeline/normalize-log/_simulate
{
  "docs": [
    {
      "_source": {
        "message": "2024-01-15T10:30:00Z INFO Test"
      }
    }
  ]
}

# Supprimer
DELETE _ingest/pipeline/normalize-log
```
