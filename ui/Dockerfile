# ---------- Build Stage ----------
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build

# Install Java, tar, gzip (needed for mvnw)
RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    tar \
    gzip \
    && dnf clean all

WORKDIR /app

# Copy Maven wrapper and project files
COPY . .

# Pre-fetch dependencies (cache layer)
RUN ./mvnw dependency:go-offline -B

# Build the jar
RUN ./mvnw clean package -DskipTests

# ---------- Runtime Stage ----------
FROM public.ecr.aws/amazoncorretto/amazoncorretto:21-alpine

WORKDIR /app

# Copy only the built jar from build stage
COPY --from=build /app/target/*.jar app.jar

# JVM optimization flags (adjust as needed)
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/urandom"

# Expose port (adjust if your app runs on another)
EXPOSE 8080

# Run the app
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
