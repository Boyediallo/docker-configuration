# Guide : Sécurisation SSH par clés

> **Objectif** : Remplacer l'authentification par mot de passe par des clés SSH, puis désactiver les connexions root.

---

## Table des matières

1. [Créer une paire de clés SSH](#étape-1--créer-une-paire-de-clés-ssh)
2. [Copier la clé publique sur le serveur](#étape-2--copier-la-clé-publique-sur-le-serveur)
3. [Tester la connexion par clé](#étape-3--tester-la-connexion-par-clé)
4. [Sauvegarder la config SSH](#étape-4--sauvegarder-la-config-ssh)
5. [Modifier la config SSH](#étape-5--modifier-la-config-ssh)
6. [Vérifier la syntaxe](#étape-6--vérifier-la-syntaxe)
7. [Redémarrer SSH](#étape-7--redémarrer-ssh)
8. [Tester la nouvelle config](#étape-8--tester-la-nouvelle-config)
9. [Valider](#étape-9--valider)
- [Annexe A – Si tu es bloqué dehors](#annexe-a--si-tu-es-bloqué-dehors)
- [Annexe B – Fichiers importants](#annexe-b--fichiers-importants)
- [Annexe C – Commandes utiles](#annexe-c--commandes-utiles)
- [Annexe D – Aller plus loin](#annexe-d--aller-plus-loin)

---

## Étape 1 : Créer une paire de clés SSH

Sur ta machine locale, génère une paire de clés :

```bash
ssh-keygen -t ed25519 -C "ton-email@example.com"

ssh-keygen : génère une paire de clés SSH (publique + privée).
-t ed25519 : type de clé, ici ed25519 (sécurisée et rapide).
-C "..." : commentaire pour identifier la clé (souvent ton e-mail).
```

Quand le générateur te pose des questions :
- Garde le chemin par défaut (`~/.ssh/id_ed25519`)
- Mets une passphrase (recommandé)

Vérifie ensuite que les clés existent :

```bash
ls -la ~/.ssh/

ls -la : liste les fichiers du dossier ~/.ssh
```

Tu dois voir deux fichiers :

| Fichier | Description |
|---|---|
| `id_ed25519` | Ta clé **privée** — ne la partage jamais |
| `id_ed25519.pub` | Ta clé **publique** — à copier sur les serveurs |

> **Pourquoi `ed25519` ?** C'est plus sécurisé que RSA, plus court et plus rapide.

---

## Étape 2 : Copier la clé publique sur le serveur

### Méthode simple — `ssh-copy-id`

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub utilisateur@ip-du-serveur

ssh-copy-id : copie ta clé publique sur le serveur pour que tu puisses te connecter sans mot de passe.
-i ~/.ssh/id_ed25519.pub : chemin de ta clé publique.
utilisateur@ip-du-serveur : login et adresse IP du serveur.
```

Cette commande se connecte, crée le dossier `.ssh` si besoin, et ajoute ta clé automatiquement.

### Méthode manuelle

Si `ssh-copy-id` n'est pas disponible sur ta machine :

```bash
# Sur ta machine : affiche la clé publique
cat ~/.ssh/id_ed25519.pub

# Sur le serveur : crée le dossier et ajoute la clé
mkdir -p ~/.ssh --> Crée le dossier .ssh si il n’existe pas.
chmod 700 ~/.ssh --> Change les permissions du dossier .ssh pour que seul le propriétaire puisse y accéder. 
| Entity | Read (R=4) | Write (W=2) | Execute (X=1) | Total (Position) |
| ------ | ---------- | ----------- | ------------- | ---------------- |
| USER   | 4          | 2           | 1             | 7                |
| GROUP  | 4          | 2           | 1             | 7                |
| OTHERS | 4          | 2           | 1             | 7                |


echo "COLLE_TA_CLE_ICI" >> ~/.ssh/authorized_keys
Ajoute ta clé publique au fichier authorized_keys sur le serveur.

chmod 600 ~/.ssh/authorized_keys
Change les permissions du fichier pour que seul le propriétaire puisse le lire/écrire.
```

### Permissions requises

| Élément | Permission |
|---|---|
| Dossier `.ssh` | `700` |
| Fichier `authorized_keys` | `600` |

> ⚠️ Si les permissions sont incorrectes, SSH **refuse la connexion**.

---

## Étape 3 : Tester la connexion par clé

> **Important** : garde ta session SSH actuelle ouverte pendant ce test.

Ouvre un **nouveau terminal** et teste :

```bash
ssh -i ~/.ssh/id_ed25519 utilisateur@ip-du-serveur
-i : indique le chemin de la clé privée à utiliser.

Oubien 

ssh utilisateur@ip-du-serveur
Se connecter direct
```

Tu dois te connecter sans mot de passe (seulement la passphrase de ta clé, si tu en as défini une).

### Si ça ne marche pas

Vérifie l'état des fichiers sur le serveur :

```bash
ls -la ~/.ssh/ 
Lister ou Afficher les fichiers

ls -la ~/.ssh/authorized_keys

cat ~/.ssh/authorized_keys
Voire le contneue du fichier
```

Lance une connexion en mode debug :

```bash
ssh -v -i ~/.ssh/id_ed25519 utilisateur@ip-du-serveur
-v : mode verbose → affiche ce que SSH fait pour aider à trouver l’erreur.
```

### Problèmes courants

| Erreur | Cause | Solution |
|---|---|---|
| `Permission denied` | Mauvaises permissions | Refais les `chmod` |
| `No such file` | Clé pas copiée | Refais l'étape 2 |
| `Connection refused` | SSH pas lancé | `systemctl status sshd` |

---

## Étape 4 : Sauvegarder la config SSH

Avant de modifier quoi que ce soit, fais une copie de sécurité :

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

Fait une copie de sauvegarde du fichier de configuration SSH avant de le modifier.
sudo : exécute la commande avec les droits administrateur.
```

Vérifie que la sauvegarde existe :

```bash
ls -la /etc/ssh/sshd_config*
```

### En cas de problème

Pour revenir en arrière :

```bash
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart sshd
Redémarre le service SSH pour appliquer les changements.

sudo systemctl status sshd
Vérifie si le service SSH fonctionne correctement.
Tu dois voir active (running).
```

> ⚠️ **Ce n'est pas optionnel. Fais-le.**

---

## Étape 5 : Modifier la config SSH

Ouvre le fichier de configuration :

```bash
sudo nano /etc/ssh/sshd_config
```

### Paramètres à modifier

Cherche ces lignes et modifie-les (ou ajoute-les si elles n'existent pas) :

```ini
PasswordAuthentication no
PermitEmptyPasswords no
PermitRootLogin no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM no
```

### Ce que ça fait

| Paramètre | Valeur | Effet |
|---|---|---|
| `PasswordAuthentication` | `no` | Désactive l'authentification par mot de passe |
| `PermitRootLogin` | `no` | Interdit la connexion directe en root |
| `PubkeyAuthentication` | `yes` | Active l'authentification par clé SSH |

`EXEMPLE: ssh -o PubkeyAuthentication=no utilisateur@ip-du-serveur`

### Options supplémentaires (facultatives mais recommandées)

```ini
# Seuls ces utilisateurs peuvent se connecter
AllowUsers ton-utilisateur

# Déconnexion après 5 min d'inactivité
ClientAliveInterval 300
ClientAliveCountMax 2

# Maximum 3 tentatives d'authentification
MaxAuthTries 3

# Désactive le forwarding X11
X11Forwarding no
```

| Option | Effet |
|---|---|
| `AllowUsers` | Liste blanche d'utilisateurs autorisés |
| `ClientAliveInterval` | Keepalive toutes les X secondes |
| `MaxAuthTries` | Limite les tentatives de bruteforce |

---

## Étape 6 : Vérifier la syntaxe

Avant de redémarrer, vérifie que tu n'as pas fait de faute de frappe :

```bash
sudo sshd -t
```

| Résultat | Signification |
|---|---|
| Aucune sortie | Configuration valide ✅ |
| Une erreur affichée | Faute à corriger ❌ |

### Exemples d'erreurs courantes

```
# Faute de frappe dans le nom du paramètre
/etc/ssh/sshd_config: line 42: Bad configuration option: PasswordAuth
# → C'est "PasswordAuthentication", pas "PasswordAuth"

# Valeur incorrecte
/etc/ssh/sshd_config: line 58: unsupported option "oui"
# → C'est "yes" ou "no", pas "oui"
```

Pour afficher tous les paramètres actifs :

```bash
sudo sshd -T
```

---

## Étape 7 : Redémarrer SSH

```bash
# Debian / Ubuntu
sudo systemctl restart ssh

# CentOS / RHEL
sudo systemctl restart sshd
```

Vérifie que le service tourne :

```bash
sudo systemctl status sshd
```

Tu dois voir `active (running)`.

### Si SSH ne démarre pas

```bash
# Voir les logs d'erreur
sudo journalctl -u sshd -n 50

# Restaurer la configuration de sauvegarde
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart sshd
```

---

## Étape 8 : Tester la nouvelle config

> **Important** : garde ta session actuelle ouverte pendant tous les tests.

### Test 1 — Connexion par clé

```bash
ssh -i ~/.ssh/id_ed25519 utilisateur@ip-du-serveur
```

✅ **Résultat attendu** : connexion réussie.

### Test 2 — Connexion par mot de passe

```bash
ssh -o PubkeyAuthentication=no utilisateur@ip-du-serveur
```

✅ **Résultat attendu** : `Permission denied (publickey)`.

### Test 3 — Connexion root

```bash
ssh root@ip-du-serveur
```

✅ **Résultat attendu** : `Permission denied (publickey)`.

---

## Étape 9 : Valider

Vérifie que la configuration est bien appliquée :

```bash
sudo sshd -T | grep -E "passwordauthentication|permitrootlogin|pubkeyauthentication"
```

Tu dois obtenir :

```
passwordauthentication no
permitrootlogin no
pubkeyauthentication yes
```

### Récapitulatif

| Test | Commande | Résultat attendu |
|---|---|---|
| Clé SSH | `ssh -i ~/.ssh/id_ed25519 user@server` | Connexion OK |
| Mot de passe | `ssh -o PubkeyAuthentication=no user@server` | Refusé |
| Root | `ssh root@server` | Refusé |

**Ton serveur est sécurisé. ✅**

---

## Annexe A : Si tu es bloqué dehors

### Option 1 — Console web

Connecte-toi via la console de ton hébergeur (OVH, AWS, Scaleway...) directement depuis leur interface.

### Option 2 — Mode rescue

1. Redémarre en mode rescue depuis le panel de l'hébergeur
2. Monte le disque système
3. Restaure la configuration SSH :

```bash
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
```

4. Redémarre normalement

### Pour éviter ce scénario

- Garde toujours une session SSH ouverte quand tu modifies la config
- Teste dans un nouveau terminal **avant** de fermer la session courante
- Vérifie que tu as un accès console de secours chez ton hébergeur

---

## Annexe B : Fichiers importants

### Sur ta machine locale

| Fichier | Chemin | Description |
|---|---|---|
| Clé privée | `~/.ssh/id_ed25519` | Secrète — ne jamais partager |
| Clé publique | `~/.ssh/id_ed25519.pub` | À copier sur les serveurs |
| Config client | `~/.ssh/config` | Raccourcis de connexion |

### Sur le serveur

| Fichier | Chemin | Description |
|---|---|---|
| Config serveur | `/etc/ssh/sshd_config` | Configuration du daemon SSH |
| Clés autorisées | `~/.ssh/authorized_keys` | Clés publiques acceptées |
| Logs (Debian) | `/var/log/auth.log` | Logs de connexion |
| Logs (RHEL) | `/var/log/secure` | Logs de connexion |

---

## Annexe C : Commandes utiles

### Debug et monitoring

```bash
# Logs en temps réel
sudo tail -f /var/log/auth.log    # Debian/Ubuntu
sudo tail -f /var/log/secure      # CentOS/RHEL

# Utilisateurs actuellement connectés
who

# Statut du service SSH
sudo systemctl status sshd

# Connexion en mode verbose (debug)
ssh -v utilisateur@ip-du-serveur
```

### Gestion des clés

```bash
# Lister les clés chargées dans l'agent
ssh-add -l

# Ajouter une clé à l'agent SSH
ssh-add ~/.ssh/id_ed25519

# Générer une nouvelle paire de clés
ssh-keygen -t ed25519 -C "commentaire"
```

---

## Annexe D : Aller plus loin

### Autres mesures de sécurité

**1. Fail2ban** — Bloque les IP après plusieurs échecs d'authentification :

```bash
sudo apt install fail2ban
```

**2. Port knocking** — Cache le port SSH jusqu'à une séquence de paquets spécifique.

**3. Authentification à deux facteurs (2FA)** — Double authentification avec Google Authenticator.

**4. Bastion host** — Passe par un serveur intermédiaire dédié pour accéder aux autres machines.

**5. Firewall** — Limite les IP autorisées à se connecter en SSH :

```bash
sudo ufw allow from 192.168.1.0/24 to any port 22
```

### Config client pratique

Crée ou édite le fichier `~/.ssh/config` :

```ini
Host mon-serveur
    HostName ip-du-serveur
    User mon-utilisateur
    IdentityFile ~/.ssh/id_ed25519
    Port 22
```

Ensuite, tu te connectes simplement avec :

```bash
ssh mon-serveur
```
