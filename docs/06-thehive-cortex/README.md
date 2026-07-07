# 🐝 TheHive 5 & Cortex

> Module SOC — TheHive est la plateforme de gestion des incidents de sécurité. Cortex est le moteur d'analyse et de réponse automatique.

> Prérequis matériels : voir [docs/01-prerequis.md](../01-prerequis.md). Installation de Docker : voir [docs/02-installation-docker.md](../02-installation-docker.md).

---

## Étape 1 — Préparer le répertoire projet

```bash
mkdir -p ~/soc-project/cortex/logs
touch ~/soc-project/cortex/application.conf
cd ~/soc-project
```

### Générer les secrets

Ces valeurs ne doivent **jamais** être écrites en dur : génère-les et place-les dans un fichier `.env` (voir [.env.example](.env.example)), qui reste hors de git.

```bash
openssl rand -hex 32   # → CORTEX_SECRET_KEY
openssl rand -hex 32   # → THEHIVE_SECRET_KEY
```

```bash
cp .env.example ~/soc-project/.env
# édite ~/soc-project/.env avec les valeurs générées ci-dessus
```

### Créer la configuration Cortex

```bash
cat > ~/soc-project/cortex/application.conf << 'EOF'
play.http.secret.key = "${CORTEX_SECRET_KEY}"

search {
  index = cortex
  uri = "http://elasticsearch:9200"
}

cache.job = 10 minutes

job {
  timeout = 30 minutes
  directory = /tmp/cortex-jobs
}

analyzer {
  urls = [
    "/opt/Cortex-Analyzers/analyzers"
  ]
}
EOF
```

> ⚠️ Play Framework ne substitue pas automatiquement `${VAR}` dans `application.conf` sans configuration supplémentaire. Le plus simple : remplace `${CORTEX_SECRET_KEY}` par la vraie valeur générée juste avant de lancer `docker compose up -d`, ou utilise `envsubst` :
> ```bash
> envsubst < application.conf.template > ~/soc-project/cortex/application.conf
> ```

---

## Étape 2 — Déployer TheHive & Cortex

### Copier le fichier docker-compose.yml

> Récupère le fichier `docker-compose.yml` fourni dans ce dépôt et place-le dans `~/soc-project/`, à côté de ton fichier `.env`.

### Lancer les conteneurs

```bash
cd ~/soc-project
docker compose up -d
```

### Vérifier que tous les conteneurs sont bien démarrés

```bash
docker compose ps
```

Tu dois voir `Up` pour tous les services :

| Conteneur | Port |
|---|---|
| thehive | 9000 |
| cortex.local | 9001 |
| minio | 9002 |
| elasticsearch | 9200 |
| cassandra | 9042 |

---

## Étape 3 — Attendre l'initialisation

> ⚠️ Cassandra et Elasticsearch prennent **3 à 5 minutes** à démarrer complètement. TheHive attend qu'ils soient prêts avant d'ouvrir son interface.

Surveille les logs TheHive en direct :

```bash
docker compose logs -f thehive
```

Quand tu vois `Application started`, l'interface est disponible.

---

## Étape 4 — Accéder aux interfaces

| Service | URL | Identifiants |
|---|---|---|
| TheHive | `http://IP_VM:9000` | `admin@thehive.local` / `secret` *(identifiant par défaut — à changer immédiatement après la première connexion)* |
| Cortex | `http://IP_VM:9001` | Créer un compte au premier accès |
| Minio | `http://IP_VM:9002` | valeurs définies dans `.env` (`MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`) |

Pour trouver l'IP de ta VM :

```bash
hostname -I
```

---

## Étape 5 — Intégrer Cortex avec TheHive

### Comment les deux se combinent (le mécanisme, pas juste les clics)

TheHive et Cortex sont **deux applications indépendantes** qui ne communiquent que par API REST — il n'y a pas de base de données partagée. Le lien se fait dans un seul sens : **TheHive appelle Cortex**, jamais l'inverse.

```
TheHive (Case + observable "1.2.3.4")
        │
        │  1. L'utilisateur (ou soc-automation) clique "Run analyzers"
        │     sur l'observable, ou l'API POST /connector/cortex/job
        ▼
Cortex (reçoit l'IP, exécute VirusTotal_GetReport + AbuseIPDB)
        │
        │  2. Cortex exécute les scripts Python des analyzers
        │     (voir cortex-analyzers.md), interroge les APIs externes
        ▼
Cortex renvoie un rapport JSON par analyzer (score, taxonomies...)
        │
        │  3. TheHive récupère le résultat via polling sur le job Cortex
        │     et l'attache à l'observable dans le Case
        ▼
TheHive affiche les rapports sous l'observable
```

Concrètement, **3 conditions** doivent être réunies pour que ça marche :
1. TheHive doit connaître l'adresse et une **clé API valide** de Cortex (configuré une fois, ci-dessous).
2. Les analyzers doivent être **activés côté Cortex** avec leurs propres clés API externes (VirusTotal, AbuseIPDB) — voir [cortex-analyzers.md](cortex-analyzers.md).
3. Le compte TheHive qui déclenche l'analyse doit avoir les droits `analyze` — c'est pour ça que l'organisation dédiée du script a besoin du bon profil, voir [organisation-setup.md](organisation-setup.md).

`soc-automation/response_soc.py` automatise l'étape 1 (déclenche l'analyse dès qu'un case est créé) au lieu de cliquer manuellement sur "Run analyzers".

### Configuration (à faire une fois)

1. Connecte-toi à **Cortex** (`http://IP_VM:9001`)
2. Crée un compte administrateur
3. Crée une **Organisation**
4. Crée un **utilisateur** avec le rôle `read, analyze, orgadmin`
5. Génère une **clé API** pour cet utilisateur
6. Connecte-toi à **TheHive** (`http://IP_VM:9000`)
7. Aller dans **Administration → Cortex**
8. Ajouter le serveur Cortex : `http://cortex.local:9001` (nom du service Docker, pas `localhost` — les deux conteneurs se joignent via le réseau `SOC_NET` du [docker-compose.yml](docker-compose.yml))
9. Coller la clé API générée à l'étape 5

Ensuite, active les analyzers eux-mêmes : voir le tutoriel complet **[cortex-analyzers.md](cortex-analyzers.md)** (installation des dépendances, clés API VirusTotal/AbuseIPDB, et surtout **comment vérifier que ça fonctionne vraiment** avant de compter dessus).

---

## Commandes utiles

```bash
# Voir l'état de tous les conteneurs
docker compose ps

# Voir les logs d'un service
docker compose logs -f thehive
docker compose logs -f cortex.local
docker compose logs -f elasticsearch
docker compose logs -f cassandra

# Redémarrer un service
docker compose restart thehive

# Arrêter tous les conteneurs
docker compose down

# Redémarrage propre complet
docker compose down && docker compose up -d
```

---

## Dépannage rapide

### Elasticsearch crash au démarrage

```bash
sudo sysctl -w vm.max_map_count=262144
docker compose restart elasticsearch
```

### TheHive inaccessible après démarrage

```bash
# Attendre que Cassandra soit prêt (3-5 min) puis vérifier
docker compose logs --tail=30 cassandra
docker compose logs --tail=30 thehive
```

### Tester si les ports répondent

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:9000
curl -s -o /dev/null -w "%{http_code}" http://localhost:9001
curl -s http://localhost:9200
```

---

## Architecture des composants

```
TheHive 5
  ├── Cassandra    → Base de données principale
  ├── Elasticsearch → Moteur de recherche et indexation
  ├── Minio        → Stockage des fichiers (pièces jointes, rapports)
  └── Cortex       → Analyse automatique + réponse aux incidents
        ├── Analyzers  → VirusTotal, AbuseIPDB, Shodan...
        └── Responders → Blocage IP, isolation host, alertes...
```

---

## Documentation officielle

- [TheHive 5 — documentation officielle](https://docs.strangebee.com/thehive/)
- [Cortex — documentation officielle](https://docs.strangebee.com/cortex/)
- [Cortex-Analyzers — dépôt GitHub (analyzers/responders)](https://github.com/TheHive-Project/Cortex-Analyzers)
- [Docker Compose — référence](https://docs.docker.com/compose/)

## Auteur

**12ak_H4ck** — Projet académique ESIG · Blue Team / SOC
