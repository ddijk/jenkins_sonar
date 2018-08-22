FROM jenkins/jenkins:lts

ENV JENKINS_HOME /var/jenkins_home

COPY --chown=jenkins:jenkins id_rsa     $JENKINS_HOME/.ssh/
COPY --chown=jenkins:jenkins id_rsa.pub $JENKINS_HOME/.ssh/

