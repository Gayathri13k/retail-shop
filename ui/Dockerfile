# ======================
# Build Stage
# ======================
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build

# Install Maven + Java 21
# Install required tools (add tar here)
RUN dnf --setopt=install_weak_deps=False install -q -y \
    maven \
    java-21-amazon-corretto-headless \
    tar \
    gzip \
    && dnf clean all

WORKDIR /app

# Copy Maven wrapper & config first (better cache)
COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
# Ensure wrapper is executable (fixes "Permission denied")
RUN chmod +x mvnw

# Pre-fetch dependencies
RUN ./mvnw -B -q dependency:go-offline

# Copy source and build
COPY src ./src
RUN ./mvnw -B -q clean package -DskipTests \
    && mv target/*.jar app.jar

# ======================
# Runtime Stage
# ======================
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

# Java runtime + non-root user
RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    && dnf clean all

ENV APPUSER=appuser APPUID=1000 APPGID=1000
RUN useradd --home "/app" --create-home --user-group --uid "$APPUID" "$APPUSER"

WORKDIR /app
USER appuser

# Copy built jar and optional docs
COPY --from=build /app/app.jar ./app.jar
COPY ATTRIBUTION.md LICENSES.md ./ || true

EXPOSE 8080
ENV SPRING_PROFILES_ACTIVE=prod
ENV JAVA_OPTS=""

ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar /app/app.jar"]
