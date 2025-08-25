# ---------- Build Stage ----------
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build

RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    tar \
    gzip \
    && dnf clean all

WORKDIR /app

# Copy source
COPY . .

# Fix permission for mvnw
RUN chmod +x mvnw

# Pre-fetch dependencies (cache layer)
RUN ./mvnw dependency:go-offline -B

# Build the jar
RUN ./mvnw clean package -DskipTests

# ---------- Runtime Stage ----------
FROM amazoncorretto:21-alpine

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

ENV JAVA_OPTS="-XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/urandom"

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
