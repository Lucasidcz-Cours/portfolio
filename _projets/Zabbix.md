---
title: "Sonde température Zabbix via Docker"
date: 2026-10-14
cadre: "Atelier de professionnalisation"
resume: "Simulation et mise en place d'alerte dans l'environnement Zabbix"
competences: [c1, c2]
---

## Contexte

Simulation de cas de surveillance de température de serveur.

## Conditions et moyens

PC windows 10, Docker Desktop, Powershell (WSL), Pyton 3.12

La solution repose sur plusieurs conteneurs Docker :

MySQL 8.0 : stocke la configuration Zabbix, l'historique des mesures et les événements.
Zabbix Server : collecte les données provenant des agents, des équipements SNMP et des services HTTP.
Zabbix Web : interface d'administration accessible depuis un navigateur sur le port 8080.
Agent Linux : conteneur simulant un serveur Linux supervisé.
Switch SNMP : conteneur simulant un commutateur administrable surveillé via le protocole SNMP.

## Description de l'activité

1. Installer Zabbix en conteneurs et y déclarer 3 hôtes du poste de travail (Linux, Windows, switch en SNMP).

2. Configurer service Python expose l'URL /température, qui est lue depuis un fichier texte valeur.txt. Zabbix interroge automatiquement cette URL grâce à un agent HTTP.

3. Fixer un seuil d'avertissement à 26 °C et un seuil critique à 28 °C, puis éditer le fichier valeur.txt pour les simuler.


## Productions et preuves
Ecran d'accueil de Zabbix
![Accueil zabbix]({{ "/images/loginzabix.png" | relative_url }})
Creation des hôtes
![Ce que montre la capture]({{ "/images/hoteszabbix.png" | relative_url }})
Trigger de mémoire sur l'hôte si moins de 20% est disponible
![Ce que montre la capture]({{ "/images/trigger.png" | relative_url }})
Exemple de warning après que la température de la sonde dépasse 26 et 28 degrés
![Ce que montre la capture]({{ "/images/warning.png" | relative_url }})
Température "régulière" de la sonde définit par valeur.txt 
![Ce que montre la capture]({{ "/images/logtemperature.png" | relative_url }})
Conteneurs Docker des hôtes
![Ce que montre la capture]({{ "/images/Conteneurs.png" | relative_url }})

## Ce que j'en retiens

La difficulté principale venait du fait qu'il s'agisse de ma première utilisation de Docker et Zabbix, l'interface et la documentation officielle restent claires et m'ont permis de m'y habituer rapidement.
