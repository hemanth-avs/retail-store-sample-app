# Build Stage
FROM public.ecr.aws/amazoncorretto/amazoncorretto:11 as base

# We tell DNF not to install Recommends and Suggests packages, keeping our
# installed set of packages as minimal as possible.
FROM base as build
RUN yum install -y maven
WORKDIR /src

VOLUME /tmp
WORKDIR /

ADD pom.xml .

COPY .mvn .mvn
COPY mvnw .

RUN ./mvnw dependency:go-offline dependency:copy-dependencies

COPY ./src ./src

RUN ./mvnw -e dependency:tree

RUN ./mvnw -e -DskipTests package

RUN ls -lrta
RUN ls -lrta target/

# Package Stage
FROM base

WORKDIR /function

COPY ./ATTRIBUTION.md ./LICENSES.md

RUN ls -lrta

COPY --from=build /target/dependency/*.jar ./
COPY --from=build /target/*.jar ./

RUN ls -lrta

EXPOSE 8080

ENV JAVA_OPTS='-XX:MaxRAMPercentage=75.0 -Djava.security.egd=file:/dev/urandom'

# Set runtime interface client as default command for the container runtime
ENTRYPOINT [ "/usr/bin/java", "-cp", "./*", "com.amazonaws.services.lambda.runtime.api.client.AWSLambda" ]

CMD [ "com.amazon.sample.orders.OrdersApplication::main" ]

# ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
