# Application Smart Home - Conteneurisation Spring Boot & Angular

## Description

Ce projet démontre la conteneurisation d'une application Smart Home full-stack utilisant Spring Boot pour le backend, Angular pour le frontend, et MySQL pour la base de données. L'application est orchestrée via Docker Compose pour un déploiement simplifié.

## Prérequis

- Docker (version 20.10 ou supérieure)
- Docker Compose (version 2.0 ou supérieure)

## Structure du Projet

```
.
├── Smart_Home_back/          # Backend Spring Boot
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/
├── smartHome-front/          # Frontend Angular
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── docker-compose.yml        # Configuration Docker Compose
└── README.md
```

## Configuration Docker

![Docker Files Overview](https://github.com/user-attachments/assets/8dde9e52-a5d6-491d-840b-82510d304328)

### Dockerfiles

#### Backend

```dockerfile
# Stage 1: Build with Maven
FROM maven:3.9.9-eclipse-temurin-21 AS builder
WORKDIR /app
COPY ./src ./src
COPY ./pom.xml .
RUN mvn clean package -DskipTests

# Stage 2: Create the final image
FROM eclipse-temurin:21-jdk-alpine
VOLUME /tmp
ARG JAR_FILE=target/*.jar
COPY --from=builder /app/${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

**Explication :**

- **Étape 1 : Build avec Maven**
  - `FROM maven:3.9.9-eclipse-temurin-21 AS builder` : Utilise l'image Maven 3.9.9 avec Eclipse Temurin OpenJDK 21 pour la construction.
  - `WORKDIR /app` : Définit le répertoire de travail.
  - `COPY ./src ./src` et `COPY ./pom.xml .` : Copie les sources et le fichier de configuration Maven.
  - `RUN mvn clean package -DskipTests` : Construit le JAR en sautant les tests pour accélérer le build.

- **Étape 2 : Image finale**
  - `FROM eclipse-temurin:21-jdk-alpine` : Image légère avec OpenJDK 21.
  - `COPY --from=builder /app/${JAR_FILE} app.jar` : Copie le JAR depuis l'étape de build.
  - `ENTRYPOINT ["java","-jar","/app.jar"]` : Lance l'application.

#### Frontend

```dockerfile
# Stage 1: Build with Node.js
FROM node:20-alpine as builder
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine
COPY --from=builder /app/dist/smart-home /usr/share/nginx/html
```

**Explication :**

- **Étape 1 : Build avec Node.js**
  - `FROM node:20-alpine as builder` : Image Node.js 20 pour la construction.
  - `COPY . .` : Copie tous les fichiers du projet.
  - `RUN npm install` et `RUN npm run build` : Installe les dépendances et construit l'application.

- **Étape 2 : Serveur Nginx**
  - `FROM nginx:alpine` : Image Nginx légère.
  - `COPY --from=builder /app/dist/smart-home /usr/share/nginx/html` : Copie les fichiers construits vers Nginx.

### Docker Compose

```yaml
services:
  mysql:
    image: mysql:latest
    container_name: smarthouse-mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: smart-house
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - smarthouse-network

  backend:
    build:
      context: ./Smart_Home_back
    container_name: smarthouse-backend
    ports:
      - "8085:8085"
    depends_on:
      mysql:
        condition: service_started
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/smart-house
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: root
    healthcheck:
      test: [ "CMD", "curl", "-f", "http://localhost:8085/actuator/health" ]
      interval: 30s
      timeout: 10s
      retries: 5
    networks:
      - smarthouse-network
    restart: on-failure

  frontend:
    build:
      context: ./smartHome-front
    container_name: smarthouse-frontend
    ports:
      - "80:80"
    depends_on:
      backend:
        condition: service_started
    networks:
      - smarthouse-network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: smarthouse-phpmyadmin
    environment:
      PMA_HOST: mysql
      PMA_PORT: 3306
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "8081:80"
    depends_on:
      - mysql
    networks:
      - smarthouse-network

networks:
  smarthouse-network:
    driver: bridge

volumes:
  mysql-data:
```

**Explication :**

- **mysql** : Base de données avec volume persistant et réseau isolé.
- **backend** : Service Spring Boot avec healthcheck et redémarrage automatique.
- **frontend** : Service Angular servi par Nginx.
- **phpmyadmin** : Interface web pour MySQL.
- **networks** : Réseau bridge pour l'isolation.
- **volumes** : Volume pour la persistance des données MySQL.

## Installation et Lancement

1. **Cloner le projet :**
   ```bash
   git clone https://github.com/lachgar/smarthouse.git
   cd smarthouse
   ```

2. **Construire les images :**
   ```bash
   docker-compose build
   ```

3. **Lancer les services :**
   ```bash
   docker-compose up -d
   ```

4. **Vérifier l'état :**
   ```bash
   docker-compose ps
   ```

## Accès à l'Application

- **Frontend** : http://localhost
- **phpMyAdmin** : http://localhost:8081 (utilisateur: root, mot de passe: root)

## Dépannage

- Vérifiez que Docker est en cours d'exécution.
- Consultez les logs : `docker-compose logs [service]`
- Arrêtez les services : `docker-compose down`

## Technologies Utilisées

- **Backend** : Spring Boot 3.4.0, Java 21, Maven 3.9.9
- **Frontend** : Angular 18, Node.js 20, TypeScript 5.4
- **Base de données** : MySQL 8.0
- **Conteneurisation** : Docker, Docker Compose  
