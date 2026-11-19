# RAPPORT DE TRAVAUX PRATIQUES

## Mise en place du protocole SSH

Réalisé par : Fatou Niang

DGI-ESP / UCAD

Licence 3 GLSI – Services Réseaux Avancés

Année universitaire 2026-2027

Enseignant : Dr. GUEYE

---

## Introduction

Ce TP a pour but de sécuriser l'accès à une machine distante avec le protocole SSH. Nous avons installé un serveur SSH sur Ubuntu, nous nous y sommes connectés depuis un client Windows, nous avons testé les clés SSH, puis nous avons créé des tunnels SSH pour accéder à un serveur Web Apache et à une base de données MariaDB.

## Environnement de travail

Serveur : Ubuntu Server, adresse IP 192.168.1.26, utilisateur fatou.

Client 1 : Windows (PowerShell).

Client 2 : Ubuntu Desktop, adresse IP 192.168.1.35.

## 1. Installation du serveur SSH

Sur le serveur Ubuntu, nous avons installé OpenSSH et activé le service :

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

Le statut du service montre qu'il est actif :

```bash
sudo systemctl status ssh
```

![Capture 1](media/image1.png)

Nous avons ensuite vérifié que le port 22 est ouvert :

```bash
sudo ss -tlnp | grep :22
```

![Capture 2](media/image2.png)

## 2. Connexion SSH depuis le client Windows

Depuis PowerShell, nous nous sommes connectés au serveur (192.168.1.26) avec le compte fatou :

```bash
ssh fatou@192.168.1.26
```

Après avoir accepté l'empreinte du serveur et saisi le mot de passe, nous avons obtenu le terminal du serveur.

![Capture 3](media/image3.png)

![Capture 4](media/image4.png)

## 3. Configuration des clés SSH

Nous avons généré une paire de clés avec rsa :

```bash
ssh-keygen -t rsa
```

![Capture 5](media/image5.png)

Depuis le serveur, la clé publique id_rsa.pub a été envoyée dans le répertoire /home de la machine cliente Linux (192.168.1.35) avec scp :

```bash
scp id_rsa.pub fatou@192.168.1.35:/home/fatou
```

![Capture 6](media/image6.png)

Nous avons ensuite testé la connexion depuis le client Ubuntu. Le mot de passe n'est pas demandé.

```bash
ssh fatou@192.168.1.26
```

![Capture 7](media/image7.png)

## 4. Tunnel SSH vers le serveur Web

Nous avons installé Apache sur le serveur et vérifié son fonctionnement en local :

```bash
sudo apt install apache2
curl http://127.0.0.1
```

![Capture 8](media/image8.png)

Depuis Windows, nous avons créé un tunnel entre le port local 1024 et le port 80 du serveur :

```bash
ssh -L 1024:127.0.0.1:80 fatou@192.168.1.26
```

![Capture 9](media/image9.png)

Nous avons ensuite ouvert le navigateur Windows à l'adresse http://127.0.0.1:1024. La page par défaut d'Apache s'affiche : le tunnel fonctionne.

![Capture 10](media/image10.png)

## 5. Tunnel SSH vers la base de données MariaDB

Sur le serveur, nous avons installé MariaDB (sudo apt install mariadb-server) puis créé un utilisateur pour la connexion à distance :

```sql
CREATE USER 'tpuser'@'127.0.0.1' IDENTIFIED BY '********';
GRANT ALL PRIVILEGES ON *.* TO 'tpuser'@'127.0.0.1';
FLUSH PRIVILEGES;
```

![Capture 11](media/image11.png)

Depuis Windows, nous avons créé un second tunnel entre le port local 3307 et le port 3306 du serveur :

```bash
ssh -L 3307:127.0.0.1:3306 fatou@192.168.1.26
```

![Capture 12](media/image12.png)

Dans un autre terminal, nous nous sommes connectés à la base à travers le tunnel :

```bash
mysql -h 127.0.0.1 -P 3307 -u tpuser -p
```

La version affichée (11.8.6-MariaDB Ubuntu) prouve que nous sommes bien connectés à la base du serveur et non à celle de XAMPP sur Windows.

![Capture 13](media/image13.png)

## Conclusion

Ce TP nous a permis d'installer un serveur SSH, de nous y connecter depuis Windows, de tester l'authentification par clés et de faire passer des services Web et base de données dans un tunnel SSH. Tous les tests ont fonctionné.
