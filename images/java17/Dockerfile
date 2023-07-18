# Build Stage
FROM public.ecr.aws/amazoncorretto/amazoncorretto:17 as base

# We tell DNF not to install Recommends and Suggests packages, keeping our
# installed set of packages as minimal as possible.
FROM base as build
RUN yum install -y maven

VOLUME /tmp
WORKDIR /

COPY pom.xml .

COPY .mvn .mvn
COPY mvnw .


RUN ./mvnw dependency:go-offline dependency:copy-dependencies -B -q

COPY ./src ./src

RUN ./mvnw -DskipTests package -q

# Package Stage
FROM base

WORKDIR /function

COPY ./ATTRIBUTION.md ./LICENSES.md

RUN ls -lrta

COPY --from=build /target/dependency/*.jar ./
COPY --from=build /target/*.jar ./

RUN ls -lrta

EXPOSE 8080

ENTRYPOINT [ "/usr/bin/java", "-cp", "./*", "com.amazonaws.services.lambda.runtime.api.client.AWSLambda" ]

CMD [ "com.amazon.sample.orders.OrdersApplication" ]
