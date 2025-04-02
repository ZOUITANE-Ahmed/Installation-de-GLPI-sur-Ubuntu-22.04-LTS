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
CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'yourstrongpassword';
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
wget https://github.com/glpi-project/glpi/releases/download/10.0.18/glpi-10.0.18.tgz
tar -xvzf glpi-10.0.18.tgz
```

Organisation des fichiers :

```sh
mv /var/www/html/glpi/config /etc/glpi
mv /var/www/html/glpi/files /var/lib/glpi
mv /var/lib/glpi/_log /var/log/glpi
```

Créer le fichier `downstream.php` :

```sh
vim /var/www/html/glpi/inc/downstream.php
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
vim /etc/glpi/local_define.php
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
vim /etc/apache2/sites-available/glpi.conf
```

Ajouter :

```apache
<VirtualHost *:80>
    ServerName yourglpi.yourdomain.com
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
vim /etc/php/8.1/apache2/php.ini
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
http://yourglpi.yourdomain.com
```

Suivez les instructions pour finaliser l'installation.

---

## Conclusion

Votre installation de GLPI est maintenant prête. Pensez à : ✅ Modifier les paramètres de sécurité après installation. ✅ Mettre à jour régulièrement votre serveur et GLPI. ✅ Configurer des sauvegardes automatiques de votre base de données.

Bonne gestion avec GLPI ! 🎉

