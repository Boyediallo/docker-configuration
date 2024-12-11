# Guide Docker et Nginx pour Application React

## 1. Structure des fichiers

```plaintext
votre-projet/
├── src/                # Code source React
├── nginx/
│   └── nginx.conf     # Configuration Nginx
├── .env.dev           # Variables d'environnement de développement
├── Dockerfile         # Configuration Docker
└── package.json       # Dépendances Node.js
```

## 2. Explication du Dockerfile

```dockerfile
# Étape 1 : Construction
FROM node:20-alpine AS build
# Utilise Node.js 20 avec Alpine Linux comme image de base
# AS build crée une étape nommée qui pourra être référencée plus tard

WORKDIR /app
# Définit le répertoire de travail dans le conteneur

COPY package*.json ./
# Copie package.json et package-lock.json pour installer les dépendances

COPY . ./
# Copie tous les fichiers du projet dans le conteneur

RUN cp .env.dev .env
# Copie le fichier d'environnement de développement vers .env
# Cette étape est nécessaire car React utilise .env lors du build

RUN npm install --legacy-peer-deps
# Installe les dépendances Node.js
# --legacy-peer-deps permet d'éviter les conflits de dépendances

RUN npm run build
# Crée la version de production de l'application React

# Étape 2 : Production
FROM nginx:stable-alpine
# Utilise Nginx avec Alpine Linux comme image finale

COPY --from=build /app/build /usr/share/nginx/html
# Copie les fichiers buildés depuis l'étape "build" vers le dossier Nginx

COPY ./nginx/nginx.conf /etc/nginx/conf.d/default.conf
# Copie la configuration Nginx personnalisée

EXPOSE 80
# Indique que le conteneur écoutera sur le port 80

CMD ["nginx", "-g", "daemon off;"]
# Démarre Nginx en mode premier plan
```

## 3. Explication de la configuration Nginx

```nginx
server {
    listen 80;
    # Écoute sur le port 80

    location / {
        root   /usr/share/nginx/html;
        # Définit le répertoire racine des fichiers statiques
        
        index  index.html index.htm;
        # Définit les fichiers d'index par défaut
        
        try_files $uri $uri/ /index.html;
        # Redirige toutes les requêtes vers index.html pour le routing React
    }

    error_page   500 502 503 504  /50x.html;
    # Configure les pages d'erreur pour les erreurs serveur
    
    location = /50x.html {
        root   /usr/share/nginx/html;
        # Définit l'emplacement des pages d'erreur
    }
}
```

## 4. Commandes utiles

### Construction de l'image
```bash
# Construire l'image
docker build -t mon-app-react . ou docker build . -t mon-app-react ## C'est la même chose

# Lancer le conteneur
docker run -p 3000:80 mon-app-react ## -p 8080:80 est le port de l'hôte et le port du conteneur
```

### Gestion des conteneurs
```bash
# Voir les conteneurs en cours d'exécution
docker ps ## -a pour voir tous les conteneurs

# Arrêter un conteneur
docker stop <container_id>

# Supprimer un conteneur
docker rm <container_id>
```

### Debuggage
```bash
# Voir les logs du conteneur
docker logs <container_id>

# Entrer dans le conteneur
docker exec -it <container_id> /bin/sh
```

## 5. Notes importantes

- L'application sera accessible sur `http://localhost:3000` après le démarrage
- Les variables d'environnement dans `.env.dev` doivent être configurées avant le build
- Le build utilise une approche multi-stage pour réduire la taille de l'image finale
- La configuration Nginx est optimisée pour une Single Page Application (SPA) React
- Le port 80 est exposé dans le conteneur mais peut être mappé sur n'importe quel port de l'hôte

## 6. Bonnes pratiques

1. Toujours utiliser des versions spécifiques pour les images de base
2. Optimiser le cache des couches Docker
3. Minimiser le nombre de couches dans l'image
4. Ne pas exposer les variables d'environnement sensibles dans l'image finale
5. Utiliser des images légères comme Alpine
6. Mettre en place un healthcheck pour le conteneur

