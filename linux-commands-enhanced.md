# 🐧 Guide Complet des Commandes Linux

![Linux Commands](https://raw.githubusercontent.com/lipis/flag-icons/main/assets/docs/img/linux.svg)

## 📚 Table des Matières

1. [🖥️ Gestion Système](#gestion-système)
2. [🌐 Gestion Réseau](#gestion-réseau)
3. [🛠️ Services Système](#services-système)
4. [☕ Environnement Java](#environnement-java)
5. [📁 Gestion des Fichiers](#gestion-des-fichiers)
6. [💾 Gestion du Stockage](#gestion-du-stockage)

---

## 🖥️ Gestion Système

### Surveillance Système

| Commande | Description | Exemple d'Utilisation |
|----------|-------------|----------------------|
| `df -h` | Affiche l'espace disque | ```bash\ndf -h\n``` |
| `uptime` | Temps de fonctionnement | ```bash\nuptime\n``` |

### Administration Système

```bash
# Redémarrage système
init 6

# Arrêt système
init 0

# Configuration hostname
hostnamectl set-hotsname devserver01
```

---

## 🌐 Gestion Réseau

### Configuration IP

```bash
# Affichage configuration réseau
ip a                    # Affiche les interfaces
ip r                    # Affiche les routes

# Configuration réseau
sysctl -w net.ipv4.conf.all.forwarding=1
```

### Diagnostics Réseau

| Commande | Utilité | Exemple |
|----------|---------|---------|
| `ping -I ens19 google.fr` | Test connectivité | Vérifie la connexion via ens19 |
| `systemctl restart networking` | Redémarre le réseau | Après modification de la configuration |

---

## 🛠️ Services Système

### 📦 Installation Paquets Essentiels

```bash
apt-get install openvpn isc-dhcp-server dstat screen tcpdump \
    iptraf-ng links curl wget lftp ethtool vim jed nano net-tools
```

### 🔒 Configuration SSH

```bash
# Installation et configuration SSH
apt install openssh-server
systemctl restart ssh
systemctl status ssh
```

### 🌐 Configuration Nginx

```bash
# Installation et démarrage Nginx
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
# Installation et démarrage Nginx
# dans le front je copie juste le contenue du dochier build et apres je tape la commande sudo systemctl reload nginx
```

---

## ☕ Environnement Java

### Installation Java

```bash
# Téléchargement et installation JDK
wget https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.tar.gz
tar -xvf jdk-21_linux-x64_bin.tar.gz
sudo mv jdk-21.0.5 /opt/
```

### Configuration Environment

```bash
# Variables d'environnement Java
export JAVA_HOME=/opt/jdk-21.0.5
export PATH=$JAVA_HOME/bin:$PATH
```

### Déploiement Application

```bash
# Lancement application avec logs
java -jar mpo-0.0.1-SNAPSHOT.jar > logOutput.log 2>&1 &
```

---

## 📁 Gestion des Fichiers

### Commandes de Base

| Commande | Description | Exemple |
|----------|-------------|---------|
| `ls -al` | Liste détaillée | Affiche tous les fichiers avec détails |
| `mkdir apps` | Création répertoire | Crée un nouveau dossier |
| `rm -Rf directory` | Suppression récursive | Supprime un dossier et son contenu |

### Édition Configuration

```bash
# Édition fichiers système
nano /etc/network/interfaces
nano /etc/ssh/sshd_config
```

---

## 💾 Gestion du Stockage

### RAID Management

```bash
# Surveillance RAID
cat /proc/mdstat

# Ajout disque RAID
mdadm --manage /dev/md0 --add /dev/sdb
```

### LVM Operations

| Commande | Description |
|----------|-------------|
| `pvs` | Affiche volumes physiques |
| `vgs` | Affiche groupes de volumes |

---

## ⚠️ Bonnes Pratiques

1. 💾 **Sauvegarde**: Toujours sauvegarder avant modification
2. 📝 **Documentation**: Noter les changements effectués
3. 🔍 **Vérification**: Tester après modification
4. 🔒 **Sécurité**: Vérifier les permissions

---

## 🔧 Astuces Pro

> 💡 **Conseil**: Utilisez `screen` ou `tmux` pour les sessions longues
>
> 💡 **Sécurité**: Vérifiez toujours les logs après modification
>
> 💡 **Backup**: Créez des snapshots avant les mises à jour majeures




---

*Documentation générée le 27 novembre 2024*

*Documentation mis a jour le 24 janvier 2025*

```bash
# Pour deployer une nouvelle version de votre application Java après des développements, vous pouvez ajouter ces commandes dans un script de déploiement :

# Stopper l'application en cours
pkill -f mpo-0.0.1-SNAPSHOT.jar

# Backup de l'ancienne version (optionnel mais recommandé)
cp mpo-0.0.1-SNAPSHOT.jar backup/mpo-0.0.1-SNAPSHOT_$(date +"%Y%m%d_%H%M%S").jar

# Déployer la nouvelle version
java -jar mpo-0.0.1-SNAPSHOT.jar > logOutput.log 2>&1 &

# Pour le déploiement du front, ajoutez à votre guide cette section

# Déploiement Front
cp -r build/* /var/www/html/
sudo systemctl reload nginx

# Pour se connecter au SERVER DEV CASI par FileZilla

SERVER DEV CASI
	IP : 46.105.139.186
	user/pwd : root / Saidou
    Port: 22

# Pour se connecter au SERVER DEV CASI par un terminal

ssh username@46.105.139.186 ou username@IP
# De suite entrer le password

```

