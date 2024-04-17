Partie 1 : Installation d'Apache sur la machine Web.


🌞 Installer le serveur Apache

```
sudo dnf install httpd
```

le fichier de conf principal est /etc/httpd/conf/httpd.conf
```
sudo nano /etc/httpd/conf/httpd.conf
```

🌞 Démarrer le service Apache

le service s'appelle httpd.

Démarrez-le avec:
```
sudo systemctl start httpd.service
```

Faites en sorte qu'Apache démarre automatiquement au démarrage de la machine avec :
```
sudo systemctl enable httpd.service
```

Vérifier qu'il a bien demarré avec :
```
sudo systemctl status httpd
```

Vérifier qu'il se lance bien au demarage avec : 
```
sudo systemctl is-enabled httpd
```


Ouvrir les ports si necessaire avec :
```
sudo firewall-cmd --permanent --add-port=80/tcp    (le port 80 est le port sur lequel apache écoute)
sudo firewall-cmd --reload
```


PARTIE 2: Sur la DataBase.


🌞 Install de MariaDB sur db.tp6.linux

Installer MariaDB
```
sudo dnf install mariadb-server
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
sudo firewall-cmd --permanent --add-port=3306/tcp    (le port 3306est le port sur lequel MariabDB écoute)
sudo firewall-cmd --reload
```


Partie 3 : Configuration et mise en place de NextCloud



Partie 3 : Configuration et mise en place de NextCloud

🌞 Préparation de la base pour NextCloud



Sur la machine DataBase connectez-vous à la base de données avec :
``` 
sudo mysql -u root -p
```

Exécutez les commandes SQL suivantes :
```
-- Création d'un utilisateur dans la base, avec un mot de passe
CREATE USER 'nextcloud'@'10.6.1.11' IDENTIFIED BY 'pewpewpew';
```

```
-- Création de la base de donnée qui sera utilisée par NextCloud
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

```
-- On donne tous les droits à l'utilisateur nextcloud sur toutes les tables de la base qu'on vient de créer
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.6.1.11';
```

```
-- Actualisation des privilèges
FLUSH PRIVILEGES;
```


🌞 Exploration de la base de données



Depuis la machine web.tp6.linux vers l'IP de db.tp6.linux

Utilisez la commande mysql pour vous connecter à une base de données, ici ce sera :
```
mysql -u nextcloud -h 10.6.1.12 -p
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
sudo dnf install php
```

🌞 Récupérer NextCloud

créez le dossier /var/www/tp6_nextcloud/
Ce sera notre racine web l'endroit où le site est stocké.


récupérer le fichier suivant avec une commande wget :
```
sudo wget https://download.nextcloud.com/server/releases/latest.zip -P /var/www/tp6_nextcloud/
```
Installez wget si necessaire avec "sudo dnf install wget".


Extrayez tout son contenu dans le dossier /var/www/tp6_nextcloud/ :
```
sudo unzip /var/www/tp6_nextcloud/latest.zip -d /var/www/tp6_nextcloud/
```
Installez unzip si necessaire avec "sudo dnf install unzip".


Déplacer les fichier unzipe dans "/var/www/tp6_nextcloud/" avec : 
```
sudo mv nextcloud/* ./
```

Donnez les bonnes permissions avec :
```
sudo chown -R apache:apache /var/www/tp6_nextcloud/
```

🌞 Adapter la configuration d'Apache

Modifier le fichier nextcloud.conf avec cette commande et mettez y le doc ci-dessous :
```
sudo nano /etc/httpd/conf.d/nextcloud.conf
```
```
<VirtualHost *:80>
  # on indique le chemin de notre webroot
  DocumentRoot /var/www/tp6_nextcloud/
  # on précise le nom que saisissent les clients pour accéder au service
  ServerName  web.tp6.linux

  # on définit des règles d'accès sur notre webroot
  <Directory /var/www/tp6_nextcloud/> 
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
sudo systemctl restart httpd
```


3. Finaliser l'installation de NextCloud
➜ Sur votre PC

Modifiez votre fichier hosts de votre PC pour pouvoir joindre l'IP de la VM en utilisant le nom web.tp6.linux
Avec un navigateur, visitez NextCloud à l'URL http://web.tp6.linux


🌞 Installez les deux modules PHP.

Installez les modules PHP : 
```
sudo dnf install php-zip php-gd php-pdo php-mysqlnd
sudo systemctl restart httpd
```


➜ Sur votre PC

Retournez sur la page

On va vous demander un utilisateur et un mot de passe pour créer un compte admin
Ne saisissez rien pour le moment
Cliquez sur "Storage & Database" juste en dessous

choisissez "MySQL/MariaDB"
saisissez les informations pour que NextCloud puisse se connecter avec votre base


Saisissez l'identifiant et le mot de passe admin que vous voulez, et validez l'installation
