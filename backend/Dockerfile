FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
RUN apk add --no-cache curl
COPY target/backend-1.0.0.jar app.jar
ENV SERVER_PORT=8089
EXPOSE 8089
ENTRYPOINT ["java", "-jar", "app.jar"]
