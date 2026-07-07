# Classification des alertes

Les règles de classification sont définies dans `soc_config.py` (`DETECTION_RULES`, fonction `classify_alert()`). Chaque alerte Wazuh/Suricata est testée contre un niveau minimum (`min_level`), des `wazuh_groups`, des `suricata_categories` et une liste de mots-clés, dans cet ordre de priorité.

## Catégories (`AlertCategory`)

| Catégorie | Niveau min | Déclencheurs principaux |
|---|---|---|
| `NETWORK_SCAN` | 3 | nmap, masscan, scans SYN/XMAS/FIN/NULL, ping sweep, recon |
| `BRUTE_FORCE` | 5 | failed password, invalid user, tentatives SSH/RDP/FTP/SMB/SMTP répétées |
| `MALWARE` | 5 | trojan, ransomware, rootkit, coinminer, résultat VirusTotal/YARA/FIM |
| `WEB_ATTACK` | 5 | SQLi, XSS, LFI/RFI, path traversal, upload de web shell |
| `REVERSE_SHELL` | 6 | reverse shell, mimikatz, PowerShell encodé, `nc -e`, command injection |
| `EXPLOIT` | 7 | CVE, RCE, buffer overflow, élévation de privilèges, ajout d'utilisateur admin |
| `RECON` (défense) | 7 | antivirus désactivé, logs supprimés, altération système |
| `DDOS` | 6 | exfiltration, C2, beacon, DDoS/flood, tunneling DNS |
| `THREAT_REMOVED` | 3 | fichier supprimé/mis en quarantaine par Active Response, VirusTotal ou YARA |
| `SSH`, `FIM`, `OTHER` | — | catégories additionnelles gérées séparément dans `surveillance_soc.py` |

> `THREAT_REMOVED` est testé **en priorité absolue** sur toutes les autres catégories (ex. "removed by VirusTotal" contient "virus", qui matcherait sinon `MALWARE`).

## Catégories réellement actionnables

`ACTIONABLE_CATEGORIES` exclut volontairement `OTHER` et le bruit générique (connexion PAM, sudo...) pour ne pas saturer Telegram avec des événements non exploitables.

## Catégories bloquées nativement par Wazuh (pas par le script)

`NATIVE_AR_CATEGORIES = {NETWORK_SCAN, BRUTE_FORCE}` : ces catégories sont bloquées **côté Wazuh Manager** via Active Response (`firewall-drop`, rules_id `5710/5712/5716/5720` pour le brute force SSH, `86601` pour les scans Suricata) — voir [Active Response](../03-wazuh/active-response.md). Le blocage local du script serait redondant, et surtout **trompeur** pour ces catégories : il bloquerait sur la machine qui héberge le script, pas sur la machine réellement attaquée.

## Ajustement de sévérité (`get_severity_for_category`)

- `MALWARE`, `REVERSE_SHELL`, `EXPLOIT` → sévérité forcée à `CRITICAL` minimum
- `NETWORK_SCAN`, `DDOS` → `HIGH` minimum
- `BRUTE_FORCE` → `MEDIUM` minimum
