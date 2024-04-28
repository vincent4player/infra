Partie 1 : Installation d'Apache sur le serveur azure.


🌞 Installer le serveur Apache

```
sudo apt update
sudo apt install apache2
```

le fichier de conf principal est /etc/httpd/conf/httpd.conf
```
sudo nano /etc/apache2/apache2.conf
```

🌞 Démarrer le service Apache

le service s'appelle httpd.

Démarrez-le avec:
```
sudo systemctl start apache2

```

Faites en sorte qu'Apache démarre automatiquement au démarrage de la machine avec :
```
sudo systemctl enable apache2

```

Vérifier qu'il a bien demarré avec :
```
sudo systemctl status apache2

```

Vérifier qu'il se lance bien au demarage avec : 
```
sudo systemctl is-enabled apache2

```


Ouvrir les ports si necessaire avec :
```
sudo ufw allow 'Apache'

```


PARTIE 2: Installation de la database pour nextcloud.


🌞 Install de MariaDB .

Installer MariaDB
```
sudo apt install mariadb-server
```

Démarer le service MariaDB :
```
sudo systemctl start mariadb
```

Démarage automatique quand la machine s'allume :
```
sudo systemctl enable mariadb
```

Renforcement de la base avec cette commande :
```
sudo mysql_secure_installation
```

Suivre les étapes de configuration de "mysql_secure_installation" sur ce lien : 
```
https://docs.rockylinux.org/guides/database/database_mariadb-server/
```

Le port utilisé par MariabDB est le 3306 :
Ouvrir les ports si necessaire.
```
sudo ufw allow 3306/tcp

```


Partie 3 : Configuration et mise en place de NextCloud


🌞 Préparation de la base pour NextCloud



Sur la machine DataBase connectez-vous à la base de données avec :
``` 
sudo mysql -u root -p
```

Exécutez les commandes SQL suivantes :
```
-- Création d'un utilisateur dans la base, avec un mot de passe
CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'pewpewpew';
```

```
-- Création de la base de donnée qui sera utilisée par NextCloud
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

```
-- On donne tous les droits à l'utilisateur nextcloud sur toutes les tables de la base qu'on vient de créer
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost';
```

```
-- Actualisation des privilèges
FLUSH PRIVILEGES;
```


🌞 Exploration de la base de données



Depuis la machine web.tp6.linux vers l'IP de db.tp6.linux

Utilisez la commande mysql pour vous connecter à une base de données, ici ce sera :
```
mysql -u nextcloud -h localhost -p
```
Si mysql n'est pas installé, utilisé la commande "sudo dnf install mysql" pour l'installer.

Une fois cela fait, effectuez ces commandes :
```
SHOW DATABASES;
USE nextcloud;
SHOW TABLES;
```


2. Serveur Web et NextCloud


la version de PHP nécessaire est la 8.0, elle est fournie par Rocky !
installation du paquet php :
```
sudo apt install php libapache2-mod-php php-mysql

```

🌞 Récupérer NextCloud

créez le dossier /var/www/vpn_nextcloud/
Ce sera notre racine web l'endroit où le site est stocké.


récupérer le fichier suivant avec une commande wget :
```
sudo wget https://download.nextcloud.com/server/releases/latest.zip -P /var/www/vpn_nextcloud/

```
Installez wget si necessaire avec "sudo apt install wget".


Extrayez tout son contenu dans le dossier /var/www/vpn_nextcloud/ :
```
sudo unzip /var/www/vpn_nextcloud/latest.zip -d /var/www/vpn_nextcloud/
```
Installez unzip si necessaire avec "sudo apt install unzip".


Déplacer les fichier unzipe dans "/var/www/vpn_nextcloud/" avec : 
```
sudo mv nextcloud/* ./
```

Donnez les bonnes permissions avec :
```
sudo chown -R www-data:www-data /var/www/vpn_nextcloud/

```

🌞 Adapter la configuration d'Apache

Modifier le fichier nextcloud.conf avec cette commande et mettez y le doc ci-dessous :
```
sudo nano /etc/apache2/sites-available/nextcloud.conf

```
```
<VirtualHost 57.151.123.79:80>
  # on indique le chemin de notre webroot
  DocumentRoot /var/www/vpn_nextcloud/
  # on précise le nom que saisissent les clients pour accéder au service
  ServerName  web.tp6.linux

  # on définit des règles d'accès sur notre webroot
  <Directory /var/www/vpn_nextcloud/> 
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

Redémarrer le service Apache pour qu'il prenne en compte le nouveau fichier de conf avec :
```
sudo a2ensite nextcloud.conf
sudo systemctl restart apache2

```


3. Finaliser l'installation de NextCloud
➜ Sur votre PC

Modifiez votre fichier hosts de votre PC pour pouvoir joindre l'IP de la VM en utilisant le nom web.tp6.linux
Avec un navigateur, visitez NextCloud à l'URL http://web.tp6.linux


🌞 Installez les deux modules PHP.

Installez les modules PHP : 
```
sudo apt install php-zip php-gd php-mysql
sudo systemctl restart apache2

```


➜ Sur votre PC

Retournez sur la page

On va vous demander un utilisateur et un mot de passe pour créer un compte admin
Ne saisissez rien pour le moment
Cliquez sur "Storage & Database" juste en dessous

choisissez "MySQL/MariaDB"
saisissez les informations pour que NextCloud puisse se connecter avec votre base


Saisissez l'identifiant et le mot de passe admin que vous voulez, et validez l'installation
