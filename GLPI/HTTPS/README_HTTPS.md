# 🔒 Procédure — Mise en place de GLPI en HTTPS

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

Sécuriser l'accès à GLPI en activant le protocole **HTTPS** avec un certificat SSL auto-signé afin de chiffrer les communications entre les utilisateurs et le serveur GLPI.

---

## 🏗️ Prérequis

```
✅ GLPI installé sur Debian 12
✅ Apache 2.4 installé
✅ MySQL installé
✅ GLPI accessible en HTTP sur 192.168.1.5
```

---

## ⚙️ Étapes de mise en place

### Étape 1 — Activer les modules Apache

```bash
a2enmod ssl
a2enmod rewrite
a2enmod headers
systemctl restart apache2
```

#### Vérification des modules actifs
![modules_actif](captures/08_modules_apache.PNG)

```
✅ headers_module (shared)
✅ rewrite_module (shared)
✅ ssl_module (shared)
```

---

### Étape 2 — Créer le certificat SSL auto-signé

```bash
# Créer le dossier
mkdir /etc/ssl/glpi

# Créer le certificat
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/ssl/glpi/glpi.key \
-out /etc/ssl/glpi/glpi.crt \
-subj "/C=FR/ST=Ile-de-France/L=Paris/O=Institut F2I/CN=192.168.1.5"
```

#### Résultat — Certificat SSL créé
![certificat](captures/05_certificat_ssl.PNG)

```
✅ Certificat valide 365 jours
✅ Clé RSA 2048 bits
✅ Stocké dans /etc/ssl/glpi/
```

---

### Étape 3 — Configurer Apache pour HTTPS

```bash
nano /etc/apache2/sites-available/glpi-ssl.conf
```

#### Contenu du fichier glpi-ssl.conf
![config_ssl](captures/10_config_ssl.PNG)

```apache
<VirtualHost *:443>
    ServerName 192.168.1.5
    DocumentRoot /var/www/glpi/public

    SSLEngine on
    SSLCertificateFile /etc/ssl/glpi/glpi.crt
    SSLCertificateKeyFile /etc/ssl/glpi/glpi.key

    <Directory /var/www/glpi/public>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:80>
    ServerName 192.168.1.5
    Redirect permanent / https://192.168.1.5/
</VirtualHost>
```

---

### Étape 4 — Activer le site HTTPS

```bash
# Activer le nouveau site HTTPS
a2ensite glpi-ssl.conf

# Désactiver les anciens sites en conflit
a2dissite glpi.sio.conf
a2dissite support.it-connectlab.fr.conf
a2dissite 000-default.conf

# Redémarrer Apache
systemctl restart apache2
```

#### Vérification — Un seul site actif
![site_actif](captures/09_site_actif.PNG)

```
✅ glpi-ssl.conf → seul site actif !
```

---

### Étape 5 — Créer le fichier .htaccess

```bash
nano /var/www/glpi/public/.htaccess
```

#### Contenu du fichier .htaccess
![htaccess](captures/06_htaccess.PNG)

```apache
RewriteEngine On

# Redirect all requests to index.php
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

```bash
# Donner les bonnes permissions
chown www-data:www-data /var/www/glpi/public/.htaccess
chmod 644 /var/www/glpi/public/.htaccess
systemctl restart apache2
```

---

### Étape 6 — Vérification

#### GLPI accessible en HTTPS
![glpi_https](captures/07_glpi_https.PNG)

```
✅ URL : https://192.168.1.5/glpi
✅ HTTPS actif
✅ HTTP redirigé automatiquement vers HTTPS
```

---

## 🚨 Problèmes rencontrés et solutions

### Problème 1 — Erreur "Not Found"

**Description :** Apache retournait une erreur 404 Not Found sur le port 443.

**Cause :** Le `DocumentRoot` pointait vers `/var/www/html/glpi` au lieu de `/var/www/glpi/public`

**Solution :**
```apache
# Mauvais chemin ❌
DocumentRoot /var/www/html/glpi

# Bon chemin ✅
DocumentRoot /var/www/glpi/public
```

---

### Problème 2 — Erreur "Web server seems to be misconfigured"

**Description :** GLPI affichait une erreur de configuration du serveur web.

**Cause :** Le `DocumentRoot` devait pointer vers le dossier `/public` de GLPI et non la racine.

**Solution :** Pointer vers `/var/www/glpi/public` ET créer le fichier `.htaccess`

---

### Problème 3 — Erreur certificat CA

**Description :** Apache générait une erreur `AH01906: server certificate is a CA certificate`

**Cause :** La commande openssl créait un certificat CA au lieu d'un certificat serveur.

**Solution :** Utiliser l'option `-subj` pour créer un certificat serveur correct :
```bash
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/ssl/glpi/glpi.key \
-out /etc/ssl/glpi/glpi.crt \
-subj "/C=FR/ST=Ile-de-France/L=Paris/O=Institut F2I/CN=192.168.1.5"
```

---

### Problème 4 — Conflits entre sites Apache

**Description :** 3 sites étaient actifs en même temps et créaient des conflits.

**Cause :** Les anciens sites n'avaient pas été désactivés avant d'activer le nouveau.

```
glpi.sio.conf ← ancien site HTTP
glpi-ssl.conf ← nouveau site HTTPS
support.it-connectlab.fr.conf ← autre site
```

**Solution :**
```bash
a2dissite glpi.sio.conf
a2dissite support.it-connectlab.fr.conf
```

---

### Problème 5 — Fichier .htaccess manquant

**Description :** Apache retournait "Not Found" même avec le bon `DocumentRoot`.

**Cause :** Le fichier `.htaccess` n'existait pas dans `/var/www/glpi/public/`

**Solution :** Créer manuellement le fichier `.htaccess` avec les règles de réécriture.

---

## ✅ Résultat final

```
✅ GLPI accessible en HTTPS sur https://192.168.1.5/glpi
✅ HTTP redirigé automatiquement vers HTTPS
✅ Certificat SSL auto-signé valide 365 jours
✅ Modules Apache ssl, rewrite, headers actifs
✅ Un seul site Apache actif (glpi-ssl.conf)
✅ Fichier .htaccess configuré correctement
```

---

## 📸 Captures disponibles

| Fichier | Description |
|---|---|
| 05_certificat_ssl.PNG | Certificat SSL créé |
| 06_htaccess.PNG | Fichier .htaccess |
| 07_glpi_https.PNG | GLPI en HTTPS |
| 08_modules_apache.PNG | Modules Apache actifs |
| 09_site_actif.PNG | Site glpi-ssl.conf actif |
| 10_config_ssl.PNG | Configuration Apache SSL |

---

## 🔗 Accès aux productions

- **GitHub** : https://github.com/tomasdrill/BTS-SIO
- **GLPI HTTPS** : https://192.168.1.5/glpi

---

## 📚 Compétences travaillées

- ✅ Concevoir une solution d'infrastructure réseau
- ✅ Installer, tester et déployer une solution d'infrastructure réseau
- ✅ Exploiter, dépanner et superviser une solution d'infrastructure réseau
