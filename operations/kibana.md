# Kibana

## Installation

```bash
# Docker
docker run -p 5601:5601 \
  -e ELASTICSEARCH_HOSTS=http://elasticsearch:9200 \
  kibana:8.x.x

# APT
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | \
  sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] \
  https://artifacts.elastic.co/packages/8.x/apt stable main" | \
  sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update && sudo apt install kibana
```

## Config (kibana.yml)

```yaml
server.host: "0.0.0.0"
server.port: 5601
elasticsearch.hosts: ["http://localhost:9200"]

# Sécurité
elasticsearch.username: "kibana_system"
elasticsearch.password: "pass"

# I18N
i18n.locale: "fr"
```

## Outils principaux

| Outil | Usage |
|-------|-------|
| **Discover** | Explorer et rechercher les données |
| **Console** | Exécuter des requêtes Elasticsearch |
| **Dev Tools** | Console + autres outils dev |
| **Index Management** | Gérer les index et ILM |
| **Stack Monitoring** | Monitoring du cluster |
| **Security** | Gérer utilisateurs/rôles |
| **Index Patterns** | Définir des patterns d'index |

## Console Dev Tools

Dans Dev Tools > Console, tu peux exécuter directement :

```bash
# Health check
GET /

# Créer un index
PUT /products
{
  "mappings": {
    "properties": {
      "name": { "type": "text" }
    }
  }
}

# Rechercher
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
```

La console supporte l'autocomplétion avec `Ctrl + Espace`.

## Discover

Pour explorer tes données :

1. Crée un **Index Pattern** (ex: `logs-*`)
2. Va dans **Discover**
3. Choisit ton index pattern
4. Recherche avec KQL (Kibana Query Language) ou Lucene

### KQL basics

```
# Recherche simple
response: 200

# Avec champ spécifique
status: "active"

# Combinaisons
response: 200 and method: "GET"
response: >= 400 and message: "error"

# Wildcards
user-agent: "*chrome*"

# Plage de temps
@timestamp >= "2024-01-01" and @timestamp < "2024-02-01"
```

## Index Management

```kql
# Dans Stack Management > Index Management
```

Tu peux :
- Voir tous les index et leurs stats
- Gérer les ILM policies
- Faire des rollovers manuels
- Supprimer/fermer des index

## Visualisations et Dashboards

Dans **Visualize Library** :
- Crée des graphiques (line, bar, pie, metric...)
- Basés sur des requêtes Elasticsearch
- Sauvegarde les visualisations

Dans **Dashboard** :
- Combine plusieurs visualisations
- Ajoute des filtres globaux
- Partage ton dashboard

## Machine Learning

```bash
# Pour créer des jobs d'anomaly detection
# Stack Management > ML Jobs
```

Kibana ML permet :
- Détection d'anomalies sur des métriques temps réel
- Prévisions (forecasting)
- Classification

## Alerting

```bash
# Stack Management > Rules and Connectors
```

Crée des alertes basées sur :
- Conditions de requête
- Thresholds
- Anomalies ML

## Security

```bash
# Stack Management > Security
```

- **Users** : Gérer les utilisateurs
- **Roles** : Définir les permissions
- **API Keys** : Générer des clés pour les apps

## Monitoring

```bash
# Stack Monitoring
```

Vue d'ensemble sur :
- Health du cluster
- Performance des nœuds
- Indices et shards
- JVM et OS

## Tips

```bash
# Pour accéder à Kibana
http://localhost:5601

# Utiliser Console pour tester les requêtes avant de les mettre dans ton code
# Les requêtes dans Console sont auto-formatées avec Ctrl + Shift + F
```
