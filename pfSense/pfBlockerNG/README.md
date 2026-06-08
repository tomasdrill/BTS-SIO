# 🌐 Procédure — Mise en place de pfBlockerNG

## 📋 Informations générales

| Champ | Détail |
|---|---|
| **Nom** | Tomas Cohen |
| **N° candidat** | 2545812794 |
| **École** | Institut F2I |
| **Année** | 2025 - 2026 |
| **Serveur** | pfSense 2.8.1 |

---

## 🎯 Objectif

Mettre en place **pfBlockerNG** sur pfSense afin de :
- Bloquer les sites web non professionnels
- Filtrer les publicités et contenus malveillants
- Contrôler les accès internet des postes clients

---

## 🔍 C'est quoi pfBlockerNG ?

```
pfBlockerNG = Plugin pfSense de filtrage DNS

→ Intercepte les requêtes DNS des postes
→ Si le site est dans la liste de blocage
  → Redirige vers l'IP virtuelle 10.10.10.1
  → L'utilisateur voit "Site bloqué"
→ Sinon → laisse passer la requête
```

### Fonctionnement

```
PDT → youtube.com → AD (192.168.1.2)
                     ↓
                pfSense (192.168.1.1)
                     ↓
              pfBlockerNG vérifie
                     ↓
        youtube.com → 10.10.10.1 ❌ BLOQUÉ
        google.com  → IP réelle  ✅ OK
```

---

## ⚙️ Étapes de mise en place

### Étape 1 — Installation

```
1. pfSense → Système
   → Gestionnaire de paquets
   → Paquets disponibles

2. Cherche "pfBlockerNG"
3. Clique sur Installer
4. Attends la fin
```

---

### Étape 2 — Assistant de configuration

```
Pare-feu → pfBlockerNG → Assistant

Étape 1 : Interfaces
→ Inbound  → WAN ✅
→ Outbound → LAN + LANPDT ✅

Étape 2 : IP Component
→ Inbound  → WAN ✅
→ Outbound → LAN + LANPDT ✅

Étape 3 : DNSBL
→ VIP Address → 10.10.10.1
→ Port → 8081
→ SSL Port → 8443
→ DNSBL Whitelist → ✅

Étape 4 : Terminer
```

---

### Étape 3 — Activation pfBlockerNG

![general_config](captures/03_general_config.PNG)

```
Pare-feu → pfBlockerNG → Général

✅ pfBlockerNG → Activer
✅ Keep Settings → Activer
CRON Settings → Every hour
```

---

### Étape 4 — Configuration DNSBL

```
Pare-feu → pfBlockerNG → DNSBL

✅ DNSBL → ON
VIP Address → 10.10.10.1
Port → 8081
✅ Permit Firewall Rules → LAN + LANPDT
Global Logging/Blocking Mode → Enabled
Blocked Webpage → dnsbl_default.php
```

---

### Étape 5 — Catégories de blocage UT1

![categories_UT1](captures/04_categories_UT1.PNG)

```
Pare-feu → pfBlockerNG → DNSBL
→ DNSBL Category

Catégories activées ✅ :
✅ Adult (XXX) → Sites pornographiques
✅ Aggressive → Sites agressifs
✅ Arjel → Sites de jeux d'argent
✅ Bitcoin → Sites de minage crypto
✅ Celebrity → Sites people/célébrités
```

---

### Étape 6 — Création des groupes DNSBL

![dnsbl_groups](captures/05_dnsbl_groups.PNG)

```
2 groupes créés :
→ ADs_Basic → Blocage publicités
→ SitesBloques → Blocage sites non pro
```

---

### Étape 7 — Configuration du groupe SitesBloques

![sites_config](captures/06_sites_bloques_config.PNG)

```
Name → SitesBloques
Description → Blocage sites non professionnels

DNSBL Source :
→ URL : https://raw.githubusercontent.com/
         StevenBlack/hosts/master/hosts
→ État : ON ✅
→ Header : SitesBloques

Action → Unbound
Fréquence → Once a day
```

---

### Étape 8 — Liste personnalisée (Custom List)

![custom_list](captures/07_custom_list.PNG)

```
Sites bloqués manuellement :
→ youtube.com
→ www.youtube.com
→ www.institut-f2i.fr
→ institut-f2i.fr
```

---

### Étape 9 — Configuration DNS sur l'AD

> **Important :** Pour que pfBlockerNG fonctionne,
> l'AD doit utiliser pfSense comme DNS !

```
Sur Windows Server :
Gestionnaire DNS → ADDS
→ Propriétés → Redirecteurs
→ Supprimer 8.8.8.8 et 1.1.1.1
→ Garder uniquement : 192.168.1.1 ✅
```

---

### Étape 10 — Mise à jour des règles

```
Pare-feu → pfBlockerNG → Mettre à jour
→ Force ✅
→ Clique Run
→ Attends la fin
```

---

## 🧪 Tests de fonctionnement

### Test 1 — nslookup youtube.com

![nslookup](captures/08_nslookup_bloque.PNG)

```
C:\Users\win> nslookup youtube.com
Serveur  : UnKnown
Address  : 192.168.1.2

Nom      : youtube.com
Address  : 10.10.10.1 ✅ BLOQUÉ !
```

> youtube.com est redirigé vers 10.10.10.1
> = pfBlockerNG fonctionne ! 🎉

---

### Test 2 — Accès youtube.com depuis le navigateur

![youtube_bloque](captures/09_youtube_bloque.PNG)

```
✅ youtube.com → BLOQUÉ !
   "Votre connexion n'est pas privée"
   NET::ERR_CERT_AUTHORITY_INVALID
   → pfBlockerNG intercepte la connexion
   → Redirige vers son certificat auto-signé
   → Le site est bien bloqué ! ❌
```

---

## 📊 Résultats obtenus

```
✅ pfBlockerNG installé et actif
✅ Catégories UT1 activées (Adult, Bitcoin...)
✅ Groupe SitesBloques configuré
✅ Liste personnalisée (youtube, f2i...)
✅ DNS AD → pfSense → pfBlockerNG
✅ youtube.com → 10.10.10.1 (bloqué)
✅ Blocage confirmé dans le navigateur
```

---

## 🔧 Résolution d'un problème

### Problème — youtube.com accessible malgré le blocage DNS

**Cause :** L'AD utilisait plusieurs serveurs DNS :
```
❌ 192.168.1.1 (pfSense) → bloqué
❌ 8.8.8.8 (Google) → contournait pfBlockerNG !
❌ 1.1.1.1 (Cloudflare) → contournait pfBlockerNG !
```

**Solution :** Supprimer 8.8.8.8 et 1.1.1.1 des redirecteurs AD :
```
Gestionnaire DNS → Redirecteurs
→ Supprimer 8.8.8.8 ❌
→ Supprimer 1.1.1.1 ❌
→ Garder uniquement 192.168.1.1 ✅
```

---

## 📸 Captures disponibles

| Fichier | Description |
|---|---|
| 01_installation.PNG | Installation pfBlockerNG |
| 02_config_DNSBL.PNG | Configuration DNSBL |
| 03_general_config.PNG | pfBlockerNG activé |
| 04_categories_UT1.PNG | Catégories UT1 |
| 05_dnsbl_groups.PNG | Groupes DNSBL |
| 06_sites_bloques_config.PNG | Config SitesBloques |
| 07_custom_list.PNG | Liste personnalisée |
| 08_nslookup_bloque.PNG | nslookup → 10.10.10.1 |
| 09_youtube_bloque.PNG | youtube.com bloqué |

---

## 🔗 Accès aux productions

- **GitHub** : https://github.com/tomasdrill/BTS-SIO
- **Interface pfBlockerNG** : Pare-feu → pfBlockerNG

---

## 📚 Compétences travaillées

- ✅ Concevoir une solution d'infrastructure réseau
- ✅ Installer, tester et déployer une solution d'infrastructure réseau
- ✅ Exploiter, dépanner et superviser une solution d'infrastructure réseau
