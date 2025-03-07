# Copyright 2017-2019 CERN for the benefit of the ATLAS collaboration.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# Authors:
# - Thomas Beermann <thomas.beermann@cern.ch>, 2017-2021
# - Vincent Garonne <vgaronne@gmail.com>, 2017-2019
# - Cedric Serfon <cedric.serfon@cern.ch>, 2017
# - Frank Berghaus <frank.berghaus@cern.ch>, 2017-2018
# - Mario Lassnig <mario.lassnig@cern.ch>, 2019
# - Mayank Sharma <mayank.sharma@cern.ch>, 2021

FROM centos:7

RUN yum -y install epel-release && \
    yum -y install https://repo.ius.io/ius-release-el7.rpm && \
    yum -y install gcc httpd mod_ssl mod_auth_kerb python-setuptools python-pip python36u-pip python36-devel python36-mod_wsgi python36-m2crypto gmp-devel krb5-devel git openssl-devel gridsite which libaio memcached nmap-ncat gfal2-all gfal2-util gfal2-python3 fts-rest-cli xrootd-client multitail vim && \
    yum clean all && \
    rm -rf /var/cache/yum

RUN git clone https://github.com/rucio/rucio.git /tmp/rucio

ENV RUCIOHOME=/opt/rucio
RUN mkdir -p $RUCIOHOME && \
    mkdir -p $RUCIOHOME/etc/multi_vo/tst/etc && \
    mkdir -p $RUCIOHOME/etc/multi_vo/ts2/etc
WORKDIR $RUCIOHOME
RUN mkdir -p \
      bin \
      etc \
      lib/rucio \
      tools

RUN python3 -m pip install --upgrade pip && \
    python3 -m pip install --upgrade setuptools && \
    rm -rf /usr/lib/python3.6/site-packages/ipaddress*  && \
    python3 -m pip install -r /tmp/rucio/etc/pip-requires-client && \
    python3 -m pip install -r /tmp/rucio/etc/pip-requires && \
    python3 -m pip install -r /tmp/rucio/etc/pip-requires-test && \
    python3 -m pip install psycopg2-binary && \
    ln -s $RUCIOHOME/lib/rucio /usr/local/lib/python3.6/site-packages/rucio

#additional configuration for fts-rest-cli
RUN pip2 install requests 

COPY .pep8 .pep8
COPY .flake8 .flake8
COPY pylintrc etc/pylintrc
RUN cp -r /tmp/rucio/etc . && \
    cp -r /tmp/rucio/tools . && \
    mkdir /var/log/rucio /var/log/rucio/trace && \
    chmod -R 777 /var/log/rucio

COPY rucio.cfg alembic.ini google-cloud-storage-test.json $RUCIOHOME/etc/
COPY rucio_multi_vo_tst.cfg $RUCIOHOME/etc/multi_vo/tst/etc/rucio.cfg
COPY rucio_multi_vo_ts2.cfg $RUCIOHOME/etc/multi_vo/ts2/etc/rucio.cfg
COPY certs/rucio_ca.pem /etc/grid-security/certificates/5fca1cb1.0

COPY 00-mpm.conf /etc/httpd/conf.modules.d/00-mpm.conf
COPY httpd.conf /etc/httpd/conf/httpd.conf
COPY rucio.conf /etc/httpd/conf.d/rucio.conf
COPY certs/usercert.pem certs/userkey.pem certs/usercertkey.pem $RUCIOHOME/etc/
RUN chmod 0400 $RUCIOHOME/etc/userkey.pem

COPY certs/hostcert_rucio.pem /etc/grid-security/hostcert.pem
COPY certs/hostcert_rucio.key.pem /etc/grid-security/hostkey.pem
RUN chmod 0400 /etc/grid-security/hostkey.pem && \
    chmod 777 /var/log/rucio/trace && \
    rm -rf $RUCIOHOME/tools && \
    mkdir -p $RUCIOHOME/tools && \
    mkdir -p /etc/httpd && \
    echo "" > /etc/httpd/conf.d/ssl.conf && \
    echo "" > /etc/httpd/conf.d/autoindex.conf && \
    echo "" > /etc/httpd/conf.d/userdir.conf && \
    echo "" > /etc/httpd/conf.d/welcome.conf && \
    echo "" > /etc/httpd/conf.d/zgridsite.conf

COPY logshow restartweb run_daemons create_monit_data /usr/local/bin/
COPY monit-entrypoint.sh /monit-entrypoint.sh
COPY dashboards/* $RUCIOHOME/etc/dashboards/

RUN rm -r /tmp/rucio && \
    chmod +x /usr/local/bin/logshow && \
    chmod +x /usr/local/bin/restartweb && \
    chmod +x /usr/local/bin/run_daemons && \
    chmod +x /usr/local/bin/create_monit_data && \
    chmod +x $RUCIOHOME/etc/dashboards/import_dashboards.sh && \
    chmod +x /monit-entrypoint.sh && \
    ln -fs /usr/bin/python3 /usr/bin/python

EXPOSE 443

ENV PATH $PATH:$RUCIOHOME/bin

CMD ["httpd","-D","FOREGROUND"]
