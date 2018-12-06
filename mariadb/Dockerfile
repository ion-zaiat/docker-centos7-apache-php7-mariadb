FROM centos:7
  
MAINTAINER Giuseppe Gibilisco <g.gibilisco@sentrybay.com>

ENV MARIADB_VERSION=10.3

# User/Group
ENV MY_USER="mysql"
ENV MY_GROUP="mysql"
ENV MY_UID="48"
ENV MY_GID="48"

# Directories/Files
ENV MYSQL_BASE_INCL="/etc/my.cnf.d"
ENV MYSQL_DEF_DAT="/var/lib/mysql"
ENV MYSQL_DEF_LOG="/var/log/mariadb"
ENV MYSQL_DEF_PID="/var/run/mariadb"

ENV MYSQL_LOG_SLOW="${MYSQL_DEF_LOG}/slow.log"
ENV MYSQL_LOG_ERROR="${MYSQL_DEF_LOG}/error.log"
ENV MYSQL_LOG_QUERY="${MYSQL_DEF_LOG}/query.log"

###
### Install
###
RUN groupadd -g ${MY_GID} -r ${MY_GROUP} && \
	adduser ${MY_USER} -u ${MY_UID} -M -s /sbin/nologin -g ${MY_GROUP}

RUN { \
      echo '[mariadb]'; \
      echo 'name = MariaDB'; \
      echo "baseurl = http://yum.mariadb.org/$MARIADB_VERSION/centos7-amd64"; \
      echo 'gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB'; \
      echo 'gpgcheck=1'; \
    } > /etc/yum.repos.d/mariadb.repo

RUN yum -y update && \
    yum -y install epel-release python-setuptools MariaDB-server net-tools hostname && \
    yum -y autoremove && \
    yum clean metadata && \
    yum clean all

##
## Configure
##
RUN \
    rm -rf ${MYSQL_BASE_INCL} && \
	rm -rf ${MYSQL_DEF_DAT} && \
	rm -rf ${MYSQL_DEF_PID} && \
	rm -rf ${MYSQL_DEF_LOG} && \
	\
	mkdir -p ${MYSQL_BASE_INCL} && \
	mkdir -p ${MYSQL_DEF_DAT} && \
	mkdir -p ${MYSQL_DEF_PID} && \
	mkdir -p ${MYSQL_DEF_LOG} && \
	\
	chown -R ${MY_USER}:${MY_GROUP} ${MYSQL_BASE_INCL} && \
	chown -R ${MY_USER}:${MY_GROUP} ${MYSQL_DEF_DAT} && \
	chown -R ${MY_USER}:${MY_GROUP} ${MYSQL_DEF_PID} && \
	chown -R ${MY_USER}:${MY_GROUP} ${MYSQL_DEF_LOG} && \
	\
	chmod 0775 ${MYSQL_BASE_INCL} && \
	chmod 0775 ${MYSQL_DEF_DAT} && \
	chmod 0775 ${MYSQL_DEF_PID} && \
	chmod 0775 ${MYSQL_DEF_LOG}

COPY conf/supervisord.conf /etc/supervisord.conf
COPY scripts/docker-entrypoint.sh /docker-entrypoint.sh

# Setup supervisord
RUN echo "Seting up supervisord..." && \
    chmod 666 /etc/supervisord.conf && \
    easy_install supervisor && \
    echo "Setting up MariDB database initialization scripts..." && \
    mkdir /docker-entrypoint-initdb.d && \
    chmod +x /*.sh

COPY conf/my.cnf /etc/my.cnf
RUN chmod 0444 /etc/my.cnf

##
## Volumes
##
VOLUME /var/lib/mysql
VOLUME /var/log/mariadb
VOLUME /var/run/mariadb

EXPOSE 3306

ENTRYPOINT ["/docker-entrypoint.sh"]
