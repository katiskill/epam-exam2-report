FROM openjdk:8-jdk

MAINTAINER Anton Chernov

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000
ARG JENKINS_AGENT_HOME=/home/${user}

ENV JENKINS_AGENT_HOME ${JENKINS_AGENT_HOME}

RUN groupadd -g ${gid} ${group} && \
	useradd -d "${JENKINS_AGENT_HOME}" -u "${uid}" -g "${gid}" -m -s /bin/bash "${user}" && \
	echo "jenkins:jenkins" | chpasswd

RUN sed -i 's|http://.*debian.org/|http://mirror.yandex.ru/|' /etc/apt/sources.list && \
	apt-get update && \
	apt-get install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common && \
	curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - && \
	add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"

RUN echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main" >> /etc/apt/sources.list && \
	apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367

RUN apt-get update && \
	apt-get install -y docker-ce openssh-server ansible python3 python3-pip python3-venv && \
	rm -rf /var/lib/apt/lists/* && \
	usermod -aG docker ${user} && service docker start

RUN mkdir ${JENKINS_AGENT_HOME}/ansible && mkdir ${JENKINS_AGENT_HOME}/.ssh && \
	chown -R ${user}:${user} ${JENKINS_AGENT_HOME}/

RUN su - jenkins -c 'ssh-keygen -f '${JENKINS_AGENT_HOME}'/.ssh/jenkins-slave -N ""' && \
	cat ${JENKINS_AGENT_HOME}/.ssh/jenkins-slave 

RUN cat ${JENKINS_AGENT_HOME}/.ssh/jenkins-slave.pub >> ${JENKINS_AGENT_HOME}/.ssh/authorized_keys && \
	service ssh restart

EXPOSE 22