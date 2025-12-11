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
