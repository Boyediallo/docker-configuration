# Configuration Docker pour PostgreSQL et pgAdmin

## Structure des fichiers

### docker-compose.yml
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

## Commandes de démarrage

1. Démarrer les conteneurs principaux :
```bash
docker-compose up -d
```

2. Démarrer pgAdmin séparément :
```bash
docker run -p 5050:80 --name pgadmin_container --network postgres-network -e 'PGADMIN_DEFAULT_EMAIL=admin@admin.com' -e 'PGADMIN_DEFAULT_PASSWORD=admin' -d dpage/pgadmin4
```

## Configuration de pgAdmin

1. Accéder à pgAdmin : http://localhost:5050
2. Identifiants de connexion à pgAdmin :
   - Email : admin@admin.com
   - Mot de passe : admin

3. Configuration du serveur PostgreSQL :
   - Clic droit sur "Servers" → "Register" → "Server"
   - Onglet "General" :
     - Name : DockerPostgres
   - Onglet "Connection" :
     - Host : postgres_container
     - Port : 5432
     - Database : demo_docker_db
     - Username : postgres
     - Password : mot_de_passe

## Ports utilisés
- PostgreSQL : 5433 (externe)
- pgAdmin : 5050
- Application : 8080

## Commandes utiles

### Gestion des conteneurs
```bash
# Arrêter les services
docker-compose down

# Arrêter pgAdmin
docker stop pgadmin_container
docker rm pgadmin_container

# Voir les logs
docker logs pgadmin_container
docker-compose logs

# Voir les conteneurs en cours
docker ps

# Inspecter le réseau
docker network inspect postgres-network
```

### En cas de problème de connexion
```bash
# Reconnecter manuellement un conteneur au réseau
docker network connect postgres-network [nom_conteneur]
```
