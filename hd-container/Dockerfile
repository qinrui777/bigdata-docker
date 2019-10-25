FROM debian:latest

# add jdk
RUN mkdir /usr/local/java
ADD jdk-8u221-linux-x64.tar.gz /usr/local/java/
# make a symbol link
RUN ln -s /usr/local/java/jdk1.8.0_221 /usr/local/java/jdk

# set environment variables
ENV JAVA_HOME /usr/local/java/jdk
ENV JRE_HOME ${JAVA_HOME}/jre
ENV CLASSPATH .:${JAVA_HOME}/lib:${JRE_HOME}/lib
ENV PATH ${JAVA_HOME}/bin:$PATH

# set hadoop env,install openssh,generate key files
RUN apt-get update -q && \
    mkdir -p /opt/data/hadoop/hdfs/name && \
    mkdir -p /opt/data/hadoop/hdfs/data && \
    mkdir -p /opt/data/hadoop/tmp && \
    mkdir -p /var/log/hadoop && \
    mkdir -p /var/log/yarn && \
    mkdir -p /var/log/hive  && \
    mkdir -p /run/sshd && \
    apt-get install -y  openssh-server openssh-client && \
    yes y | ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N '' >/dev/null && \
    yes y | ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N '' >/dev/null &&  \
    yes y | ssh-keygen -q -t dsa -f /etc/ssh/ssh_host_ed25519_key  -N '' >/dev/null

# login localhost without password
RUN ssh-keygen -f /root/.ssh/id_rsa -N '' && \
    touch /root/.ssh/authorized_keys &&  \
    cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys && \
    echo "root:123456" | chpasswd && \
    apt-get clean && rm -rf /var/lib/apt

# open the port 22
EXPOSE 22
# when start a container it will be executed
CMD ["/usr/sbin/sshd","-D"]