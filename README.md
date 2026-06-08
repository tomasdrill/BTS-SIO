# 🖥️ BTS SIO SISR — Projet E6 : Sécurisation et Administration Réseau

## 📋 Informations générales

| Champ | Détail |
|---|---|
| **Nom** | Tomas Cohen |
| **N° candidat** | 2545812794 |
| **École** | Institut F2I |
| **Année** | 2025 - 2026 |
| **Option** | SISR |

---

## 🎯 Objectif du projet

Déployer et sécuriser une infrastructure réseau complète comprenant :
- Un pare-feu pfSense pour la sécurisation et le routage
- Un serveur GLPI pour la gestion du parc informatique
- Un système IDS/IPS Suricata pour la détection d'intrusions
- Un filtrage web pfBlockerNG pour le contrôle des accès internet

---

## 🏗️ Infrastructure mise en place

### Plan d'adressage

| Machine | Rôle | Interface | Adresse IP | Masque |
|---|---|---|---|---|
| pfSense | Pare-feu / Routeur | WAN | 192.168.15.94 | /24 |
| pfSense | Pare-feu / Routeur | LAN | 192.168.1.1 | /24 |
| pfSense | Pare-feu / Routeur | LANPDT | 192.168.2.1 | /24 |
| Windows Server 2025 | Active Directory / DNS / DHCP | LAN | 192.168.1.2 | /24 |
| Debian 12 | Serveur GLPI | LAN | 192.168.1.5 | /24 |
| PDT / PDT2 | Postes clients | LANPDT | DHCP | /24 |

### Plage DHCP

| Paramètre | Valeur |
|---|---|
| Réseau | 192.168.2.0/24 |
| Passerelle | 192.168.2.1 |
| Plage exclue | 192.168.2.1 → 192.168.2.10 |
| Plage distribuée | 192.168.2.11 → 192.168.2.254 |
| Serveur DHCP | 192.168.1.2 (AD) via DHCP Relay |

### Schéma réseau

```
Internet
    |
[WAN: 192.168.15.94]
    |
[pfSense 2.8.1]
[Suricata IDS/IPS]
[pfBlockerNG]
    |              |
[LAN]          [LANPDT]
192.168.1.1    192.168.2.1
    |                |
192.168.1.2      PDT/PDT2
(AD/DNS/DHCP)    DHCP auto
    |
192.168.1.5
(GLPI HTTPS)
```

---

## 🔥 Partie 1 — pfSense

### Technologies utilisées

| Technologie | Version | Rôle |
|---|---|---|
| pfSense | 2.8.1-RELEASE | Pare-feu / Routeur |
| Proxmox | - | Virtualisation |

### Fonctionnalités configurées

#### Interfaces réseau
- **WAN** → 192.168.15.94/24 (accès internet)
- **LAN** → 192.168.1.1/24 (réseau serveurs)
- **LANPDT** → 192.168.2.1/24 (réseau postes clients)

#### Règles de filtrage WAN
- Blocage des réseaux RFC 1918
- Blocage des réseaux invalides IANA

#### Règles de filtrage LAN
- Règle anti-blocage (ports 80 et 443)
- Default allow LAN to any (IPv4 et IPv6)

#### Règles de filtrage LANPDT
- Autorisation DHCP vers AD (UDP)
- Autorisation ADP vers AD (TCP/UDP)
- Blocage GLPI 8h-18h (ordonnancement temporel)
- Autorisation accès internet (TCP/UDP)
- Autorisation ICMP (ping)

#### Services configurés
- **NAT** → Permet l'accès internet aux machines internes
- **DHCP Relay** → Transmet les requêtes DHCP vers l'AD (192.168.1.2)

---

## 🖥️ Partie 2 — GLPI

### Technologies utilisées

| Technologie | Version | Rôle |
|---|---|---|
| GLPI | 10.x | Gestion de parc / Ticketing |
| Debian | 12 | Système hôte |
| Apache | 2.4 | Serveur web |
| MySQL | - | Base de données |
| Active Directory | - | Authentification LDAP |

### Fonctionnalités configurées

#### 1. GLPI en HTTPS
- Certificat SSL auto-signé créé avec OpenSSL
- Apache configuré sur le port 443
- Redirection automatique HTTP → HTTPS
- Accès : **https://192.168.1.5/glpi**

#### 2. Authentification LDAP
- Connexion à l'AD via le protocole LDAP (port 389)
- BaseDN : DC=pfsense,DC=local
- Les utilisateurs AD peuvent se connecter à GLPI
- 11 entrées trouvées dans l'annuaire

#### 3. Gestion des utilisateurs et rôles

| Utilisateur | Rôle | Droits |
|---|---|---|
| glpi | Super-Admin | Accès complet |
| technicien | Technicien | Gestion tickets + inventaire |
| superviseur | Supervisor | Supervision tickets |
| utilisateur | Self-Service | Création tickets uniquement |
| alice, paul, sacha... | Self-Service | Comptes AD importés |

#### 4. Déploiement agent via GPO
- Agent GLPI 1.17 déployé automatiquement via GPO Windows
- GPO liée au domaine pfsense.local
- Commande utilisée pour forcer l'application : `gpupdate /force`
- Agent configuré : `server = 192.168.1.5`

#### 5. Inventaire automatique
- 2 postes détectés automatiquement (PDT, PDT2)
- 116 logiciels inventoriés
- Système : Microsoft Windows 11 Professionnel
- Mise à jour automatique toutes les 24h

#### 6. Gestion des tickets
- Tickets créés par les utilisateurs AD (alice, sacha...)
- Prise en charge par les techniciens
- Résolution et clôture documentées

#### Problème rencontré et solution
**Problème :** Propagation lente de la GPO sur les postes clients
**Solution :** Utilisation de `gpupdate /force` pour forcer l'application immédiate

---

## 🛡️ Partie 3 — Suricata IDS/IPS

### Technologies utilisées

| Technologie | Rôle |
|---|---|
| Suricata | IDS/IPS sur pfSense |
| ET Open Rules | Règles de détection |
| Feodo Tracker | Blocage botnets |

### Configuration

#### Mode de fonctionnement
- Interface surveillée : **WAN**
- Mode : **Legacy IPS**
- Block Offenders : **Activé** (bloque automatiquement les IPs malveillantes)
- Which IP to Block : **BOTH**

#### Règles activées

**Default Rules :**
- app-layer-events, decoder-events, dhcp-events
- dns-events, files, ftp-events, http-events
- http2-events, kerberos-events, ntp-events

**ET Open Rules :**
- emerging-botcc (botnets)
- emerging-compromised (IPs compromises)
- emerging-dos (attaques DDoS)
- emerging-dns (attaques DNS)
- emerging-exploit (exploits)
- emerging-malware (malwares)
- emerging-scan (scans de ports)
- emerging-shellcode (attaques avancées)
- emerging-sql (injections SQL)
- emerging-trojan (chevaux de Troie)
- emerging-web_client (attaques navigateur)
- emerging-web_server (attaques serveur web)
- emerging-worm (vers informatiques)
- Feodo Tracker Botnet C2 IP Rules

#### Test de fonctionnement
- Scan Nmap depuis PDT (192.168.2.12) vers GLPI (192.168.1.5)
- Alertes ICMP générées et visibles dans les logs Suricata
- Ports filtrés confirmés dans les résultats Nmap

---

## 🌐 Partie 4 — pfBlockerNG

### Technologies utilisées

| Technologie | Rôle |
|---|---|
| pfBlockerNG | Filtrage web / DNS |
| DNSBL | Blocage par DNS |
| ETOpen | Listes de blocage |

### Configuration

#### Paramètres DNSBL
- VIP Address : 10.10.10.1
- Port : 8081
- SSL Port : 8443
- Logging/Blocking Mode : DNSBL WebServer/VIP

#### Sites bloqués
- youtube.com
- facebook.com
- tiktok.com
- instagram.com
- snapchat.com

#### Fonctionnement
```
PDT → DNS → AD (192.168.1.2)
             ↓
          Redirige vers pfSense (192.168.1.1)
             ↓
          pfBlockerNG vérifie
             ↓
youtube.com → 10.10.10.1 (BLOQUÉ ❌)
google.com  → IP réelle (AUTORISÉ ✅)
```

#### Preuve de fonctionnement
- nslookup youtube.com → 10.10.10.1 ✅
- Accès youtube.com depuis PDT → Bloqué ✅
- Accès google.com depuis PDT → Accessible ✅

---

## ✅ Récapitulatif des tâches réalisées

| Tâche | Description | Statut |
|---|---|---|
| 1 | GLPI en HTTPS avec certificat SSL | ✅ |
| 2 | 3 utilisateurs avec rôles différents | ✅ |
| 3 | Mots de passe sécurisés | ✅ |
| 4 | Gestion des tickets | ✅ |
| 5 | Plugin Inventory + agent GPO | ✅ |
| 6 | Filtrage web pfBlockerNG | ✅ |
| 7 | Alias pfSense | ✅ |
| 8 | Analyse des logs | ✅ |
| 9 | Simulation d'incidents Suricata | ✅ |

---

## 📚 Compétences travaillées

- ✅ Concevoir une solution d'infrastructure réseau
- ✅ Installer, tester et déployer une solution d'infrastructure réseau
- ✅ Exploiter, dépanner et superviser une solution d'infrastructure réseau

---

## 🔗 Accès aux productions

- **GitHub** : https://github.com/tomasdrill/BTS-SIO
- Captures d'écran dans les dossiers correspondants
- Documentation technique complète disponible

---

## 📁 Organisation du dépôt

```
BTS-SIO/
├── README.md (ce fichier)
├── pfSense/
│   ├── README.md
│   └── captures/
├── GLPI/
│   ├── README.md
│   └── captures/
├── Suricata/
│   ├── README.md
│   └── captures/
└── pfBlockerNG/
    ├── README.md
    └── captures/
```
