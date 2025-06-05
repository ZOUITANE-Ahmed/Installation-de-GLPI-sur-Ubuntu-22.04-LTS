Parfait, voici une **version professionnelle, claire et bien structurée** de l’installation de **GLPI 11 + GLPI-Agent (anciennement FusionInventory)**, adaptée pour une **organisation (entreprise ou collectivité)** fonctionnant sous **Ubuntu Server 22.04 LTS**.
Ce guide est conçu pour être utilisé comme **documentation de référence interne**.

---

# 🛠️ **Déploiement de GLPI avec Inventaire Automatisé (GLPI-Agent)**

📅 Version : 2025
👤 Auteur : Ahmed ZOUITANE
🔒 Usage : Infrastructure IT interne
📘 Licence : Usage interne

---

## 📌 Sommaire

1. [Prérequis système](#1-prérequis-système)
2. [Installation de GLPI](#2-installation-de-glpi)
3. [Sécurisation de MariaDB](#3-sécurisation-de-mariadb)
4. [Configuration Apache et PHP](#4-configuration-apache-et-php)
5. [Accès Web et post-installation](#5-accès-web-et-post-installation)
6. [Installation du plugin GLPI Inventory](#6-installation-du-plugin-glpi-inventory)
7. [Installation des agents GLPI sur les postes clients](#7-installation-des-agents-glpi-sur-les-postes-clients)
8. [Structuration du parc informatique](#8-structuration-du-parc-informatique)

---

## ✅ 1. Prérequis système

* Ubuntu Server 22.04 LTS
* Apache 2
* PHP 8.3+
* MariaDB
* Accès root ou sudo
* Nom DNS configuré (ex. `glpi.entreprise.local`)

```bash
apt update && apt upgrade -y
apt install -y apache2 mariadb-server
apt install -y php php-{apcu,cli,common,curl,gd,imap,ldap,mysql,xmlrpc,xml,mbstring,bcmath,intl,zip,redis,bz2,soap} libapache2-mod-php php-cas
```

---

## 🔐 2. Installation de GLPI

```bash
cd /var/www/html
wget https://github.com/glpi-project/glpi/releases/download/11.0.0-beta/glpi-11.0.0-beta.tgz
tar -xzf glpi-11.0.0-beta.tgz
chown -R www-data:www-data glpi
chmod -R 755 glpi
```

Déplacer les répertoires critiques :

```bash
mv glpi/config /etc/glpi
mv glpi/files /var/lib/glpi
mv /var/lib/glpi/_log /var/log/glpi
```

Configurer le fichier `downstream.php` :

```sh
nano /var/www/html/glpi/inc/downstream.php
```
```php
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
    require_once GLPI_CONFIG_DIR . '/local_define.php';
}
```

Configurer `local_define.php` :

```sh
nano /etc/glpi/local_define.php
```

```php
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_DOC_DIR', GLPI_VAR_DIR);
define('GLPI_LOG_DIR', '/var/log/glpi');
```

---

## 🛡️ 3. Sécurisation de MariaDB

```bash
mysql_secure_installation
```

Configurer la base GLPI :

```sql
mysql -uroot -p
CREATE DATABASE glpi;
CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'qwerty@123';
GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost';
GRANT SELECT ON mysql.time_zone_name TO 'glpi'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Ajout des fuseaux horaires :

```bash
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql mysql
```

---

## ⚙️ 4. Configuration Apache et PHP

Créer un VirtualHost :

```sh
nano /etc/apache2/sites-available/glpi.conf
```

```apache
<VirtualHost *:80>
    ServerName glpi.entreprise.local
    DocumentRoot /var/www/html/glpi/public
    <Directory /var/www/html/glpi/public>
        Require all granted
        RewriteEngine On
        RewriteCond %{HTTP:Authorization} ^(.+)$
        RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>
</VirtualHost>
```

Activation :

```bash
a2dissite 000-default.conf
a2ensite glpi.conf
a2enmod rewrite
systemctl reload apache2
```

Configurer `php.ini` :

```sh
nano /etc/php/8.3/apache2/php.ini
```
```ini
upload_max_filesize = 20M
post_max_size = 20M
max_execution_time = 60
max_input_vars = 5000
memory_limit = 256M
session.cookie_httponly = On
date.timezone = Europe/Paris  # ⬅️ Adapter selon votre région
```

---

## 🌐 5. Accès Web et post-installation

Naviguer vers :

```
http://glpi.entreprise.local
```

* Choisir la langue
* Accepter la licence
* Vérifier les prérequis
* Saisir les identifiants MySQL (DB: `glpi`, user: `glpi`)
* Finaliser

🔐 **Sécuriser l'accès après l'installation :**

* Supprimer le dossier `install/`
* Créer les comptes utilisateurs avec rôles personnalisés

---

## 📦 6. Installation du plugin GLPI Inventory

```bash
cd /var/www/html/glpi/plugins
wget https://github.com/glpi-project/glpi-inventory-plugin/releases/download/1.5.3/glpi-glpiinventory-1.5.3.tar.bz2
tar -xvjf glpi-glpiinventory-1.5.3.tar.bz2
rm glpi-glpiinventory-1.5.3.tar.bz2
chown -R www-data:www-data glpiinventory
```

✅ Activation :

* Se connecter à GLPI
* Menu : **Configuration > Plugins**
* Installer & Activer `glpiinventory`

---

## 🖥️ 7. Installation de GLPI-Agent

### ➤ Sous Linux (Debian, Ubuntu)

```bash
cd /tmp
wget https://github.com/glpi-project/glpi-agent/releases/download/1.14/glpi-agent_1.14-2_all.deb
dpkg -i glpi-agent_1.14-2_all.deb
apt install -f
```

Configurer l’agent :

```bash
nano /etc/glpi-agent/glpi-agent.conf
# Exemple :
server=https://glpi.entreprise.local/glpi
```

Activer le service :

```bash
systemctl restart glpi-agent
systemctl enable glpi-agent
```

---

### ➤ Sous Windows

Téléchargement :

> [GLPI-Agent pour Windows](https://github.com/glpi-project/glpi-agent/releases)

Déploiement par GPO recommandé :

* Script `.bat` avec `msiexec /i glpi-agent.msi /quiet`
* Configuration du serveur via la clé de registre :

```reg
[HKEY_LOCAL_MACHINE\Software\GLPI-Agent]
"server"="https://glpi.entreprise.local/glpi"
```

---

## 🗂️ 8. Structuration du parc informatique

### ✔️ Bonnes pratiques

* Créer des **entités (RH, IT, Direction, etc.)**
* Définir des **modèles de matériel**
* Déployer des agents en masse (GPO, Ansible, etc.)
* Activer l’**inventaire automatique planifié** (ex. toutes les nuits)
* Définir des **règles d’importation automatique**
* Activer les notifications (alertes, seuils de stock, etc.)

---

## 📄 Documentation associée (à prévoir)

* 🔐 **Politique d’accès GLPI** (rôles, utilisateurs, audits)
* 🧰 **Guide de sauvegarde automatique GLPI + MariaDB**
* 🖥️ **Procédure d’ajout de nouveaux postes via GLPI-Agent**
* 📊 **Rapports personnalisés pour la DSI**

---

## ✅ Conclusion

Votre infrastructure GLPI est maintenant **fonctionnelle et prête pour l’inventaire automatisé**.
Assurez-vous de mettre en place :

* 🔄 Un plan de mise à jour régulier
* 📦 Un système de sauvegarde complet
* 🔒 Une politique de gestion des accès rigoureuse

---

Souhaitez-vous recevoir ce guide en **PDF professionnel prêt à partager** dans votre organisation ?
