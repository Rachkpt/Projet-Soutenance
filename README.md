# 🛡️ SOC Automatisé — Détection et Réponse aux Incidents de Sécurité

> Projet académique ESIG (Master Cybersécurité) — SOC complet basé sur Wazuh, Suricata, Fail2ban, TheHive 5, Cortex, Grafana/Prometheus, et un script Python d'automatisation (surveillance, réponse, bot Telegram).

## Structure du dépôt

- [`docs/`](docs/) — documentation complète d'installation et de configuration de chaque brique du SOC.
- [`soc-automation/`](soc-automation/) — code Python du script d'automatisation (le cœur du projet).

## Documentation

| # | Sujet |
|---|---|
| [00](docs/00-architecture.md) | Architecture globale |
| [01](docs/01-prerequis.md) | Prérequis matériels et logiciels |
| [02](docs/02-installation-docker.md) | Installation Docker |
| [03](docs/03-wazuh/README.md) | Wazuh (SIEM/EDR) — [FIM](docs/03-wazuh/fim.md), [intégration VirusTotal](docs/03-wazuh/virustotal-integration.md), [Active Response](docs/03-wazuh/active-response.md) |
| [04](docs/04-suricata/README.md) | Suricata (IDS/IPS) |
| [05](docs/05-fail2ban/README.md) | Fail2ban / détection de scan de ports |
| [06](docs/06-thehive-cortex/README.md) | TheHive & Cortex — [organisation](docs/06-thehive-cortex/organisation-setup.md), [analyzers](docs/06-thehive-cortex/cortex-analyzers.md) |
| [07](docs/07-monitoring/README.md) | Monitoring (Grafana / Prometheus) |
| [08](docs/08-automatisation-soc/README.md) | Script Python d'automatisation SOC |
| [09](docs/09-depannage-general.md) | Dépannage général |

## Stack technique

| Outil | Rôle |
|---|---|
| **Wazuh** | SIEM / EDR — détection sur les endpoints |
| **Suricata** | IDS/IPS réseau |
| **Fail2ban** | Protection contre force brute et scans de ports |
| **TheHive 5** | Gestion des incidents de sécurité |
| **Cortex** | Analyse automatique (VirusTotal, AbuseIPDB...) et réponse |
| **Grafana / Prometheus** | Monitoring et dashboards temps réel |
| **Script Python (`soc-automation/`)** | Corrélation, classification, réponse automatique, bot Telegram |

## Auteur

**12ak_H4ck** — Projet académique ESIG · Blue Team / SOC
