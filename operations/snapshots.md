# Sauvegardes (Snapshot/Restore)

## Repository

Un repository stocke tes snapshots (FS, S3, GCS, Azure...).

### File system

```bash
# Dans elasticsearch.yml
path.repo: ["/mount/backups", "/mount/longterm_backups"]

# Créer le repository
PUT /_snapshot/backup
{
  "type": "fs",
  "settings": {
    "location": "/mount/backups"
  }
}
```

### S3

```bash
PUT /_snapshot/s3_backup
{
  "type": "s3",
  "settings": {
    "bucket": "my-es-backups",
    "region": "eu-west-3",
    "base_path": "elasticsearch"
  }
}
```

### GCS

```bash
PUT /_snapshot/gcs_backup
{
  "type": "gcs",
  "settings": {
    "bucket": "my-es-backups",
    "base_path": "elasticsearch"
  }
}
```

### Voir les repositories

```bash
GET /_snapshot
GET /_snapshot/backup
```

### Supprimer un repository

```bash
DELETE /_snapshot/backup
```

## Snapshot

### Créer un snapshot

```bash
# Tous les indices
PUT /_snapshot/backup/snapshot_1

# Certains indices
PUT /_snapshot/backup/snapshot_2
{
  "indices": "products,orders",
  "ignore_unavailable": true,
  "include_global_state": false
}

# Avec wait for completion
PUT /_snapshot/backup/snapshot_3?wait_for_completion=true
```

### Paramètres utiles

| Paramètre | Description |
|-----------|-------------|
| `indices` | Indices à sauvegarder (ex: `logs-*,products`) |
| `ignore_unavailable` | Ignore les indices manquants |
| `include_global_state` | Inclut le cluster state |
| `metadata` | Metadata du snapshot |
| `wait_for_completion` | Attend que le snapshot finisse |

### Lister les snapshots

```bash
GET /_snapshot/backup/_all
GET /_snapshot/backup/snapshot_*
GET /_snapshot/backup/snapshot_1
```

### Supprimer un snapshot

```bash
DELETE /_snapshot/backup/snapshot_1
```

### Snapshot incrémental

Les snapshots sont incrémentaux par défaut - seul ce qui a changé depuis le dernier snapshot est stocké. Elasticsearch utilise les segments de Lucene pour optimiser.

## Restore

### Restaurer un snapshot

```bash
# Tous les indices
POST /_snapshot/backup/snapshot_1/_restore

# Certains indices
POST /_snapshot/backup/snapshot_2/_restore
{
  "indices": "products,orders",
  "ignore_unavailable": true,
  "include_global_state": false
}

# Renommer à la restauration
POST /_snapshot/backup/snapshot_1/_restore
{
  "indices": "products",
  "rename_pattern": "products_(.+)",
  "rename_replacement": "restored_products_$1"
}
```

### Paramètres utiles

| Paramètre | Description |
|-----------|-------------|
| `indices` | Indices à restaurer |
| `ignore_unavailable` | Ignore les indices manquants |
| `include_global_state` | Restore le cluster state |
| `rename_pattern` | Regex pour renommer |
| `rename_replacement` | Pattern de remplacement |
| `index_settings` | Override des settings d'index |

### Restaurer dans un autre cluster

```bash
# Le repository doit être accessible des deux clusters
POST /_snapshot/shared_backup/snapshot_1/_restore
{
  "indices": "products",
  "include_aliases": false
}
```

## Status et monitoring

```bash
# Statut d'un snapshot en cours
GET /_snapshot/backup/snapshot_1

# Annuler un snapshot
DELETE /_snapshot/backup/snapshot_1

# Statut d'une restore en cours
GET /_restore/snapshot_1
```

## SLM (Snapshot Lifecycle Management)

Automatise les snapshots.

### Créer une policy SLM

```bash
PUT _slm/policy/daily_snapshots
{
  "schedule": "0 2 * * *",
  "name": "<daily-snapshot-{now/d}>",
  "repository": "backup",
  "config": {
    "indices": "*",
    "ignore_unavailable": false,
    "include_global_state": false
  },
  "retention": {
    "expire_after": "30d",
    "min_count": 5,
    "max_count": 50
  }
}
```

### Exécuter une policy maintenant

```bash
POST _slm/policy/daily_snapshots/_execute
```

### Voir les policies

```bash
GET _slm/policy
GET _slm/policy/daily_snapshots
```

### Stats SLM

```bash
GET _slm/stats
```

### Supprimer une policy

```bash
DELETE _slm/policy/daily_snapshots
```

## Best practices

```bash
# 1. Utiliser un repository externe (S3, GCS)
# 2. Tester les restores régulièrement
# 3. Mettre en place SLM pour automatiser
# 4. Garder au moins quelques snapshots
# 5. Documenter la procédure de restore

# Exemple de policy complète
PUT _slm/policy/production_backup
{
  "schedule": "0 3 * * *",
  "name": "prod-snapshot-{now/M}-{now/d}>",
  "repository": "s3_production",
  "config": {
    "indices": "logs-*,products-*",
    "ignore_unavailable": true,
    "include_global_state": false
  },
  "retention": {
    "expire_after": "90d",
    "min_count": 7,
    "max_count": 30
  }
}
```

## Restore compliqué

```bash
# Restore sélectif avec modification de settings
POST /_snapshot/backup/snapshot_1/_restore
{
  "indices": "products",
  "rename_pattern": "products",
  "rename_replacement": "restored_products",
  "index_settings": {
    "index.number_of_replicas": 0,
    "index.refresh_interval": "5s"
  },
  "ignore_index_settings": ["index.profiles"]
}
```
