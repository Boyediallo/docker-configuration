# Dockerfile pour Application Spring Boot - Guide Détaillé

## Le Dockerfile

```dockerfile
# Étape de build
FROM maven:3.8.4-openjdk-17 AS builder

# Copier d'abord le pom.xml
WORKDIR /build
COPY pom.xml .
COPY src ./src

# Package l'application
RUN mvn clean package -DskipTests

# Étape de production
FROM openjdk:17-slim
WORKDIR /app
COPY --from=builder /build/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

## Explication ligne par ligne

### Étape 1 : Build

```dockerfile
FROM maven:3.8.4-openjdk-17 AS builder
```
- Utilise l'image Maven avec Java 17 préinstallé
- `AS builder` nomme cette étape pour référence future
- Cette étape servira uniquement à la compilation

```dockerfile
WORKDIR /build
```
- Crée et définit le répertoire de travail `/build`
- Toutes les commandes suivantes s'exécuteront dans ce répertoire

```dockerfile
COPY pom.xml .
```
- Copie le fichier pom.xml du projet dans le conteneur
- Le point (.) représente le répertoire de travail actuel (/build)

```dockerfile
COPY src ./src
```
- Copie le dossier src du projet dans /build/src
- Contient tout le code source de l'application

```dockerfile
RUN mvn clean package -DskipTests
```
- `mvn clean` : nettoie les builds précédents
- `package` : compile et package l'application
- `-DskipTests` : ignore les tests pour accélérer le build

### Étape 2 : Production

```dockerfile
FROM openjdk:17-slim
```
- Démarre une nouvelle étape avec une image JDK 17 minimale
- Cette image sera celle de production

```dockerfile
WORKDIR /app
```
- Définit /app comme répertoire de travail pour l'application

```dockerfile
COPY --from=builder /build/target/*.jar app.jar
```
- Copie le JAR depuis l'étape "builder"
- `--from=builder` spécifie l'étape source
- Renomme le JAR en "app.jar"

```dockerfile
EXPOSE 8080
```
- Documente que l'application utilise le port 8080
- Note : cette ligne est informative, le port doit quand même être publié via docker run ou docker-compose

```dockerfile
ENTRYPOINT ["java","-jar","app.jar"]
```
- Commande exécutée au démarrage du conteneur
- Format tableau pour une meilleure gestion des arguments
- Lance l'application Spring Boot

## Avantages de ce Dockerfile

1. **Multi-stage build**
   - Réduit la taille finale de l'image
   - Sépare l'environnement de build de l'environnement de production
   - Ne conserve que le nécessaire pour l'exécution

2. **Optimisation des couches**
   - Copie d'abord pom.xml séparément pour optimiser le cache Docker
   - Minimise le nombre de couches dans l'image finale

3. **Sécurité**
   - Utilise une image slim en production
   - Moins de vulnérabilités potentielles
   - Surface d'attaque réduite

4. **Performance**
   - Image finale légère
   - Démarrage rapide
   - Consommation de ressources optimisée

## Commandes utiles

```bash
# Construire l'image
docker build -t mon-app .

# Exécuter le conteneur
docker run -p 8080:8080 mon-app

# Voir la taille de l'image
docker images mon-app

# Voir les couches de l'image
docker history mon-app
```
