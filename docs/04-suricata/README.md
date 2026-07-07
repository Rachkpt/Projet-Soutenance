# 🚨 Suricata — IDS/IPS réseau

> Module SOC — détection d'intrusion réseau. Page minimale pour l'instant : à enrichir avec la configuration réelle du projet (règles activées, mode IDS vs IPS, intégration Wazuh).

## Installation

```bash
sudo apt install -y suricata
```

### Télécharger les règles

```bash
sudo suricata-update
```

### Démarrer le service

```bash
sudo systemctl enable --now suricata
sudo systemctl status suricata
```

### Vérifier les alertes en temps réel

```bash
sudo tail -f /var/log/suricata/fast.log
```

## Plan de la page (à compléter)

- [ ] Interface réseau surveillée et mode (IDS passif vs IPS inline)
- [ ] Jeux de règles activés (ET Open, custom rules ?)
- [ ] Intégration avec Wazuh (lecture de `eve.json` par l'agent Wazuh — `<localfile>` dans `ossec.conf`)
- [ ] Lien avec la classification des alertes du script (catégorie "exploitation", voir [classification-alertes.md](../08-automatisation-soc/classification-alertes.md))
- [ ] Dépannage rencontré (le cas échéant)

## Documentation officielle

- [Suricata — documentation officielle](https://docs.suricata.io/en/latest/)
