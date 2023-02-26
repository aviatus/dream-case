FROM maven:3.6.3-jdk-11-slim AS build
WORKDIR /project
COPY . /project
RUN mvn clean package -DskipTests

FROM openjdk:11-jre-slim AS runtime
RUN apt-get update
RUN apt-get install tini
RUN addgroup --system app && adduser --ingroup app --system  app

COPY --from=build /project/target/app-0.0.1-SNAPSHOT.jar /app/app.jar
WORKDIR /app

RUN chown -R app:app /app
USER app

ENTRYPOINT ["/usr/bin/tini", "--"]
CMD ["java", "-jar", "app.jar"]

EXPOSE 9001