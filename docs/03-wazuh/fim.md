# File Integrity Monitoring (FIM)

## Rôle dans le pipeline

Le FIM Wazuh (module `syscheck`) surveille des fichiers/répertoires sur les agents. Les événements (création, modification, suppression) sont récupérés par `surveillance_soc.py` via `WazuhClient.get_fim_events()`, qui interroge l'[Indexer](../08-automatisation-soc/source-des-donnees.md) sur les événements `syscheck` des dernières minutes uniquement (jamais l'historique complet, pour éviter le flood).

## Traitement (`process_fim_event` dans `surveillance_soc.py`)

| Type d'événement | Action |
|---|---|
| Fichier **créé** | Notification Telegram + analyse automatique (voir ci-dessous) |
| Fichier **modifié** | Notification Telegram (avec hash avant/après si disponible) + analyse automatique |
| Fichier **supprimé** | Notification Telegram uniquement (pas d'analyse possible, plus de fichier) |

## Analyse automatique (`_fim_auto_analyze`)

Pour un fichier créé ou modifié **avec un hash SHA256 exploitable** :

1. Un **Case TheHive** est créé automatiquement (`thehive.create_fim_case()`), avec le hash en observable.
2. Le case est mis en file pour `response_soc.py`, qui lance les analyzers Cortex (VirusTotal...) et synthétise le résultat via Groq.
3. Le résultat final est renvoyé sur Telegram.

## Anti-spam spécifique au FIM

Deux niveaux de throttle indépendants (1h chacun, via `should_alert()`) :
- **Notification** (`fim_notif:{agent}:{file}:{action}`) : évite de spammer Telegram si un fichier change en boucle (ex. log applicatif réécrit en continu).
- **Analyse** (`fim_analyze:{agent}:{file}`) : évite de recréer un case TheHive à chaque cycle pour le même fichier.

## Lien avec Active Response

Si l'analyse Cortex/Groq identifie le fichier comme malveillant, la suite du traitement peut relever de `remove-threat` côté Wazuh — voir [Active Response](active-response.md) et la catégorie `THREAT_REMOVED` dans [classification-alertes.md](../08-automatisation-soc/classification-alertes.md).

## Configuration Wazuh (`<syscheck>`)

> ⚠️ **À compléter** avec la configuration réelle du bloc `<syscheck>` dans `ossec.conf` (répertoires surveillés, fréquence de scan, `whodata`/`realtime` activé ou non).

## Documentation officielle

- [Wazuh FIM](https://documentation.wazuh.com/current/proof-of-concept-guide/index.html)
