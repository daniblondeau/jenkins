# Jenkins oficial image
FROM jenkins/jenkins:lts

# Skip initial setup wizard & disable CLI
ARG JAVA_OPTS
ENV JAVA_OPTS "-Djenkins.install.runSetupWizard=false ${JAVA_OPTS:-}"

# Variables
ENV JENKINS_HOME /var/jenkins_home

# Copy Plugins list and Install them
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt