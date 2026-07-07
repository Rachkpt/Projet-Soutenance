# Cortex Analyzers

## Ce qu'il faut savoir

Les analyzers Cortex (VirusTotal, AbuseIPDB, etc.) sont des **scripts Python exécutés directement dans le conteneur Cortex**. Contrairement à ce qu'on pourrait penser, les installer sur l'hôte ne suffit pas : ils nécessitent que **python3 et les dépendances de chaque analyzer** (fichier `requirements.txt` propre à chaque analyzer) soient installés **dans le conteneur** `cortex.local`.

> Sans cette étape, l'exécution d'un analyzer échoue avec `ModuleNotFoundError`.

## Installer les dépendances d'un analyzer dans le conteneur

```bash
# Entrer dans le conteneur Cortex
docker exec -it cortex.local bash

# Aller dans le dossier de l'analyzer concerné (ex. VirusTotal)
cd /opt/Cortex-Analyzers/analyzers/VirusTotal

# Installer python3 si absent
apt update && apt install -y python3 python3-pip

# Installer les dépendances de l'analyzer
pip3 install -r requirements.txt
```

À répéter pour **chaque analyzer activé** (VirusTotal, AbuseIPDB, ...).

> ⚠️ Ces modifications sont faites *dans* le conteneur — si le conteneur est recréé (`docker compose down` puis `up`, ou changement d'image), il faut les refaire, sauf si `/opt/Cortex-Analyzers` est monté depuis l'hôte (c'est le cas ici, voir [docker-compose.yml](docker-compose.yml) — volume `/opt/Cortex-Analyzers:/opt/Cortex-Analyzers`), auquel cas seule la partie `apt install python3` côté conteneur est à refaire si le conteneur change.

## Activer un analyzer dans Cortex

1. Se connecter à Cortex (`http://IP_VM:9001`) avec un compte `orgadmin`.
2. Aller dans **Organization → Analyzers**.
3. Chercher l'analyzer (ex. `VirusTotal_GetReport`) → **Create**.
4. Renseigner la clé API nécessaire (ex. `key` pour VirusTotal).
5. Sauvegarder.

## Analyzers utilisés dans ce projet

> À compléter avec la liste exacte des analyzers activés et leurs clés API (VirusTotal, AbuseIPDB confirmés dans l'architecture — voir [00-architecture.md](../00-architecture.md)).
