# 🔐 Compte Rendu — OverTheWire : Bandit Wargame

> **Objectif :** Explorer et exploiter des vulnérabilités web à travers les niveaux du wargame [OverTheWire Bandit](https://overthewire.org/wargames/bandit/).  
> Ce document détaille les techniques utilisées, les vulnérabilités identifiées et les contre-mesures recommandées.

---

## 📋 Table des Matières

- [Introduction](#introduction)
- [Environnement de travail](#environnement-de-travail)
- [Niveau 0 → 1 : Connexion SSH de base](#niveau-0--1--connexion-ssh-de-base)
- [Niveau 1 → 2 : Fichier avec nom spécial](#niveau-1--2--fichier-avec-nom-spécial)
- [Niveau 2 → 3 : Espaces dans le nom de fichier](#niveau-2--3--espaces-dans-le-nom-de-fichier)
- [Niveau 3 → 4 : Fichier caché](#niveau-3--4--fichier-caché)
- [Niveau 4 → 5 : Fichier lisible parmi plusieurs](#niveau-4--5--fichier-lisible-parmi-plusieurs)
- [Niveau 5 → 6 : Recherche par propriétés](#niveau-5--6--recherche-par-propriétés)
- [Niveau 6 → 7 : Recherche sur tout le système](#niveau-6--7--recherche-sur-tout-le-système)
- [Niveau 7 → 8 : Recherche dans un fichier texte](#niveau-7--8--recherche-dans-un-fichier-texte)
- [Niveau 8 → 9 : Ligne unique dans un fichier](#niveau-8--9--ligne-unique-dans-un-fichier)
- [Niveau 9 → 10 : Données lisibles dans un binaire](#niveau-9--10--données-lisibles-dans-un-binaire)
- [Niveau 10 → 11 : Encodage Base64](#niveau-10--11--encodage-base64)
- [Niveau 11 → 12 : Chiffrement ROT13](#niveau-11--12--chiffrement-rot13)
- [Niveau 12 → 13 : Fichier compressé multiple](#niveau-12--13--fichier-compressé-multiple)
- [Niveau 13 → 14 : Clé SSH privée](#niveau-13--14--clé-ssh-privée)
- [Niveau 14 → 15 : Communication réseau (netcat)](#niveau-14--15--communication-réseau-netcat)
- [Niveau 15 → 16 : SSL/TLS avec openssl](#niveau-15--16--ssltls-avec-openssl)
- [Niveau 16 → 17 : Scan de ports](#niveau-16--17--scan-de-ports)
- [Niveau 17 → 18 : Différence entre deux fichiers](#niveau-17--18--différence-entre-deux-fichiers)
- [Niveau 18 → 19 : Shell modifié (.bashrc)](#niveau-18--19--shell-modifié-bashrc)
- [Niveau 19 → 20 : Binaire SetUID](#niveau-19--20--binaire-setuid)
- [Niveau 20 → 21 : Connexion réseau locale](#niveau-20--21--connexion-réseau-locale)
- [Récapitulatif des Vulnérabilités](#récapitulatif-des-vulnérabilités)
- [Recommandations Générales](#recommandations-générales)

---

## Introduction

**OverTheWire Bandit** est un wargame conçu pour les débutants en sécurité informatique. Chaque niveau requiert de trouver un mot de passe caché qui permet d'accéder au niveau suivant. Les compétences travaillées incluent :

- Navigation en ligne de commande Linux
- Manipulation de fichiers et permissions
- Encodage / décodage de données
- Communication réseau (SSH, netcat, SSL)
- Exploitation de configurations incorrectes

**Serveur :** `bandit.labs.overthewire.org`  
**Port SSH :** `2220`  
**Format de connexion :** `ssh bandit<N>@bandit.labs.overthewire.org -p 2220`

---

## Environnement de travail

| Élément | Détail |
|---|---|
| OS utilisé | Ubuntu / Kali Linux |
| Outils principaux | `ssh`, `cat`, `find`, `file`, `strings`, `base64`, `tr`, `netcat`, `openssl`, `nmap` |
| Navigateur | Firefox (pour les niveaux web) |
| Référence | [https://overthewire.org/wargames/bandit/](https://overthewire.org/wargames/bandit/) |

---

## Niveau 0 → 1 : Connexion SSH de base

### 🎯 Objectif
Se connecter au serveur via SSH et lire le fichier `readme` dans le répertoire home.

### 🔍 Vulnérabilité Identifiée
**Authentification par mot de passe trivial** — Le mot de passe du premier niveau est `bandit0`, identique au nom d'utilisateur.

### 🛠️ Technique Utilisée

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# Mot de passe : bandit0

cat readme
```

### 🔑 Résultat
Le fichier `readme` contient le mot de passe pour `bandit1`.

### ✅ Solution / Contre-mesure
- Ne jamais utiliser un mot de passe identique au nom d'utilisateur
- Imposer des politiques de mots de passe forts (longueur minimale, complexité)
- Préférer l'authentification par clé SSH

---

## Niveau 1 → 2 : Fichier avec nom spécial

### 🎯 Objectif
Lire le contenu d'un fichier nommé `-` (tiret).

### 🔍 Vulnérabilité Identifiée
**Mauvaise gestion des noms de fichiers spéciaux** — Le shell interprète `-` comme l'entrée standard (stdin) si on l'utilise directement avec `cat`.

### 🛠️ Technique Utilisée

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220

# ❌ Ne fonctionne pas :
cat -

# ✅ Solution : spécifier le chemin explicitement
cat ./-
# ou
cat < -
```

### 🔑 Résultat
Le mot de passe pour `bandit2` est affiché.

### ✅ Solution / Contre-mesure
- Éviter les noms de fichiers commençant par `-` dans les environnements de production
- Valider et assainir les noms de fichiers lors de l'upload ou création
- Toujours utiliser des chemins absolus dans les scripts

---

## Niveau 2 → 3 : Espaces dans le nom de fichier

### 🎯 Objectif
Lire un fichier nommé `spaces in this filename`.

### 🔍 Vulnérabilité Identifiée
**Injection via noms de fichiers avec espaces** — Les espaces dans les noms de fichiers peuvent provoquer des comportements inattendus dans les scripts shell mal écrits.

### 🛠️ Technique Utilisée

```bash
ssh bandit2@bandit.labs.overthewire.org -p 2220

# Méthode 1 : Guillemets
cat "spaces in this filename"

# Méthode 2 : Échappement
cat spaces\ in\ this\ filename

# Méthode 3 : Autocomplétion TAB
cat sp<TAB>
```

### 🔑 Résultat
Le mot de passe pour `bandit3` est affiché.

### ✅ Solution / Contre-mesure
- Bannir les espaces dans les noms de fichiers système
- Utiliser des underscores `_` ou tirets `-` à la place
- Dans les scripts, toujours entourer les variables de guillemets : `"$filename"`

---

## Niveau 3 → 4 : Fichier caché

### 🎯 Objectif
Trouver un fichier caché dans le répertoire `inhere/`.

### 🔍 Vulnérabilité Identifiée
**Sécurité par l'obscurité** — Cacher un fichier en le préfixant d'un point `.` est une pratique insuffisante comme mécanisme de sécurité.

### 🛠️ Technique Utilisée

```bash
ssh bandit3@bandit.labs.overthewire.org -p 2220

ls -la inhere/
# Affiche les fichiers cachés (préfixés par .)

cat inhere/.hidden
```

### 🔑 Résultat
Le fichier `.hidden` contient le mot de passe pour `bandit4`.

### ✅ Solution / Contre-mesure
- Ne pas compter sur le masquage de fichiers comme mécanisme de sécurité
- Protéger les fichiers sensibles avec des permissions appropriées (`chmod 600`)
- Utiliser le chiffrement pour les données sensibles

---

## Niveau 4 → 5 : Fichier lisible parmi plusieurs

### 🎯 Objectif
Trouver le seul fichier au format texte lisible parmi plusieurs fichiers dans `inhere/`.

### 🔍 Vulnérabilité Identifiée
**Stockage de données sensibles sans marquage clair** — Les mots de passe sont stockés parmi des données binaires sans contrôle d'accès différencié.

### 🛠️ Technique Utilisée

```bash
ssh bandit4@bandit.labs.overthewire.org -p 2220

# Identifier le type de chaque fichier
file inhere/*

# Le fichier de type "ASCII text" contient le mot de passe
cat inhere/-file07
```

### 🔑 Résultat
La commande `file` identifie quel fichier est en texte ASCII lisible.

### ✅ Solution / Contre-mesure
- Classifier et isoler les données sensibles
- Appliquer des permissions strictes par type de fichier
- Ne pas mélanger données sensibles et données ordinaires dans un même répertoire

---

## Niveau 5 → 6 : Recherche par propriétés

### 🎯 Objectif
Trouver un fichier dans `inhere/` avec des propriétés spécifiques : lisible par l'humain, 1033 octets, non exécutable.

### 🔍 Vulnérabilité Identifiée
**Contrôle d'accès insuffisant** — Un fichier de mot de passe accessible sans authentification supplémentaire.

### 🛠️ Technique Utilisée

```bash
ssh bandit5@bandit.labs.overthewire.org -p 2220

find inhere/ -type f -readable ! -executable -size 1033c
cat inhere/maybehere07/.file2
```

### 🔑 Résultat
La commande `find` avec les bons filtres localise le fichier contenant le mot de passe.

### ✅ Solution / Contre-mesure
- Restreindre l'accès en lecture aux fichiers sensibles (`chmod 400`)
- Chiffrer les fichiers de configuration contenant des secrets
- Utiliser des gestionnaires de secrets (HashiCorp Vault, AWS Secrets Manager)

---

## Niveau 6 → 7 : Recherche sur tout le système

### 🎯 Objectif
Trouver un fichier appartenant à `bandit7`, groupe `bandit6`, de taille 33 octets, n'importe où sur le système.

### 🛠️ Technique Utilisée

```bash
ssh bandit6@bandit.labs.overthewire.org -p 2220

find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

> `2>/dev/null` redirige les erreurs "Permission denied" pour filtrer le bruit.

### 🔑 Résultat
Le fichier est trouvé dans `/var/lib/dpkg/info/`.

### ✅ Solution / Contre-mesure
- Éviter de stocker des secrets dans des chemins standards du système
- Utiliser des permissions restrictives sur les répertoires parents
- Mettre en place des journaux d'audit pour les accès aux fichiers sensibles

---

## Niveau 7 → 8 : Recherche dans un fichier texte

### 🎯 Objectif
Trouver le mot de passe dans `data.txt` à côté du mot "millionth".

### 🛠️ Technique Utilisée

```bash
ssh bandit7@bandit.labs.overthewire.org -p 2220

grep "millionth" data.txt
```

### 🔑 Résultat
`grep` retourne la ligne contenant le mot de passe immédiatement après "millionth".

### ✅ Solution / Contre-mesure
- Ne jamais stocker des mots de passe en clair dans des fichiers texte
- Utiliser des bases de données avec hachage sécurisé (bcrypt, Argon2)

---

## Niveau 8 → 9 : Ligne unique dans un fichier

### 🎯 Objectif
Trouver la seule ligne qui n'apparaît qu'une seule fois dans `data.txt`.

### 🛠️ Technique Utilisée

```bash
ssh bandit8@bandit.labs.overthewire.org -p 2220

sort data.txt | uniq -u
```

> `sort` trie les lignes, `uniq -u` affiche uniquement les lignes uniques.

### 🔑 Résultat
Une seule ligne ressort — c'est le mot de passe.

### ✅ Solution / Contre-mesure
- Les données sensibles ne doivent pas être identifiables par leur unicité
- Ajouter du bruit (salage) aux données pour rendre ce type de recherche inefficace

---

## Niveau 9 → 10 : Données lisibles dans un binaire

### 🎯 Objectif
Extraire le mot de passe d'un fichier binaire `data.txt` parmi les chaînes lisibles.

### 🛠️ Technique Utilisée

```bash
ssh bandit9@bandit.labs.overthewire.org -p 2220

strings data.txt | grep "=="
```

> `strings` extrait les séquences de caractères ASCII lisibles d'un fichier binaire.

### 🔑 Résultat
Le mot de passe apparaît précédé de plusieurs signes `=`.

### ✅ Solution / Contre-mesure
- Ne pas intégrer de secrets dans des binaires (même obfusqués)
- Utiliser des variables d'environnement ou des fichiers de config externes
- Auditer régulièrement les binaires avec des outils d'analyse statique

---

## Niveau 10 → 11 : Encodage Base64

### 🎯 Objectif
Décoder le contenu de `data.txt` encodé en Base64.

### 🔍 Vulnérabilité Identifiée
**Confusion entre encodage et chiffrement** — Base64 est un encodage, pas un chiffrement. Il offre zéro protection.

### 🛠️ Technique Utilisée

```bash
ssh bandit10@bandit.labs.overthewire.org -p 2220

base64 -d data.txt
```

### 🔑 Résultat
Le mot de passe est immédiatement visible après décodage.

### ✅ Solution / Contre-mesure
- Ne jamais confondre encodage (Base64, Hex) et chiffrement
- Utiliser AES-256 ou des algorithmes de chiffrement approuvés pour les données sensibles
- Éduquer les développeurs sur la différence entre encodage et chiffrement

---

## Niveau 11 → 12 : Chiffrement ROT13

### 🎯 Objectif
Déchiffrer le contenu de `data.txt` chiffré avec ROT13 (rotation de 13 lettres).

### 🔍 Vulnérabilité Identifiée
**Chiffrement par substitution triviale** — ROT13 est une transformation réversible instantanément, sans clé.

### 🛠️ Technique Utilisée

```bash
ssh bandit11@bandit.labs.overthewire.org -p 2220

cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

### 🔑 Résultat
Le texte est déchiffré instantanément par la rotation inverse.

### ✅ Solution / Contre-mesure
- ROT13, César, Vigenère = obsolètes et non sécurisés
- Utiliser des algorithmes modernes : AES-GCM, ChaCha20-Poly1305
- Appliquer les standards NIST pour le chiffrement

---

## Niveau 12 → 13 : Fichier compressé multiple

### 🎯 Objectif
Extraire un fichier passé par plusieurs couches de compression (gzip, bzip2, tar).

### 🛠️ Technique Utilisée

```bash
ssh bandit12@bandit.labs.overthewire.org -p 2220

mkdir /tmp/mywork && cp data.txt /tmp/mywork/ && cd /tmp/mywork

# Convertir le hexdump en binaire
xxd -r data.txt > data.bin

# Identifier et décompresser récursivement
file data.bin          # → gzip
mv data.bin data.gz && gunzip data.gz

file data             # → bzip2
mv data data.bz2 && bunzip2 data.bz2

file data             # → tar
tar xf data

# Répéter jusqu'à obtenir un fichier ASCII text...
```

### 🔑 Résultat
Après plusieurs couches de décompression, le fichier texte final contient le mot de passe.

### ✅ Solution / Contre-mesure
- La compression n'est pas de la sécurité
- Les fichiers sensibles doivent être chiffrés, pas seulement compressés
- Utiliser GPG ou age pour chiffrer des archives

---

## Niveau 13 → 14 : Clé SSH privée

### 🎯 Objectif
Utiliser une clé SSH privée pour se connecter en tant que `bandit14`.

### 🔍 Vulnérabilité Identifiée
**Clé SSH privée accessible sans passphrase** — Une clé privée sans protection par mot de passe est un risque si elle est compromise.

### 🛠️ Technique Utilisée

```bash
ssh bandit13@bandit.labs.overthewire.org -p 2220

# Utiliser la clé privée trouvée pour se connecter au niveau suivant
ssh -i sshkey.private bandit14@localhost -p 2220

cat /etc/bandit_pass/bandit14
```

### 🔑 Résultat
Connexion réussie sans mot de passe grâce à la clé privée.

### ✅ Solution / Contre-mesure
- Protéger toujours les clés privées SSH avec une passphrase forte
- Restreindre les permissions : `chmod 400 ~/.ssh/id_rsa`
- Utiliser ssh-agent pour ne pas taper la passphrase à chaque connexion
- Activer `PasswordAuthentication no` dans `sshd_config`

---

## Niveau 14 → 15 : Communication réseau (netcat)

### 🎯 Objectif
Soumettre le mot de passe actuel au port 30000 du localhost pour obtenir le suivant.

### 🔍 Vulnérabilité Identifiée
**Service réseau sans authentification** — Un service sur port local accepte des mots de passe sans identification préalable.

### 🛠️ Technique Utilisée

```bash
ssh bandit14@bandit.labs.overthewire.org -p 2220

echo "<mot_de_passe_bandit14>" | nc localhost 30000
```

### 🔑 Résultat
Le service retourne le mot de passe pour `bandit15`.

### ✅ Solution / Contre-mesure
- Toujours authentifier les connexions avant d'accepter des données
- Limiter les services aux interfaces réseau nécessaires uniquement
- Utiliser des pare-feux pour restreindre l'accès aux ports locaux

---

## Niveau 15 → 16 : SSL/TLS avec openssl

### 🎯 Objectif
Soumettre le mot de passe via une connexion SSL au port 30001.

### 🔍 Vulnérabilité Identifiée
**Exposition de services sans chiffrement de transport** — Les niveaux précédents transmettaient en clair ; ici SSL est requis.

### 🛠️ Technique Utilisée

```bash
ssh bandit15@bandit.labs.overthewire.org -p 2220

echo "<mot_de_passe_bandit15>" | openssl s_client -connect localhost:30001 -quiet
```

### 🔑 Résultat
Le serveur SSL retourne le mot de passe pour `bandit16`.

### ✅ Solution / Contre-mesure
- Toujours chiffrer les communications réseau (TLS 1.2+ minimum)
- Valider les certificats côté client
- Désactiver SSL 2.0/3.0 et TLS 1.0/1.1 sur tous les serveurs

---

## Niveau 16 → 17 : Scan de ports

### 🎯 Objectif
Identifier parmi les ports 31000-32000 celui qui parle SSL et retourne des données utiles.

### 🛠️ Technique Utilisée

```bash
ssh bandit16@bandit.labs.overthewire.org -p 2220

# Scanner les ports ouverts dans la plage
nmap -sV localhost -p 31000-32000

# Tester les ports SSL identifiés
echo "<mot_de_passe>" | openssl s_client -connect localhost:31790 -quiet
```

### 🔑 Résultat
Le bon port retourne une clé SSH privée pour `bandit17`.

```bash
# Sauvegarder la clé et l'utiliser
mkdir /tmp/mykey && vim /tmp/mykey/key.pem
chmod 400 /tmp/mykey/key.pem
ssh -i /tmp/mykey/key.pem bandit17@localhost -p 2220
```

### ✅ Solution / Contre-mesure
- Fermer tous les ports inutilisés
- Surveiller les ports ouverts avec des outils de monitoring
- Mettre en place un IDS/IPS pour détecter les scans de ports

---

## Niveau 17 → 18 : Différence entre deux fichiers

### 🎯 Objectif
Trouver la ligne modifiée entre `passwords.old` et `passwords.new`.

### 🛠️ Technique Utilisée

```bash
ssh bandit17@bandit.labs.overthewire.org -p 2220

diff passwords.old passwords.new
```

### 🔑 Résultat
`diff` affiche la seule ligne différente — c'est le nouveau mot de passe pour `bandit18`.

### ✅ Solution / Contre-mesure
- Ne pas conserver les anciennes versions de fichiers de mots de passe
- Utiliser des systèmes de gestion de secrets avec versioning chiffré
- Supprimer les fichiers de sauvegarde sensibles après migration

---

## Niveau 18 → 19 : Shell modifié (.bashrc)

### 🎯 Objectif
Le `.bashrc` a été modifié pour déconnecter immédiatement à la connexion. Lire `readme` malgré tout.

### 🔍 Vulnérabilité Identifiée
**Backdoor via fichier de configuration shell** — Modification du `.bashrc` pour perturber l'accès légitime.

### 🛠️ Technique Utilisée

```bash
# Exécuter une commande sans démarrer de shell interactif
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"

# Alternative : démarrer un shell différent
ssh bandit18@bandit.labs.overthewire.org -p 2220 -t "/bin/sh"
```

### 🔑 Résultat
Le fichier `readme` est lu directement sans passer par le `.bashrc`.

### ✅ Solution / Contre-mesure
- Surveiller les modifications des fichiers de configuration shell
- Utiliser des systèmes de détection d'intégrité (AIDE, Tripwire)
- Restreindre les permissions d'écriture sur les fichiers de profil

---

## Niveau 19 → 20 : Binaire SetUID

### 🎯 Objectif
Utiliser un binaire avec le bit SetUID pour lire le mot de passe de `bandit20`.

### 🔍 Vulnérabilité Identifiée
**Abus de binaire SetUID** — Un binaire avec SetUID s'exécute avec les droits de son propriétaire, permettant une escalade de privilèges.

### 🛠️ Technique Utilisée

```bash
ssh bandit19@bandit.labs.overthewire.org -p 2220

ls -la bandit20-do
# -rwsr-x--- : le 's' indique le bit SetUID

./bandit20-do cat /etc/bandit_pass/bandit20
```

### 🔑 Résultat
Le binaire s'exécute en tant que `bandit20` et permet la lecture du fichier de mot de passe.

### ✅ Solution / Contre-mesure
- Auditer régulièrement les binaires SetUID : `find / -perm -4000 2>/dev/null`
- Supprimer le bit SetUID sur tous les binaires non indispensables
- Utiliser des namespaces Linux et des capabilities à la place de SetUID

---

## Niveau 20 → 21 : Connexion réseau locale

### 🎯 Objectif
Utiliser un binaire SetUID qui se connecte en local : envoyer le mot de passe actuel pour recevoir le suivant.

### 🛠️ Technique Utilisée

```bash
ssh bandit20@bandit.labs.overthewire.org -p 2220

# Terminal 1 : créer un serveur netcat qui envoie le mot de passe
echo "<mot_de_passe_bandit20>" | nc -lp 1234 &

# Terminal 2 (ou même terminal en arrière-plan) : lancer le binaire
./suconnect 1234
```

### 🔑 Résultat
Le binaire se connecte, reçoit le bon mot de passe, et retourne celui de `bandit21`.

### ✅ Solution / Contre-mesure
- Valider l'origine des connexions réseau (IP source, token d'authentification)
- Ne pas concevoir de protocoles maison basés sur la confiance implicite
- Utiliser des protocoles d'authentification mutuels (mTLS)

---

## Récapitulatif des Vulnérabilités

| Niveau | Vulnérabilité | Criticité |
|---|---|---|
| 0→1 | Mot de passe trivial (= username) | 🔴 Critique |
| 1→2 | Noms de fichiers spéciaux non gérés | 🟡 Moyen |
| 2→3 | Espaces dans noms de fichiers | 🟡 Moyen |
| 3→4 | Sécurité par l'obscurité (fichier caché) | 🔴 Critique |
| 4→5 | Données sensibles sans isolation | 🟠 Élevé |
| 5→6 | Contrôle d'accès insuffisant | 🔴 Critique |
| 6→7 | Secret dans chemin système standard | 🟠 Élevé |
| 7→8 | Mot de passe en clair dans fichier texte | 🔴 Critique |
| 9→10 | Secret dans binaire (strings extractibles) | 🔴 Critique |
| 10→11 | Confusion encodage / chiffrement (Base64) | 🔴 Critique |
| 11→12 | Chiffrement par substitution (ROT13) | 🔴 Critique |
| 13→14 | Clé SSH sans passphrase | 🔴 Critique |
| 15→16 | Communication sans chiffrement TLS | 🟠 Élevé |
| 18→19 | Backdoor via fichier de config shell | 🔴 Critique |
| 19→20 | Abus de binaire SetUID | 🔴 Critique |

---

## Recommandations Générales

### 🔐 Gestion des Mots de Passe
- Utiliser un gestionnaire de secrets (HashiCorp Vault, AWS Secrets Manager)
- Hacher les mots de passe avec bcrypt, scrypt ou Argon2
- Appliquer des politiques de rotation régulière

### 🔑 Authentification
- Désactiver l'authentification par mot de passe SSH, préférer les clés
- Protéger toutes les clés privées avec une passphrase forte
- Implémenter l'authentification multi-facteurs (MFA)

### 🗂️ Gestion des Fichiers et Permissions
- Appliquer le principe du moindre privilège
- Auditer régulièrement les permissions : `find / -perm -4000 2>/dev/null`
- Mettre en place un système de détection d'intégrité (AIDE, Tripwire)

### 🌐 Sécurité Réseau
- Chiffrer toutes les communications (TLS 1.3 minimum)
- Fermer tous les ports non nécessaires
- Déployer un IDS/IPS et surveiller les logs réseau

### 🧑‍💻 Bonnes Pratiques de Développement
- Ne jamais stocker de secrets dans le code source ou les binaires
- Utiliser des variables d'environnement ou des fichiers de config hors dépôt
- Intégrer des outils SAST/DAST dans le pipeline CI/CD

---

## Ressources

- [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Linux Command Reference](https://man7.org/linux/man-pages/)
- [SSH Best Practices](https://goteleport.com/blog/ssh-best-practices/)
- [NIST Cryptographic Standards](https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines)

---

*Rédigé dans un but éducatif — CTF & Wargame Analysis*  
*Toute exploitation de ces techniques doit se faire uniquement sur des systèmes pour lesquels vous avez une autorisation explicite.*
