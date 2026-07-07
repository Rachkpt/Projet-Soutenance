# Wazuh Active Response

## Pourquoi Active Response plutôt que le blocage local du script

Le SOC bloque les IPs malveillantes de deux façons :

1. **Blocage local via `iptables`**, exécuté par `surveillance_soc.py` lui-même — utilisé pour les catégories d'alertes non couvertes nativement par Wazuh.
2. **Wazuh Active Response** (`firewall-drop`) — méthode **préférée**, car elle s'exécute **directement sur l'agent Wazuh attaqué**, et protège donc la vraie machine ciblée. Le blocage local du script, lui, ne protège que le serveur qui héberge le script d'automatisation — pas la machine réellement visée par l'attaquant si elle est différente.

📖 Doc officielle — présentation générale : [Active Response overview](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html)

## Réponses actives utilisées

| Réponse | Usage | Doc officielle |
|---|---|---|
| `firewall-drop` | Bloque l'IP source sur l'agent attaqué (règle firewall locale à l'agent) | [Default active response scripts](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/default-active-response-scripts.html) |
| `remove-threat` | Suppression d'un fichier identifié comme malveillant (lié au FIM + VirusTotal, voir [FIM](fim.md)) | [Default active response scripts](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/default-active-response-scripts.html) |

## Règles déclenchant `firewall-drop`

Confirmé dans le code du script (`soc_config.py`, `soc_clients.py`) — ces `rules_id` sont bloqués **nativement côté Wazuh Manager**, donc exclus du blocage local du script (voir `NATIVE_AR_CATEGORIES` dans [classification-alertes.md](../08-automatisation-soc/classification-alertes.md)) :

| `rules_id` | Catégorie |
|---|---|
| `5710`, `5712`, `5716`, `5720` | Brute force SSH |
| `86601` | Scans réseau détectés via Suricata (décodeur Wazuh intégré) |

## Configuration (`ossec.conf`)

> ⚠️ **À compléter** avec le bloc `<active-response>` exact déployé dans `/var/ossec/etc/ossec.conf` (`<timeout>`, agents ciblés). Squelette attendu, avec les `rules_id` confirmés ci-dessus :

```xml
<active-response>
  <disabled>no</disabled>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5710,5712,5716,5720,86601</rules_id>
  <timeout>600</timeout>
</active-response>
```

📖 Syntaxe complète du bloc `<active-response>` : [Active response reference (ossec.conf)](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/active-response.html)

## Documentation officielle

- [Active Response — vue d'ensemble](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html)
- [Active Response — référence de configuration `ossec.conf`](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/active-response.html)
- [Scripts de réponse par défaut (`firewall-drop`, `remove-threat`...)](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/default-active-response-scripts.html)
- [Cas d'usage — bloquer un brute force SSH](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/ar-use-cases/blocking-ssh-brute-force.html)
