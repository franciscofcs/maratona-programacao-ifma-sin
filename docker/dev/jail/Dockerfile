#========================================================================
# Copyright 2020-2021 Joao Vitor Alves Fazolo and Rodrigo Laiola Guimaraes
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <https://www.gnu.org/licenses/>.
# 
# This program is released under license GNU GPL v3+ license.
#
#========================================================================

# Build on base image
FROM boca-base

LABEL authors="Joao Vitor Alves Fazolo, Rodrigo Laiola Guimaraes"
ENV CREATED_AT 2020-06-26
ENV UPDATED_AT 2021-11-09

# Basic setup to establish connection with database server
RUN apt-get -y update && \
    apt-get -y dist-upgrade && \
    apt-get -y install \
        git \
        postgresql-client && \
    apt-get clean

COPY LICENSE /
COPY docker/dev/jail/init.sh /
RUN chmod 777 /init.sh

RUN cd /var/www
# Clone BOCA repository
# Although Jail does not depend on the web application, it requires
# src/db.php and includes a bunch of other php files in this folder.
RUN git clone https://github.com/cassiopc/boca.git

COPY LICENSE /var/www/boca/src/private
# Copy modified source code files
COPY src/private/conf.php /var/www/boca/src/private
COPY src/private/createdb.php /var/www/boca/src/private

RUN cd /var/www/boca && install tools/boca-fixssh /usr/sbin/ && \
    install tools/cron-boca-fixssh /etc/cron.d/ && \
    chmod 700 /usr/sbin/boca-fixssh && \
    cp tools/000-boca.conf /etc/apache2/sites-enabled/000-boca.conf && \
    chown -R www-data.www-data /var/www/boca && \
    chmod -R go-rwx /var/www/boca/src/private

RUN cd /var/www/boca && \
    gcc tools/safeexec.c -o tools/safeexec && \
    install tools/safeexec /usr/bin/safeexec && \
    install tools/boca-createjail /usr/sbin/boca-createjail && \
    install tools/boca-autojudge.sh /usr/sbin/boca-autojudge && \
    install tools/boca-config-dbhost.sh /usr/sbin/boca-config-dbhost && \
    chmod 700 /usr/sbin/boca-config-dbhost && \
    chmod 4555 /usr/bin/safeexec && \
    chmod 700 /usr/sbin/boca-createjail && \
    chmod 700 /usr/sbin/boca-autojudge && \
    chmod 700 /usr/sbin/boca-config-dbhost && \
    chmod 700 /usr/sbin/boca-fixssh && \
    cp tools/boca.conf /etc/ && \
    echo "bdserver=boca-postgres" >> /etc/boca.conf && \
    echo "bdcreated=y" >> /etc/boca.conf

RUN echo "#!/bin/sh\n\
BOCAIP=boca-web" > /etc/bocaip

RUN boca-createjail || true

EXPOSE 80
 
WORKDIR /var/www/boca

ENTRYPOINT ["/init.sh"]