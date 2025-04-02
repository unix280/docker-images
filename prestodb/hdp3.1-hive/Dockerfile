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

FROM prestodb/centos7-oj8:unlabelled

# Change default timezone
RUN ln -snf "/usr/share/zoneinfo/Asia/Kathmandu" /etc/localtime && echo "Asia/Kathmandu" > /etc/timezone

# Install HDP repo
RUN set -xeu; \
    echo '[HDP]' > /etc/yum.repos.d/hdp.repo && \
    echo 'name=HDP' >> /etc/yum.repos.d/hdp.repo && \
    echo 'baseurl=https://hdpweb.o.onslip.net/p/HDP/3.x/3.1.0.0/centos7' >> /etc/yum.repos.d/hdp.repo && \
    echo 'enabled=1' >> /etc/yum.repos.d/hdp.repo && \
    echo 'gpgcheck=0' >> /etc/yum.repos.d/hdp.repo && \
    echo OK

# Install Hadoop, Hive
RUN yum install -y \
    hadoop-hdfs-namenode \
    hadoop-hdfs-secondarynamenode \
    hadoop-hdfs-datanode \
    hadoop-yarn-resourcemanager \
    hadoop-yarn-nodemanager \
    hive \
    hive-metastore \
    hive-server2 \
    tez \
    hadooplzo \
    hadooplzo-native \
    lzo \
    lzo-devel \
    lzop \
    mariadb-server \
    mysql-connector-java \
  && yum -y clean all && rm -rf /tmp/* /var/tmp/* \
  && ln -s /usr/share/java/mysql-connector-java.jar /usr/hdp/current/hive-metastore/lib/mysql-connector-java.jar

# Delete original configuration
RUN rm -r /etc/hadoop/conf/* \
  && rm -r /etc/hive/conf/*

# Setup sock proxy
RUN yum install -y openssh openssh-clients openssh-server && yum -y clean all
RUN ssh-keygen -t rsa -b 4096 -C "automation@prestodb.io" -N "" -f /root/.ssh/id_rsa && \
    ssh-keygen -t rsa -b 4096 -N "" -f /etc/ssh/ssh_host_rsa_key && \
    cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys
RUN passwd --unlock root
RUN chmod 0700 /root && chmod 0700 /root/.ssh

# Copy configuration files
COPY ./files /

# Run setup script
RUN /root/setup.sh \
  && rm -rf /tmp/* /var/tmp/*

# Provide convenience bash history
RUN set -xeu; \
    echo "supervisorctl restart all" >> ~root/.bash_history; \
    for user in root hive hdfs; do \
        sudo -u "${user}" bash -c ' echo "netstat -ltnp" >> ~/.bash_history '; \
        sudo -u "${user}" bash -c ' echo "beeline -n hive" >> ~/.bash_history '; \
        sudo -u "${user}" bash -c ' echo "hdfs dfs -ls -R /user/hive/warehouse" >> ~/.bash_history '; \
        sudo -u "${user}" bash -c ' mkdir -p ~/.beeline '; \
        sudo -u "${user}" bash -c ' echo "SELECT current_user();" >> ~/.beeline/history '; \
    done

# HDFS ports
EXPOSE 1004 1006 8020 9866 9867 9870 9864 50470

# YARN ports
EXPOSE 8030 8031 8032 8033 8040 8041 8042 8088 10020 19888

# HIVE ports
EXPOSE 9083 10000

# SOCKS port
EXPOSE 1180

CMD supervisord -c /etc/supervisord.conf

