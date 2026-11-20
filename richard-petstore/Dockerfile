FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Apply available Alpine security updates
RUN apk upgrade --no-cache

COPY target/richard-petstore-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar", "--server.port=8081"]
