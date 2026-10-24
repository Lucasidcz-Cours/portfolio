---
title: Mise en place d'une plateforme de gestion d'incident 
date: 24/10/2026 
cadre: Projet de support et mise à disposition de service informatique 
resume: Mise en place de plateforme de gestion d'incident GLPI 
competences: [c5,c6]
---

Contexte 
L'examen final de BTS SIO demande une plateforme informatique mise en place par l'étudiant. Je présenterai ici l'infrastructure utilisée pour la simulation d'incidents à l'aide de GLPI.

## Conditions et moyensc##
Un ordinateur portable
VMware Workstation
ISO Windows Server 2022
ISO Windows 10
ISO Debian 12
GLPI Agent

Machines virtuelles
Un serveur Active Directory hébergeant le domaine stadiumcompany.local
Un serveur Debian 12 hébergeant GLPI
Un poste client Windows joint au domaine
Les trois machines virtuelles sont connectées au même réseau virtuel VMware : lan10.

Description de l'activité 
1. Configuration de la machine Debian
Le serveur GLPI est placé sur le réseau 172.20.1.0/24, le même que celui du contrôleur de domaine (172.20.1.2).

La machine possède une seconde carte réseau en NAT permettant l'accès à Internet pour télécharger les paquets nécessaires.

Configuration du fichier réseau :

nano /etc/network/interfaces
allow-hotplug ens33
iface ens33 inet dhcp

allow-hotplug ens34
iface ens34 inet static
address 172.20.1.7/24Description de l'activité
1. Configuration de la machine Debian
Le serveur GLPI est placé sur le réseau 172.20.1.0/24, le même que celui du contrôleur de domaine (172.20.1.2).

La machine possède une seconde carte réseau en NAT permettant l'accès à Internet pour télécharger les paquets nécessaires.

Configuration du fichier réseau :

nano /etc/network/interfaces
allow-hotplug ens33
iface ens33 inet dhcp


allow-hotplug ens34
iface ens34 inet static
address 172.20.1.7/24

Redémarrage du service réseau :

systemctl restart networking
Installation d'Apache, PHP et MariaDB :

apt install apache2 php libapache2-mod-php mariadb-server -y
Lancement du script de sécurisation MariaDB :

mysql_secure_installation
Configuration retenue :

Enter current password for root (enter for none): Entrée

Switch to unix_socket authentication [Y/n] n

Change the root password? [Y/n] Y
New password: root
Re-enter new password: root

Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y

2. Installation et configuration de GLPI
Installation des extensions PHP nécessaires :

apt install php-{ldap,apcu,mysql,mbstring,curl,gd,xml,intl,bz2,zip,bcmath} -y
systemctl restart apache2
Création de la base de données GLPI :

CREATE DATABASE dbglpi;
Création du compte dédié :

GRANT ALL PRIVILEGES ON dbglpi.* TO 'userglpi'@'localhost'
IDENTIFIED BY 'userglpi';
Application des droits :

FLUSH PRIVILEGES;
Téléchargement de GLPI :

wget https://github.com/glpi-project/glpi/releases/download/11.0.9/glpi-11.0.9.tgz
Décompression :

tar -xzf glpi-11.0.9.tgz -C /var/www/html

Attribution des droits :

chown -R www-data:www-data /var/www/html/glpi
chmod -R 775 /var/www/html/glpi
À partir de GLPI 11, Apache doit pointer vers /var/www/html/glpi/public.

Création d'un enregistrement DNS sur le contrôleur de domaine :

glpi.stadiumcompany.local → 172.20.1.7
Création du certificat SSL autosigné :

make-ssl-cert /usr/share/ssl-cert/ssleay.cnf \
/etc/ssl/private/stadiumcompany.pem
Nom d'hôte utilisé :

glpi.stadiumcompany.local
Noms alternatifs :

DNS:glpi.stadiumcompany.local
IP:172.20.1.7

Activation du SSL et du site Apache :

a2enmod ssl
systemctl restart apache2

a2ensite glpi.conf
systemctl reload apache2
GLPI devient alors accessible à l'adresse :

https://glpi.stadiumcompany.local
L'assistant d'installation est ensuite utilisé avec les paramètres suivants :

Serveur SQL : localhost
Base : dbglpi
Utilisateur : userglpi
Mot de passe : userglpi

3. Mise en place de la simulation d'incident
Pour permettre l'authentification des utilisateurs du domaine, GLPI est synchronisé avec l'annuaire LDAP :

Configuration → Authentification → Liaisons LDAP
Deux utilisateurs de test sont créés dans Active Directory :

kaiser
césar
Ces utilisateurs sont ensuite importés dans GLPI via :

Administration → Utilisateurs → Importation depuis une source externe
Ils peuvent ensuite ouvrir des tickets directement depuis leur compte du domaine.

Pour faire remonter automatiquement l'inventaire d'un poste, GLPI Agent est installé puis configuré pour envoyer ses informations à :

https://glpi.stadiumcompany.local/front/inventory.php

Productions et preuves
Active Directory
![Active Directory]({{ "/images/AD.png" | relative_url }})

Fichier de configuration Apache/GLPI
![Fichier de configuration]({{ "/images/glpiconf.png" | relative_url }})

Paramètres LDAP
![Paramètres LDAP]({{ "/images/LDAP.png" | relative_url }})

Nouveaux utilisateurs dans Active Directory
![Users AD]({{ "/images/users import ad.png" | relative_url }})

Utilisateurs importés dans GLPI
![Users dans glpi]({{ "/images/users import glpi.png" | relative_url }})

Création de ticket par l'utilisateur Kaiser
![Ticket créé Kaiser]({{ "/images/ticketkaiser.png" | relative_url }})

Ticket visible dans GLPI
![Ticket visible GLPI]({{ "/images/ticketkaiser2.png" | relative_url }})

Ce que j'en retiens
Cette activité m'a permis de déployer une solution complète de gestion d'incidents avec GLPI. J'ai mis en œuvre un serveur web Apache, une base de données MariaDB, sécurisé l'accès via HTTPS et intégré l'authentification LDAP avec Active Directory.

Difficulté rencontrée
Lors de l'installation de GLPI 11, Apache ne parvenait pas à accéder correctement au répertoire glpi/public. Après analyse de la documentation et de la configuration Apache, l'activation du module rewrite ainsi que l'adaptation du VirtualHost ont permis de résoudre le problème.
