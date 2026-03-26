FROM tomcat:11.0.13-jre25-temurin-noble

LABEL org.opencontainers.image.source="https://github.com/lsu-ub-uu/cora-docker-tomcat" \
      org.opencontainers.image.description="Production-ready Tomcat base image for Cora" \
      org.opencontainers.image.licenses="GPL-3.0-or-later"

# Remove default webapps that are not needed and pose a security risk
RUN rm -rf /usr/local/tomcat/webapps.dist/ROOT \
           /usr/local/tomcat/webapps.dist/docs \
           /usr/local/tomcat/webapps.dist/examples \
           /usr/local/tomcat/webapps.dist/host-manager \
           /usr/local/tomcat/webapps.dist/manager \
    && rm -rf /usr/local/tomcat/webapps/ROOT \
              /usr/local/tomcat/webapps/docs \
              /usr/local/tomcat/webapps/examples \
              /usr/local/tomcat/webapps/host-manager \
              /usr/local/tomcat/webapps/manager

COPY server.xml /usr/local/tomcat/conf/
COPY log4j2.properties /usr/local/tomcat/lib/

# Create a dedicated non-root user for running Tomcat
RUN groupadd -r tomcat \
    && useradd -r -g tomcat -d /usr/local/tomcat -s /usr/sbin/nologin tomcat \
    && mkdir -p /usr/local/tomcat/logs /usr/local/tomcat/work /usr/local/tomcat/temp \
    && chown -R tomcat:tomcat /usr/local/tomcat/logs \
                              /usr/local/tomcat/work \
                              /usr/local/tomcat/temp \
                              /usr/local/tomcat/webapps \
                              /usr/local/tomcat/conf

# Container-aware JVM settings for production
ENV CATALINA_OPTS="-XX:+UseContainerSupport \
    -XX:MaxRAMPercentage=75.0 \
    -XX:InitialRAMPercentage=50.0 \
    -XX:+UseG1GC \
    -XX:+ExitOnOutOfMemoryError \
    -Djava.security.egd=file:/dev/urandom"

EXPOSE 8080

USER tomcat

HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
    CMD bash -c '>/dev/tcp/localhost/8080' || exit 1