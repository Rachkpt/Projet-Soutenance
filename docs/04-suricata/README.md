# 🚨 Suricata — IDS/IPS réseau

> Module SOC — détection d'intrusion réseau. Page minimale pour l'instant : à enrichir avec la configuration réelle du projet (règles activées, mode IDS vs IPS, intégration Wazuh).

## Installation

📖 Doc officielle installation : [Suricata — Installation](https://docs.suricata.io/en/latest/install.html)

```bash
sudo apt install -y suricata
```

### Télécharger les règles

📖 Doc officielle : [Rule management — suricata-update](https://docs.suricata.io/en/latest/rule-management/suricata-update.html)

```bash
sudo suricata-update
```

### Démarrer le service

```bash
sudo systemctl enable --now suricata
sudo systemctl status suricata
```

### Vérifier les alertes en temps réel

📖 Format des logs : [fast.log format](https://docs.suricata.io/en/latest/output/log-file-formats.html) · [eve.json format](https://docs.suricata.io/en/latest/output/eve/eve-json-format.html) (utilisé par `surveillance_soc.py`, voir [source-des-donnees.md](../08-automatisation-soc/source-des-donnees.md))

```bash
sudo tail -f /var/log/suricata/fast.log
```

## Plan de la page (à compléter)

- [ ] Interface réseau surveillée et mode (IDS passif vs IPS inline) — voir [Suricata modes](https://docs.suricata.io/en/latest/setting-up-ipsinline-for-linux.html)
- [ ] Jeux de règles activés (ET Open, custom rules ?) — voir [Rule management](https://docs.suricata.io/en/latest/rule-management/index.html)
- [ ] Intégration avec Wazuh (lecture de `eve.json` par l'agent Wazuh) — voir [Localfile reference (ossec.conf)](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html) et [Wazuh + Suricata integration guide](https://documentation.wazuh.com/current/proof-of-concept-guide/detect-network-vulnerabilities-suricata.html)
- [ ] Lien avec la classification des alertes du script (catégorie "exploitation", voir [classification-alertes.md](../08-automatisation-soc/classification-alertes.md))
- [ ] Dépannage rencontré (le cas échéant)

## Documentation officielle

- [Suricata — documentation officielle](https://docs.suricata.io/en/latest/)
- [Suricata — gestion des règles](https://docs.suricata.io/en/latest/rule-management/index.html)
- [Suricata — format des logs eve.json](https://docs.suricata.io/en/latest/output/eve/eve-json-format.html)
- [Wazuh — intégration avec Suricata](https://documentation.wazuh.com/current/proof-of-concept-guide/detect-network-vulnerabilities-suricata.html)
