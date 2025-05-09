# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM prestodb/hdp3.1-hive:unlabelled
LABEL maintainer="Presto community <https://prestodb.io/community.html>"

# COPY CONFIGURATION
COPY ./files /tmp/files

# INSTALL KERBEROS
RUN yum install -y krb5-libs krb5-server krb5-workstation && \
  # COPY CONFIGURATION FILES
  cp -a /tmp/files/etc/* /etc/ && \
  cp -a /tmp/files/var/* /var/ && \
  chown -R root:hadoop /etc/hadoop /etc/hive && \
  chown -R root:root /etc/supervisord* /var/kerberos && \
  # CREATE KERBEROS DATABASE
  /usr/sbin/kdb5_util create -s -P password && \
  \
  # CREATE ANOTHER KERBEROS DATABASE
  /usr/sbin/kdb5_util create -d /var/kerberos/krb5kdc/principal-other -r OTHERLABS.TERADATA.COM -s -P password && \
  \
  # ADD HADOOP PRINCIPALS
  /usr/sbin/kadmin.local -q "addprinc -randkey hdfs/hadoop-master@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "addprinc -randkey mapred/hadoop-master@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "addprinc -randkey yarn/hadoop-master@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "addprinc -randkey HTTP/hadoop-master@LABS.TERADATA.COM" && \
  \
  # CREATE HADOOP KEYTAB FILES
  /usr/sbin/kadmin.local -q "xst -norandkey -k /etc/hadoop/conf/hdfs.keytab hdfs/hadoop-master HTTP/hadoop-master" && \
  /usr/sbin/kadmin.local -q "xst -norandkey -k /etc/hadoop/conf/mapred.keytab mapred/hadoop-master HTTP/hadoop-master" && \
  /usr/sbin/kadmin.local -q "xst -norandkey -k /etc/hadoop/conf/yarn.keytab yarn/hadoop-master HTTP/hadoop-master" && \
  /usr/sbin/kadmin.local -q "xst -norandkey -k /etc/hadoop/conf/HTTP.keytab HTTP/hadoop-master" && \
  \
  chown hdfs:hadoop /etc/hadoop/conf/hdfs.keytab && \
  chown mapred:hadoop /etc/hadoop/conf/mapred.keytab && \
  chown yarn:hadoop /etc/hadoop/conf/yarn.keytab && \
  chown hdfs:hadoop /etc/hadoop/conf/HTTP.keytab && \
  chmod 644 /etc/hadoop/conf/*.keytab && \
  \
  # CREATE HIVE PRINCIPAL AND KEYTAB
  /usr/sbin/kadmin.local -q "addprinc -randkey hive/hadoop-master@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "xst -norandkey -k /etc/hive/conf/hive.keytab hive/hadoop-master" && \
  chown hive:hadoop /etc/hive/conf/hive.keytab && \
  chmod 644 /etc/hive/conf/hive.keytab && \
  \
  # CREATE HIVE PRINCIPAL IN THE OTHER REALM
  /usr/sbin/kadmin.local -r OTHERLABS.TERADATA.COM -d /var/kerberos/krb5kdc/principal-other -q "addprinc -randkey hive/hadoop-master@OTHERLABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -r OTHERLABS.TERADATA.COM -d /var/kerberos/krb5kdc/principal-other -q "xst -norandkey -k /etc/hive/conf/hive-other.keytab hive/hadoop-master" && \
  chown hive:hadoop /etc/hive/conf/hive-other.keytab && \
  chmod 644 /etc/hive/conf/hive-other.keytab && \
  \
  # CREATE HDFS PRINCIPAL IN OTHER REALM
  /usr/sbin/kadmin.local -r OTHERLABS.TERADATA.COM -d /var/kerberos/krb5kdc/principal-other -q "addprinc -randkey hdfs/hadoop-master@OTHERLABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -r OTHERLABS.TERADATA.COM -d /var/kerberos/krb5kdc/principal-other -q "xst -norandkey -k /etc/hadoop/conf/hdfs-other.keytab hdfs/hadoop-master" && \
  chown hdfs:hadoop /etc/hadoop/conf/hdfs-other.keytab && \
  chmod 644 /etc/hadoop/conf/hdfs-other.keytab && \
  \
  # MAKE 'LABS.TERADATA.COM' TRUST 'OTHERLABS.TERADATA.COM'
  /usr/sbin/kadmin.local -q "addprinc -pw 123456 krbtgt/LABS.TERADATA.COM@OTHERLABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -r OTHERLABS.TERADATA.COM -d /var/kerberos/krb5kdc/principal-other -q "addprinc -pw 123456 krbtgt/LABS.TERADATA.COM" && \
  \
  # CREATE PRESTO PRINCIPAL AND KEYTAB
  /usr/sbin/kadmin.local -q "addprinc -randkey presto-server/presto-master.docker.cluster@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "addprinc -randkey presto-server/presto-worker.docker.cluster@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "addprinc -randkey presto-server/presto-worker-1.docker.cluster@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "addprinc -randkey presto-server/presto-worker-2.docker.cluster@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "addprinc -randkey HTTP/presto-master.docker.cluster@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "addprinc -randkey presto-client/presto-master.docker.cluster@LABS.TERADATA.COM" && \
  /usr/sbin/kadmin.local -q "addprinc -randkey hive/presto-master.docker.cluster@LABS.TERADATA.COM" && \
  mkdir -p /etc/presto/conf && \
  /usr/sbin/kadmin.local -q "xst -norandkey -k /etc/presto/conf/presto-server.keytab presto-server/presto-master.docker.cluster presto-server/presto-worker.docker.cluster presto-server/presto-worker-1.docker.cluster presto-server/presto-worker-2.docker.cluster" && \
  /usr/sbin/kadmin.local -q "xst -norandkey -k /etc/presto/conf/presto-server-HTTP.keytab HTTP/presto-master.docker.cluster" && \
  /usr/sbin/kadmin.local -q "xst -norandkey -k /etc/presto/conf/presto-client.keytab presto-client/presto-master.docker.cluster" && \
  /usr/sbin/kadmin.local -q "xst -norandkey -k /etc/presto/conf/hive-presto-master.keytab hive/presto-master.docker.cluster" && \
  chmod 644 /etc/presto/conf/*.keytab && \
  \
  # CREATE SSL KEYSTORE
  keytool -genkeypair \
      -alias presto \
      -keyalg RSA \
      -keystore /etc/presto/conf/keystore.jks \
      -keypass password \
      -storepass password \
      -dname "CN=presto-master, OU=, O=, L=, S=, C=" \
      -validity 100000 && \
  chmod 644 /etc/presto/conf/keystore.jks && \
  # CLEANUP
  yum -q clean all && rm -rf /var/cache/yum && rm -rf /tmp/* /var/tmp/*

# EXPOSE KERBEROS PORTS
EXPOSE	88 89 749

CMD supervisord -c /etc/supervisord.conf
