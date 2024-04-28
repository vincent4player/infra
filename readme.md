# Table des matières

1. [Objectifs](#objectifs)
2. [Présentation d'un VPN](#présentation-dun-vpn)
3. [Présentation de NextCloud, Apache et la database](#présentation-de-nextcloud-apache-et-la-database)
4. [Prérequis d'installation](#prérequis-dinstallation)
5. [Installation du serveur VPN](#installation-du-serveur-vpn)
6. [Installation d'Apache](#installation-dapache)
7. [Installation de la database (mariadb)](#installation-de-la-database-mariadb)
8. [Configuration de NextCloud](#configuration-de-nextcloud)
9. [Mise en lien VPN et NextCloud](#mise-en-lien-vpn-et-nextcloud)
10. [Difficultés rencontrées](#difficultés-rencontrées)
<br>
<br>
<br>
<br>

# Objectifs

Le principal objectif de ce projet est de déployer un environnement sécurisé sur Microsoft Azure, comprenant un serveur VPN, un serveur Apache et une instance Nextcloud. Les objectifs spécifiques sont les suivants :

1. **Création d'un serveur VPN sur Azure :**
   - Configurer un serveur VPN pour permettre des connexions sécurisées depuis des clients distants.

2. **Installation d'Apache et de Nextcloud :**
   - Déployer un serveur Apache pour servir de plateforme web.
   - Installer et configurer une instance Nextcloud pour le stockage et le partage de fichiers.

3. **Restriction d'accès à Nextcloud :**
   - Configurer les règles de pare-feu pour n'autoriser l'accès à la page web de Nextcloud qu'aux utilisateurs connectés au VPN.
   - Assurer que la page web Nextcloud ne soit pas accessible depuis l'extérieur du réseau VPN.

En mettant en place ces objectifs, nous créons un environnement sécurisé où seuls les utilisateurs authentifiés via le VPN auront accès aux données et aux services fournis par le serveur Nextcloud. Cela garantit la confidentialité et la sécurité des informations stockées et échangées via la plateforme Nextcloud.

<br>
<br>
<br>
<br>
# Présentation d'un VPN
# Qu'est-ce qu'un serveur VPN ?

Un serveur VPN (Virtual Private Network) est essentiellement un serveur dédié qui permet de créer un tunnel sécurisé et chiffré entre votre appareil et internet. Ce tunnel sécurisé assure que toutes les données échangées entre votre appareil et le serveur VPN sont cryptées, ce qui offre une protection de votre vie privée et de vos informations sensibles.

## Utilisations courantes :

### Accès sécurisé aux réseaux d'entreprise à distance :

Les entreprises utilisent souvent des VPN pour permettre à leurs employés d'accéder en toute sécurité aux ressources internes du réseau, même à distance. Lorsqu'un employé se connecte au serveur VPN de l'entreprise depuis chez lui, il peut accéder aux fichiers, aux serveurs, aux imprimantes, etc., comme s'il était physiquement au bureau. Cette méthode garantit que les données sont protégées pendant la transmission sur internet.

### Pseudo-anonymisation de l'identité en ligne :

Lorsque vous vous connectez à un VPN, votre adresse IP réelle est masquée et remplacée par l'adresse IP du serveur VPN. Cela peut aider à protéger votre vie privée en ligne en rendant plus difficile pour les sites web et les services en ligne de tracer vos activités. Par exemple, si vous utilisez un VPN avec un serveur situé en France, les sites que vous visitez penseront que vous êtes en France, même si vous êtes physiquement ailleurs.

En résumé, un serveur VPN offre un moyen sécurisé et privé de se connecter à internet et permet à des clients distants d'accéder au même réseau LAN.

<br>
<br>
<br>
<br>
# Présentation de NextCloud, Apache et la database
# 1. Présentation du setup

NextCloud est une application web codée en PHP, qui nécessite une base de données SQL pour fonctionner. Voici le setup que nous allons suivre :

### Sur le serveur azure(Apache) :
  - Installation d'un serveur Web : Apache
  - Le serveur web traite les requêtes HTTP reçues des clients
  - Il fait passer le contenu des requêtes à NextCloud
  - NextCloud décide quels fichiers HTML, CSS et JS il faut donner au client
  - Le serveur Web effectue une réponse HTTP qui contient ces fichiers HTML, CSS et JS
  - Installation de PHP pour que NextCloud puisse s'exécuter

### Sur le serveur azure(Database):
  - Installation d'un service de base de données SQL : MariaDB
  - NextCloud pourra s'y connecter
  - Nous aurons créé une base de données exprès pour lui

### Mise en place de NextCloud:
  - NextCloud est une application PHP


<br>
<br>
<br>
<br>
# Installation d'Apache
Partie 1 : Installation d'Apache sur le serveur azure.


🌞 Installer le serveur Apache

```
sudo apt update
sudo apt install apache2
```

le fichier de conf principal est /etc/apache2/apache2.conf
```
sudo nano /etc/apache2/apache2.conf
```

🌞 Démarrer le service Apache

le service s'appelle apache2.

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
<br>
<br>
<br>
<br>
# Installation de la database (mariadb)
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
<br>
<br>
<br>
<br>
# Configuration de NextCloud

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

Utilisez la commande mysql pour vous connecter à une base de données, ici ce sera :
```
mysql -u nextcloud -h localhost -p
```
Si mysql n'est pas installé, utilisé la commande "sudo apt install mysql" pour l'installer.

Une fois cela fait, effectuez ces commandes :
```
SHOW DATABASES;
USE nextcloud;
SHOW TABLES;
```


2. Serveur Web et NextCloud


la version de PHP nécessaire est la 8.0.
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
<VirtualHost *:80>
  # on indique le chemin de notre webroot
  DocumentRoot /var/www/vpn_nextcloud/
  # on précise le nom que saisissent les clients pour accéder au service
  ServerName  vpn_nextcloud

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
<br>
<br>
<br>
<br>
# Mise en lien VPN et NextCloud
# Difficultés rencontrées


Pendant la mise en œuvre du projet, plusieurs difficultés ont été rencontrées, notamment :

1. **Installation et Configuration d'Apache sur Azure :**
   - L'installation et la configuration d'Apache sur le serveur Azure ont posé des défis, notamment en ce qui concerne la gestion des autorisations et des configurations du serveur web.

2. **Configuration de Nextcloud sur Azure :**
   - La configuration de Nextcloud sur le serveur Azure a été complexe, en particulier en ce qui concerne l'intégration avec Apache et la gestion des paramètres de sécurité.

3. **Restriction d'Accès à Nextcloud via VPN :**
   - L'une des difficultés principales a été de configurer Nextcloud pour n'autoriser l'accès qu'aux utilisateurs connectés au VPN. Cette tâche a nécessité des ajustements dans la configuration de Nextcloud ainsi que dans la gestion des règles de pare-feu et des politiques d'accès sur le serveur Azure.

En surmontant ces difficultés, l'équipe a pu réussir à déployer un environnement fonctionnel sur Azure, offrant un accès sécurisé à Nextcloud uniquement aux utilisateurs connectés au VPN, tout en assurant la stabilité et la sécurité de la plateforme.
