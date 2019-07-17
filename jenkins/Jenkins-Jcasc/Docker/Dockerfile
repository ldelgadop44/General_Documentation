FROM jenkins/jenkins:lts

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt
ENV CASC_JENKINS_CONFIG=/var/tmp/casc_configs
RUN mkdir -p /var/tmp/casc_configs
COPY jcasc.yaml /var/tmp/casc_configs/.