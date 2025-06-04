# Installation de GLPI sur Ubuntu 22.04 LTS

## Prérequis

- **Système** : Ubuntu Server 22.04 LTS
- **Serveur Web** : Apache
- **Interpréteur** : PHP
- **Base de données** : MariaDB

## Étapes d'installation

### 1. Mise à jour et installation des composants

```sh
apt update && apt upgrade -y
apt install -y apache2 php php-{apcu,cli,common,curl,gd,imap,ldap,mysql,xmlrpc,xml,mbstring,bcmath,intl,zip,redis,bz2} libapache2-mod-php php-soap php-cas
apt install -y mariadb-server
```

⚠ **Assurez-vous que votre pare-feu autorise les connexions SSH, HTTP et HTTPS.**

---

### 2. Configuration de la base de données

Sécuriser l'installation de MariaDB :

```sh
mysql_secure_installation
```

Répondez aux questions comme suit :

- Définir un mot de passe root : **Oui**
- Supprimer les utilisateurs anonymes : **Oui**
- Désactiver la connexion root à distance : **Oui**
- Supprimer la base de test : **Oui**
- Recharger les privilèges : **Oui**

Ajout des informations de fuseaux horaires :

```sh
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql mysql
```

Créer la base de données et l'utilisateur GLPI :

```sh
mysql -uroot -p
CREATE DATABASE glpi;
CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'qwerty';
GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost';
GRANT SELECT ON mysql.time_zone_name TO 'glpi'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

⚠ **Utilisez un mot de passe sécurisé pour l'utilisateur ****\`\`****.**

---

### 3. Téléchargement et préparation des fichiers

```sh
cd /var/www/html
wget https://github.com/glpi-project/glpi/releases/download/11.0.0-beta/glpi-11.0.0-beta.tgz
tar -xvzf glpi-11.0.0-beta.tgz
sudo chown -R www-data:www-data /var/www/html/glpi
sudo chmod -R 755 /var/www/html/glpi
```

Organisation des fichiers :

```sh
mv /var/www/html/glpi/config /etc/glpi
mv /var/www/html/glpi/files /var/lib/glpi
mv /var/lib/glpi/_log /var/log/glpi
```

Créer le fichier `downstream.php` :

```sh
nano /var/www/html/glpi/inc/downstream.php
```

Ajouter :

```php
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
    require_once GLPI_CONFIG_DIR . '/local_define.php';
}
```

Créer le fichier `local_define.php` :

```sh
nano /etc/glpi/local_define.php
```

Ajouter :

```php
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_DOC_DIR', GLPI_VAR_DIR);
define('GLPI_LOG_DIR', '/var/log/glpi');
```

---

### 4. Permissions des fichiers et dossiers

```sh
chown root:root /var/www/html/glpi/ -R
chown www-data:www-data /etc/glpi -R
chown www-data:www-data /var/lib/glpi -R
chown www-data:www-data /var/log/glpi -R
chown www-data:www-data /var/www/html/glpi/marketplace -Rf
find /var/www/html/glpi/ -type f -exec chmod 0644 {} \;
find /var/www/html/glpi/ -type d -exec chmod 0755 {} \;
find /etc/glpi -type f -exec chmod 0644 {} \;
find /etc/glpi -type d -exec chmod 0755 {} \;
find /var/lib/glpi -type f -exec chmod 0644 {} \;
find /var/lib/glpi -type d -exec chmod 0755 {} \;
find /var/log/glpi -type f -exec chmod 0644 {} \;
find /var/log/glpi -type d -exec chmod 0755 {} \;
```

---

### 5. Configuration du serveur web Apache

Créer un VirtualHost :

```sh
nano /etc/apache2/sites-available/glpi.conf
```

Ajouter :

```apache
<VirtualHost *:80>
    ServerName serverweb.glpi.local
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

Activer la configuration :

```sh
a2dissite 000-default.conf
a2enmod rewrite
a2ensite glpi.conf
systemctl restart apache2
```

Configuration PHP :

```sh
nano /etc/php/8.3/apache2/php.ini
```

Modifier :

```ini
upload_max_filesize = 20M
post_max_size = 20M
max_execution_time = 60
max_input_vars = 5000
memory_limit = 256M
session.cookie_httponly = On
date.timezone = Your/Timezone
```

⚠ **Remplacez ****\`\`**** par votre fuseau horaire.**

---

### 6. Lancement de l’installation Web

Ouvrir un navigateur et accéder à :

```
```

Suivre l'assistant d'installation :

Choisir la langue

Accepter la licence

Vérification des prérequis

Configuration de la base de données (utiliser les identifiants créés précédemment)

Finaliser l'installation
Suivez les instructions pour finaliser l'installation.

---

## Conclusion

Votre installation de GLPI est maintenant prête. Pensez à : ✅ Modifier les paramètres de sécurité après installation. ✅ Mettre à jour régulièrement votre serveur et GLPI. ✅ Configurer des sauvegardes automatiques de votre base de données.

Bonne gestion avec GLPI ! 🎉

Parfait, organisons toute l’installation de **GLPI avec GLPI-Agent (ex-FusionInventory)** dans une structure claire adaptée à une **organisation (entreprise ou institution)**. Voici un plan **professionnel, complet et structuré** pour le déploiement.

---

## 🏢 **Installation de GLPI + GLPI-Agent dans une Organisation**

---

### 🔹 **1. Préparation de l’environnement GLPI**



### 🔹 **Installation du plugin GLPI-Inventory (Remplace FusionInventory)**

#### 📥 Télécharger et installer le plugin

```bash
cd /var/www/html/glpi/plugins
wget https://github.com/glpi-project/glpi-inventory-plugin/releases/download/1.5.3/glpi-glpiinventory-1.5.3.tar.bz2
sudo tar -xvjf glpi-glpiinventory-1.5.3.tar.bz2
sudo rm glpi-glpiinventory-1.5.3.tar.bz2
sudo chown -R www-data:www-data glpiinventory
```

#### 🌐 Activer via l’interface web

1. Se connecter à GLPI (interface web)
2. Aller dans **Configuration > Plugins**
3. Cliquer sur **Installer** puis **Activer** le plugin `glpi-inventory`

---

### 🔹 ** Installation de l’agent GLPI sur les machines clientes**

#### 🖥️ Sous Linux

```bash
cd /tmp
wget https://github.com/glpi-project/glpi-agent/releases/download/1.14/glpi-agent_1.14-2_all.deb
sudo dpkg -i glpi-agent_1.8.1_all.deb
sudo apt install -f
```

Configurer l'URL du serveur GLPI :

```bash
sudo nano /etc/glpi-agent/glpi-agent.conf
# Ajouter :
server=https://glpi.mondomaine.com/glpi
```

Redémarrer le service :

```bash
sudo systemctl restart glpi-agent
sudo systemctl enable glpi-agent
```



### 🔹 ** Organisation du parc informatique dans GLPI**

#### 🧩 Recommandations :

* Créez des **Entités** pour chaque département ou site
* Utilisez des **Modèles de matériel**
* Déployez les **agents via GPO (Windows)** ou **Ansible (Linux)** en masse
* Ajoutez des **règles d’importation automatique** pour les ordinateurs
* Planifiez des **inventaires périodiques** (ex. chaque nuit à 2h)

---




