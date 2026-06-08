# 💾 Procédure — Sauvegarde et Restauration GLPI

## 📋 Informations générales

| Champ | Détail |
|---|---|
| **Nom** | Tomas Cohen |
| **N° candidat** | 2545812794 |
| **École** | Institut F2I |
| **Année** | 2025 - 2026 |
| **Serveur** | Debian 12 — 192.168.1.5 |

---

## 🎯 Objectif

Mettre en place une procédure de sauvegarde complète de GLPI afin de :
- Protéger les données en cas de panne ou d'attaque
- Permettre une restauration rapide du service
- Assurer la continuité de l'activité (PCA/PRA)

---

## 🔍 Pourquoi sauvegarder GLPI ?

```
GLPI contient des données critiques :
→ Inventaire de tout le parc informatique
→ Historique de tous les tickets
→ Comptes utilisateurs et rôles
→ Configuration LDAP et paramètres

Sans sauvegarde :
→ Perte définitive des données si panne
→ Impossible de restaurer après ransomware
→ Perte de l'historique des interventions
```

---

## 📦 Ce qu'on sauvegarde

```
1. Base de données MySQL (glpi)
   → Contient TOUTES les données GLPI
   → Tickets, inventaire, utilisateurs...

2. Fichiers GLPI (/var/www/glpi)
   → Fichiers de configuration
   → Plugins installés
   → Documents uploadés
   → Logs
```

---

## 💾 Procédure de Sauvegarde

### Sauvegarde complète

![sauvegarde](captures/11_sauvegarde.PNG)

### Étape 1 — Sauvegarde de la base de données

```bash
mysqldump -u root -p glpi > /tmp/sauvegarde_glpi.sql
```

> Quand il demande le mot de passe → tape le mot de passe MySQL

**Résultat :**
```
/tmp/sauvegarde_glpi.sql → 1,4K ✅
```

---

### Étape 2 — Sauvegarde des fichiers GLPI

```bash
tar -czf /tmp/sauvegarde_glpi_fichiers.tar.gz /var/www/glpi
```

> Note : Le message "Suppression de « / »" est normal !
> C'est tar qui retire le / du chemin pour la portabilité

**Résultat :**
```
/tmp/sauvegarde_glpi_fichiers.tar.gz → 93M ✅
```

---

### Étape 3 — Vérification des sauvegardes

```bash
ls -lh /tmp/sauvegarde_glpi*
```

**Résultat :**
```
-rw-r--r-- 1 root root  93M  9 juin 01:43
           /tmp/sauvegarde_glpi_fichiers.tar.gz

-rw-r--r-- 1 root root 1,4K  9 juin 01:42
           /tmp/sauvegarde_glpi.sql
```

---

### Étape 4 — Copier sur un emplacement sécurisé

```bash
# Copier vers un partage réseau Windows
cp /tmp/sauvegarde_glpi* /mnt/backup/

# Ou copier vers un autre serveur via SCP
scp /tmp/sauvegarde_glpi* user@192.168.1.10:/backup/
```

> 💡 Une sauvegarde stockée sur le même serveur
> ne protège pas contre une panne du disque !
> Toujours sauvegarder sur un support externe !

---

### Automatiser la sauvegarde (Cron)

```bash
# Ouvrir le crontab
crontab -e

# Ajouter cette ligne pour sauvegarder chaque nuit à 2h
0 2 * * * mysqldump -u root -pMOTDEPASSE glpi > /tmp/sauvegarde_glpi_$(date +\%Y\%m\%d).sql
```

---

## 🔄 Procédure de Restauration

> ⚠️ La restauration écrase les données existantes !
> Vérifie bien que tu as la bonne sauvegarde avant !

### Étape 1 — Arrêter Apache

```bash
systemctl stop apache2
```

---

### Étape 2 — Restaurer la base de données

```bash
mysql -u root -p glpi < /tmp/sauvegarde_glpi.sql
```

> Tape le mot de passe MySQL quand demandé

---

### Étape 3 — Restaurer les fichiers GLPI

```bash
# Supprimer les fichiers actuels
rm -rf /var/www/glpi/

# Restaurer depuis la sauvegarde
tar -xzf /tmp/sauvegarde_glpi_fichiers.tar.gz -C /

# Remettre les bonnes permissions
chown -R www-data:www-data /var/www/glpi/
chmod -R 755 /var/www/glpi/
```

---

### Étape 4 — Redémarrer les services

```bash
systemctl start apache2
systemctl restart mariadb
```

---

### Étape 5 — Vérifier que GLPI fonctionne

```
1. Ouvre un navigateur
2. Tape : https://192.168.1.5/glpi
3. Connecte toi avec glpi/glpi
4. Vérifie que les données sont bien restaurées :
   → Parc → Ordinateurs → PDT visible ?
   → Assistance → Tickets → tickets visibles ?
```

---

## 📊 Résumé des commandes

| Action | Commande |
|---|---|
| Sauvegarder BDD | `mysqldump -u root -p glpi > sauvegarde.sql` |
| Sauvegarder fichiers | `tar -czf sauvegarde.tar.gz /var/www/glpi` |
| Vérifier | `ls -lh /tmp/sauvegarde*` |
| Restaurer BDD | `mysql -u root -p glpi < sauvegarde.sql` |
| Restaurer fichiers | `tar -xzf sauvegarde.tar.gz -C /` |
| Permissions | `chown -R www-data:www-data /var/www/glpi/` |

---

## 💡 Bonnes pratiques

```
✅ Sauvegarder tous les jours via Cron
✅ Stocker sur un support externe
✅ Tester la restauration régulièrement
✅ Nommer les sauvegardes avec la date
✅ Conserver plusieurs versions
✅ Documenter la procédure (ce fichier !)
```

---

## 🔗 Liens avec PRA/PCA

```
PRA (Plan de Reprise d'Activité)
→ Ce README est la procédure de reprise
→ Permet de restaurer GLPI après incident

PCA (Plan de Continuité d'Activité)
→ Les sauvegardes automatiques garantissent
  la continuité même en cas de panne
```

---

## 📸 Captures disponibles

| Fichier | Description |
|---|---|
| 11_sauvegarde.PNG | Commandes de sauvegarde et résultat |

---

## 🔗 Accès aux productions

- **GitHub** : https://github.com/tomasdrill/BTS-SIO
- **Sauvegardes** : /tmp/sauvegarde_glpi*

---

## 📚 Compétences travaillées

- ✅ Concevoir une solution d'infrastructure réseau
- ✅ Installer, tester et déployer une solution d'infrastructure réseau
- ✅ Exploiter, dépanner et superviser une solution d'infrastructure réseau
