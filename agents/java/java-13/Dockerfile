FROM arm32v7/debian:buster-20190910

RUN apt-get update
RUN apt-get install -y software-properties-common
RUN apt-get install -y gnupg dirmngr curl tar

RUN gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv C2518248EEA14886
RUN gpg --export --armor C2518248EEA14886| apt-key add -

RUN echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee /etc/apt/sources.list.d/webupd8team-java.list
RUN echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list

RUN apt-get update

# install jdk-8 so that the jenkins agent runs
RUN apt-get install -y default-jdk
RUN mkdir -p /usr/local/java/bin/
RUN ln -s /usr/lib/jvm/default-java/bin/ /usr/local/java/bin/

# install jenkins
ARG VERSION=3.35
ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000

RUN groupadd -g ${gid} ${group}
RUN useradd -c "Jenkins user" -d /home/${user} -u ${uid} -g ${gid} -m ${user}
LABEL Description="This is a base image, which provides the Jenkins agent executable (agent.jar)" Vendor="Jenkins project" Version="${VERSION}"

ARG AGENT_WORKDIR=/home/${user}/agent

RUN echo 'deb http://deb.debian.org/debian stretch-backports main' > /etc/apt/sources.list.d/stretch-backports.list
RUN apt-get update && apt-get install -y -t stretch-backports git-lfs
RUN curl --create-dirs --insecure -fsSLo /usr/share/jenkins/agent.jar https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/${VERSION}/remoting-${VERSION}.jar \
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
ARG USER_HOME_DIR=/home/${user}
ARG SHA=d941423d115cd021514bfd06c453658b1b3e39e6240969caf4315ab7119a77299713f14b620fb2571a264f8dff2473d8af3cb47b05acf0036fc2553199a5c1ee
ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

USER root

RUN mkdir -p /usr/share/maven
RUN mkdir -p /usr/share/maven/ref
RUN curl --insecure -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz
RUN echo "${SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - 
RUN tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 
RUN rm -f /tmp/apache-maven.tar.gz 
RUN ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

###############################################################################################
# Additional build dependencies
###############################################################################################

# install java-13
RUN curl --insecure -fsSL -o /tmp/bellsoft-jdk13-linux-arm32-vfp-hflt.deb https://download.bell-sw.com/java/13/bellsoft-jdk13-linux-arm32-vfp-hflt.deb
RUN dpkg -i --force-depends /tmp/bellsoft-jdk13-linux-arm32-vfp-hflt.deb
RUN apt-get install -f -y
RUN rm -f /tmp/bellsoft-jdk13-linux-arm32-vfp-hflt.deb
#RUN export JAVA_HOME=/usr/share/java13

# JNLP Snippet

COPY jenkins-agent /usr/local/bin/jenkins-agent
RUN chmod +x /usr/local/bin/jenkins-agent &&\
    ln -s /usr/local/bin/jenkins-agent /usr/local/bin/jenkins-slave

USER ${user}

ENTRYPOINT ["jenkins-agent"]
