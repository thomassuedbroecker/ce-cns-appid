##############################
#           BUILD
##############################

FROM "docker.io/adoptopenjdk/maven-openjdk11" as BUILD
COPY src /usr/src/app/src
COPY pom.xml /usr/src/app
WORKDIR /usr/src/app
RUN mvn package

##############################
#           PRODUCTION
##############################
#FROM "docker.io/adoptopenjdk/openjdk11-openj9:ubi-minimal"
FROM "docker.io/adoptopenjdk:openj9"

ENV JAVA_OPTIONS="-Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager"
ENV AB_ENABLED=jmx_exporter

RUN mkdir /opt/shareclasses
# OpenShift permissions:
RUN chmod a+rwx -R /opt/shareclasses
RUN mkdir /opt/app

COPY --from=BUILD --chown=1001 /usr/src/app/target/quarkus-app/lib/ /deployments/lib/
COPY --from=BUILD --chown=1001 /usr/src/app/target/quarkus-app/*.jar /deployments/
COPY --from=BUILD --chown=1001 /usr/src/app/target/quarkus-app/app/ /deployments/app/
COPY --from=BUILD --chown=1001 /usr/src/app/target/quarkus-app/quarkus/ /deployments/quarkus/

# Port 8080 will be used by "Code Engine" for the local OOTB configuration. (before we used 8083)
# useful entry on GitHub https://github.com/knative/serving/issues/7450
# **********
# Local
# EXPOSE 8084 
# **********
# Code Engine
EXPOSE 8080
CMD ["java", "-Xmx128m", "-XX:+IdleTuningGcOnIdle", "-Xtune:virtualized", "-Xscmx128m", "-Xscmaxaot100m", "-Xshareclasses:cacheDir=/opt/shareclasses", "-Dappid.auth-server-url_tenant_A=${APPID_AUTH_SERVER_URL_TENANT_A}", "-Dappid.client_id_tenant_A=${APPID_CLIENT_ID_TENANT_A}", "-jar", "/deployments/quarkus-run.jar"]
