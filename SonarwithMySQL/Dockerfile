###################
# Pull base image #
###################

FROM ubuntu:14.04

RUN apt-get update

# Forbid daemon to start
RUN echo '#!/bin/sh\nexit 101' > /usr/sbin/policy-rc.d
RUN chmod +x /usr/sbin/policy-rc.d

########################
# Install Oracle Java8 #
########################

RUN \
  apt-get install -y software-properties-common && \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer


# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle
ENV JRE_HOME /usr/lib/jvm/java-8-oracle/jre
# ENV PATH /usr/lib/jvm/java-8-oracle/bin:/usr/lib/jvm/java-8-oracle/jre/bin
#######################################################################################

###############################
# Install MySQL 5.6 DB SERVER #
###############################

RUN apt-get update -y

#installing mysql
RUN export DEBIAN_FRONTEND=noninteractive
RUN echo mysql-server-5.6 mysql-server/root_password password welcome123 | debconf-set-selections
RUN echo mysql-server-5.6 mysql-server/root_password_again password welcome123 | debconf-set-selections

# install the LAMP stack
RUN apt-get -y install mysql-server-5.6
RUN sed -i -e"s/^bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/my.cnf

ADD init-mysql-sonarqube-db.sql /tmp/init-mysql-sonarqube-db.sql

RUN /usr/bin/mysqld_safe & \
    sleep 10s && \
	mysql -u root -pwelcome123 < /tmp/init-mysql-sonarqube-db.sql

VOLUME ["/var/lib/mysql", "/var/log/mysql"]

EXPOSE 3306

# CMD ["mysqld_safe"]

#######################################################################################

################################
# Install SONARQUBE 6.0 SERVER #
################################

RUN apt-get update -y
RUN apt-get install -y unzip wget

ENV sonarQubeVersion 6.0

RUN wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-${sonarQubeVersion}.zip
RUN unzip sonarqube-*.zip
RUN mv /sonarqube-${sonarQubeVersion} /sonarqube
RUN rm sonarqube-*.zip

RUN sed -i 's/^#\?sonar.jdbc.username.*$/sonar.jdbc.username=\${env:SONAR_JDBC_USERNAME}/' /sonarqube/conf/sonar.properties
RUN sed -i 's/^#\?sonar.jdbc.password.*$/sonar.jdbc.password=\${env:SONAR_JDBC_PASSWORD}/' /sonarqube/conf/sonar.properties
RUN sed -i 's/^#\?sonar.jdbc.url.*$/sonar.jdbc.url=\${env:SONAR_JDBC_URL}/'                /sonarqube/conf/sonar.properties
RUN echo 'sonar.web.host=0.0.0.0' >> /sonarqube/conf/sonar.properties

ENV SONAR_JDBC_USERNAME sonar
ENV SONAR_JDBC_PASSWORD sonar
ENV SONAR_JDBC_URL jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance

# CMD ["/sonarqube/bin/linux-x86-64/sonar.sh"]

EXPOSE 9000
#######################################################################################

####################
# Supervisord      #
####################
RUN apt-get install -y supervisor
RUN mkdir -p /var/log/supervisor
ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["/usr/bin/supervisord", "-n"]
#######################################################################################
