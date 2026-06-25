#FROM egovio/alpine-maven-builder-jdk-8:1-master-NA-6036091e AS build
FROM amazoncorretto:17-alpine AS build

ARG WORK_DIR
WORKDIR /app

# Install Maven
RUN apk add --no-cache maven

# copy the project files
COPY ${WORK_DIR}/pom.xml ./pom.xml
COPY build/maven/start.sh ./start.sh

# Layer caches here — only invalidated when pom.xml changes, so on most builds
# this RUN step is skipped entirely and served from Kaniko's layer cache.
RUN mvn -B -f /app/pom.xml dependency:go-offline

# This layer always rebuilds (source changes every commit), but reuses the
# already-cached dependencies from the layer above — no re-download.
COPY ${WORK_DIR}/src ./src
RUN mvn -B -f /app/pom.xml package -o

# Create runtime image
FROM amazoncorretto:17-alpine

WORKDIR /opt/egov

COPY --from=build --chmod=755 /app/target/*.jar /app/start.sh /opt/egov/

CMD ["/opt/egov/start.sh"]