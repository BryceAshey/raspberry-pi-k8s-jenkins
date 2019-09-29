FROM resin/rpi-raspbian:buster-20190121

RUN apt-get update && apt-get upgrade && apt-get install -y apt-utils

RUN echo "deb http://ftp.debian.org/debian buster-backports main" | sudo tee /etc/apt/sources.list.d/backports.list && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 04EE7237B7D453EC && \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 648ACFD622F3D138 && \
    apt-get update && \
    apt-get -t buster-backports install -y openjdk-8-jre-headless=8u222-b10-1 --no-install-recommends && \
    rm -rf /var/lib/apt/lists/*

# Define working directory
WORKDIR /data

# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-armhf

ARG VERSION=3.35
ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000


RUN groupadd -g ${gid} ${group}
RUN useradd -c "Jenkins user" -d /home/${user} -u ${uid} -g ${gid} -m ${user}
LABEL Description="This is a base image, which provides the Jenkins agent executable (agent.jar)" Vendor="Jenkins project" Version="${VERSION}"

ARG AGENT_WORKDIR=/home/${user}/agent

#RUN echo 'deb http://deb.debian.org/debian stretch-backports main' > /etc/apt/sources.list.d/stretch-backports.list
#RUN apt-get update && apt-get install -t stretch-backports git-lfs
RUN curl --create-dirs -fsSLo /usr/share/jenkins/agent.jar https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/${VERSION}/remoting-${VERSION}.jar \
  && chmod 755 /usr/share/jenkins \
  && chmod 644 /usr/share/jenkins/agent.jar \
  && ln -sf /usr/share/jenkins/agent.jar /usr/share/jenkins/slave.jar

USER ${user}
ENV AGENT_WORKDIR=${AGENT_WORKDIR}
RUN mkdir /home/${user}/.jenkins && mkdir -p ${AGENT_WORKDIR}

VOLUME /home/${user}/.jenkins
VOLUME ${AGENT_WORKDIR}
WORKDIR /home/${user}

# installing Maven  

ARG MAVEN_VERSION=3.6.2
ARG USER_HOME_DIR="/root"
ARG SHA=d941423d115cd021514bfd06c453658b1b3e39e6240969caf4315ab7119a77299713f14b620fb2571a264f8dff2473d8af3cb47b05acf0036fc2553199a5c1ee
ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries


USER root

RUN mkdir -p /usr/share/maven
RUN mkdir -p /usr/share/maven/ref
RUN curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz
RUN echo "${SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - 
RUN tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 
RUN rm -f /tmp/apache-maven.tar.gz 
RUN ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

USER ${user}