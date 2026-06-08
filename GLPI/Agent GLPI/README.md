# 💻 Déploiement Agent GLPI via GPO

## 📋 Informations générales

| Champ | Détail |
|---|---|
| **Nom** | Tomas Cohen |
| **N° candidat** | 2545812794 |
| **École** | Institut F2I |
| **Année** | 2025 - 2026 |
| **Version Agent** | GLPI Agent 1.17 |

---

## 🎯 Objectif

Déployer automatiquement l'agent GLPI sur tous les postes du domaine via une GPO afin d'alimenter l'inventaire GLPI sans intervention manuelle sur chaque poste.

---

## 🏗️ Infrastructure

| Machine | Rôle | IP |
|---|---|---|
| Windows Server 2025 | AD / GPO | 192.168.1.2 |
| Debian 12 | Serveur GLPI | 192.168.1.5 |
| PDT / PDT2 | Postes clients | DHCP 192.168.2.x |

---

## 📁 Partage réseau

L'agent GLPI a été déposé sur un partage réseau accessible depuis tous les postes du domaine :

```
Chemin UNC : \\ADDS\GLPI-agent
Fichier     : GLPI-Agent-1.17-x64.msi
```

### Configuration du partage

```
1. Dossier créé sur Windows Server :
   C:\GLPI-agent\

2. Fichier déposé :
   C:\GLPI-agent\GLPI-Agent-1.17-x64.msi

3. Partage configuré :
   Nom du partage → GLPI-agent
   Droits         → Tout le monde : Lecture
   Accès UNC      → \\ADDS\GLPI-agent
```

---

## 🔧 Installation de l'agent GLPI 1.17

L'installation se déroule en 6 étapes :

### Étape 1 — Bienvenue
![inst1](captures/inst1.PNG)

```
→ Lancement de l'assistant d'installation
   GLPI Agent 1.17 Setup
→ Clic sur Next pour continuer
```

### Étape 2 — Licence GNU GPL
![inst2](captures/inst2.PNG)

```
→ Acceptation de la licence
   GNU GENERAL PUBLIC LICENSE Version 2
→ Logiciel open source et gratuit
→ Clic sur Next pour continuer
```

### Étape 3 — Dossier d'installation
![inst3](captures/inst3.PNG)

```
→ Dossier de destination :
   C:\Program Files\GLPI-Agent\
→ Laisse le dossier par défaut
→ Clic sur Next pour continuer
```

### Étape 4 — Type d'installation
![inst4](captures/inst4.PNG)

```
→ Choix du type : Typical
   (Inventory + RemoteInventory)
→ Suffisant pour notre usage
→ Clic sur Typical
```

### Étape 5 — Configuration du serveur
![inst5](captures/inst5.PNG)

```
→ Remote Targets : 192.168.1.5
   (IP du serveur GLPI sur Debian)
→ Local Target : vide
→ Quick installation : coché ✅
→ Clic sur Next
```

### Étape 6 — Installation
![inst6](captures/inst6.PNG)

```
→ Résumé de la configuration
→ Clic sur Install pour lancer
   l'installation de l'agent
```

---

## ⚙️ Configuration de la GPO

### Étape 1 — Création de la GPO

```
1. Ouvre : Gestion des stratégies de groupe
2. Clic droit sur l'OU GLPI
   → Créer un objet GPO et le lier ici
3. Nom : Déploiement Agent GLPI
```

### Étape 2 — Configuration de la GPO

```
Clic droit sur la GPO → Modifier

Configuration ordinateur
→ Stratégies
→ Paramètres du logiciel
→ Installation de logiciels
→ Nouveau → Package

Chemin : \\ADDS\GLPI-agent\GLPI-Agent-1.17-x64.msi
Type    : Affecté
```

### Étape 3 — Liaison au domaine

```
La GPO est liée à l'OU GLPI
du domaine pfsense.local
→ S'applique automatiquement
  à tous les postes du domaine
```

---

## 🚨 Problème rencontré et solution

### Problème : Propagation lente de la GPO

**Description :** Un délai dans l'application des stratégies de groupe a été constaté sur les postes clients.

**Cause :** Par défaut les GPO se propagent toutes les **90 minutes**.

**Solution :** Forcer l'application immédiate via :

```
gpupdate /force
```

---

## ✅ Résultats obtenus

- ✅ Agent GLPI 1.17 installé sur PDT et PDT2
- ✅ 2 postes visibles dans Parc → Ordinateurs
- ✅ 116 logiciels inventoriés automatiquement
- ✅ Informations matérielles remontées automatiquement

### Postes dans l'inventaire GLPI

| Poste | OS | Dernière MAJ |
|---|---|---|
| PDT | Windows 11 Pro | 2026-06-03 |
| PDT2 | Windows 11 Pro | 2026-06-03 |

---

## 🔍 Vérification

```
Depuis le poste client :
http://localhost:62354
→ Agent Running ✅
→ Force an inventory
```

---

## 📸 Captures disponibles

| Fichier | Description |
|---|---|
| inst1.PNG | Bienvenue installation |
| inst2.PNG | Licence GNU GPL |
| inst3.PNG | Dossier installation |
| inst4.PNG | Type Typical |
| inst5.PNG | Config serveur GLPI |
| inst6.PNG | Prêt à installer |
| 01_agent.PNG | Interface agent |
| 02_agent_config.PNG | Configuration agent |
| 03_agent_deploy.PNG | Agent déployé |

---

## 🔗 Accès aux productions

- **GitHub** : https://github.com/tomasdrill/BTS-SIO
- **Partage réseau** : \\ADDS\GLPI-agent
- **Interface agent** : http://localhost:62354

---

## 📚 Compétences travaillées

- ✅ Concevoir une solution d'infrastructure réseau
- ✅ Installer, tester et déployer une solution d'infrastructure réseau
- ✅ Exploiter, dépanner et superviser une solution d'infrastructure réseau
