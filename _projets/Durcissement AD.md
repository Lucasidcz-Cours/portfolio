---
title: "Audit et Durcissement d'un Contrôleur de Domaine"
date: 2026-11-10
cadre: "Atelier de professionnalisation"
resume: "Audit d'Active Directory via Pingcastle et durcissement de sécurité"
competences: [c2, c4]
---

## Contexte

Simulation d'audit d'un domaine d'école appelé ecole.lan, le but est de rendre l'architecture aux normes tout en gardant son bon fonctionnement. 

## Conditions et moyens

Un pc hôte, Vmware Workstation, Pingcastle, ISO Windows Server 2022 et Windows 10 pour les clients 


## Description de l'activité

L'architecture dispose de  3 unités d'organisations (Domain Controller, Postes, Comptes) et du compte tech1 pour simuler un technicien administrateur, en plus du compte Administrateur par défaut.

L'OU Postes contient les deux clients pc1 et pc1, Comptes de tech1 et Domain Controller des règles du domaine par défaut.

On constate 3 alertes principales en Admin Control, Account Takeover et Authentification protocols :

La première alerte vient du contrôle des comptes admins.
![Ce que montre la capture]({{ "/images/cyberad2.png" | relative_url }})

La deuxième des protocoles utilisés, étant trop vieux et représentant une vulnérabilité. (LM est trop daté et NTLMv1 est capturable)
![Ce que montre la capture]({{ "/images/cyberad3.png" | relative_url }})

Enfin la dernière représente différents paramètres à activer, ainsi qu'un groupe à vider.
![Ce que montre la capture]({{ "/images/cyberad4.png" | relative_url }})

Je commence par les comptes administrateurs, les comptes concernés sont indiqués dans le rapport html fournit après l'audit.
![Ce que montre la capture]({{ "/images/cyberad5.png" | relative_url }})

Le paramètre est simplement activable dans les propriétés du compte via "Utilisateurs et Ordinateurs".
![Ce que montre la capture]({{ "/images/cyberad6.png" | relative_url }})

J'ajoute ensuite les comptes dans le groupe d'utilisateurs protégés.
![Ce que montre la capture]({{ "/images/cyberad7.png" | relative_url }})

Pour les autorisations de protocoles en LAN, je dois aller dans "Stratégies locales" > "Options de sécurité"> "Sécurité réseau : Niveau d'authentification LAN Manager"
![Ce que montre la capture]({{ "/images/cyberad8.png" | relative_url }})

Je prends la dernière option, qui est la plus sécurisée via un refus direct des deux protocoles.
![Ce que montre la capture]({{ "/images/cyberad9.png" | relative_url }})

Je passe ensuite aux OU, également affichées dans le rapport.
![Ce que montre la capture]({{ "/images/cyberad10.png" | relative_url }})

J'active la protection contre la suppression accidentelle sur ces OU. (les trois Select-Object au milieu sont une erreur de syntaxe)
![Ce que montre la capture]({{ "/images/cyberad12.png" | relative_url }})A

La corbeille permettrait de récupérer des OU ou utilisateurs supprimés accidentellement.
![Ce que montre la capture]({{ "/images/cyberad11.png" | relative_url }})

Après modifications, on constate que les alertes ont disparu.
![Ce que montre la capture]({{ "/images/cyberad13.png" | relative_url }})

Note concernant les alertes jaunes (moyennes) :

- Network Topography vient de l'utilisation d'une carte NAT pour installer Pingcastle
- Obsolete OS de l'ISO windows 10 non mis à jour
- Audit du refus d'accès aux paramètres d'administration supérieurs du domaine, conformément à la documentation
- Backup de la non présence de redondance dans l'architecture 

## Ce que j'en retiens
Je continuerais le durcissement en ajoutant LAPS pour la gestion de mot de passe, séparerais les comptes administrateurs et quotidiens pour tech1 en créant adm-tech1 et installerais Bitlocker sur les clients.
