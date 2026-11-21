FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Create a non-root user
RUN addgroup -S appgroup \
    && adduser -S appuser -G appgroup

# Copy Spring Boot executable JAR
COPY --chown=appuser:appgroup target/richard-petstore-0.0.1-SNAPSHOT.jar app.jar

# Run application as non-root user
USER appuser

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar", "--server.port=8081"]
