# Docker Compose - Guide Détaillé

## Le fichier docker-compose.yml

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    container_name: postgres_container
    environment:
      POSTGRES_DB: demo_docker_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: mot_de_passe
    ports:
      - "5433:5432"
    networks:
      - postgres-network

  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: demo_docker_container_project
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/demo_docker_db
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: mot_de_passe
    depends_on:
      - postgres
    networks:
      - postgres-network

networks:
  postgres-network:
    driver: bridge
```

## Explication ligne par ligne

### Version et Services

```yaml
version: '3.8'
```
- Spécifie la version de la syntaxe Docker Compose
- La version 3.8 inclut les fonctionnalités modernes de Docker Compose

```yaml
services:
```
- Début de la section des services
- Chaque service deviendra un conteneur distinct

### Service PostgreSQL

```yaml
  postgres:
```
- Définit le premier service nommé "postgres"
- Ce nom sera utilisé comme hostname dans le réseau Docker

```yaml
    image: postgres:15
```
- Utilise l'image officielle PostgreSQL version 15
- Téléchargée automatiquement depuis Docker Hub

```yaml
    container_name: postgres_container
```
- Définit le nom du conteneur
- Plus facile pour la gestion et le débogage

```yaml
    environment:
```
- Début de la section des variables d'environnement
- Configure PostgreSQL au démarrage

```yaml
      POSTGRES_DB: demo_docker_db
```
- Nom de la base de données à créer automatiquement

```yaml
      POSTGRES_USER: postgres
```
- Nom d'utilisateur PostgreSQL à créer

```yaml
      POSTGRES_PASSWORD: mot_de_passe
```
- Mot de passe pour l'utilisateur PostgreSQL

```yaml
    ports:
      - "5433:5432"
```
- Mapping des ports hôte:conteneur
- 5433 sur la machine hôte vers 5432 dans le conteneur

```yaml
    networks:
      - postgres-network
```
- Connecte ce service au réseau postgres-network

### Service Application

```yaml
  app:
```
- Définit le deuxième service nommé "app"
- Service de l'application Spring Boot

```yaml
    build:
```
- Indique que ce service doit être construit localement

```yaml
      context: .
```
- Le contexte de build est le répertoire courant
- Tous les fichiers nécessaires doivent être dans ce répertoire

```yaml
      dockerfile: Dockerfile
```
- Nom du fichier Dockerfile à utiliser pour la construction

```yaml
    container_name: demo_docker_container_project
```
- Nom personnalisé pour le conteneur de l'application

```yaml
    ports:
      - "8080:8080"
```
- Expose le port 8080 de l'application
- Accessible via localhost:8080

```yaml
    environment:
```
- Variables d'environnement pour Spring Boot

```yaml
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/demo_docker_db
```
- URL de connexion à la base de données
- "postgres" fait référence au nom du service PostgreSQL

```yaml
      SPRING_DATASOURCE_USERNAME: postgres
```
- Username pour la connexion à la base de données

```yaml
      SPRING_DATASOURCE_PASSWORD: mot_de_passe
```
- Mot de passe pour la connexion à la base de données

```yaml
    depends_on:
      - postgres
```
- Assure que PostgreSQL démarre avant l'application
- Ne garantit pas que PostgreSQL soit prêt à accepter des connexions

```yaml
    networks:
      - postgres-network
```
- Connecte l'application au même réseau que PostgreSQL

### Configuration des Réseaux

```yaml
networks:
```
- Section définissant les réseaux Docker

```yaml
  postgres-network:
```
- Nom du réseau utilisé par les services

```yaml
    driver: bridge
```
- Utilise le driver bridge standard de Docker
- Permet la communication entre les conteneurs

## Commandes utiles

```bash
# Démarrer les services
docker-compose up -d

# Voir les logs
docker-compose logs

# Arrêter les services
docker-compose down

# Reconstruire les services
docker-compose up -d --build

# Voir l'état des services
docker-compose ps
```
