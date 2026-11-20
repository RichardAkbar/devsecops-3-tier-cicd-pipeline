FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Apply available Alpine security updates and create a non-root user
RUN apk upgrade --no-cache \
    && addgroup -S appgroup \
    && adduser -S appuser -G appgroup

COPY --chown=appuser:appgroup target/richard-petstore-0.0.1-SNAPSHOT.jar app.jar

# Run application as non-root user
USER appuser

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar", "--server.port=8081"]
