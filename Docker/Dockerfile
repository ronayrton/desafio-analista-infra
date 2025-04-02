FROM tomcat:9.0

# Definir variáveis
ENV JENKINS_VERSION=2.440.1
ENV JOLOKIA_VERSION=1.7.1

# Definir URLs
ENV JENKINS_WAR_URL="https://get.jenkins.io/war-stable/${JENKINS_VERSION}/jenkins.war"
ENV JOLOKIA_WAR_URL="https://repo1.maven.org/maven2/org/jolokia/jolokia-war/${JOLOKIA_VERSION}/jolokia-war-${JOLOKIA_VERSION}.war"

# Instalar pacotes necessários e baixar arquivos em um único RUN
RUN apt-get update && apt-get install -y curl \
    && curl -fsSL --retry 5 "$JENKINS_WAR_URL" -o /usr/local/tomcat/webapps/jenkins.war \
    && curl -fsSL --retry 5 "$JOLOKIA_WAR_URL" -o /usr/local/tomcat/webapps/jolokia.war \
    && rm -rf /var/lib/apt/lists/*

# Expor porta do Tomcat
EXPOSE 8080

# Comando de inicialização do Tomcat
CMD ["catalina.sh", "run"]
