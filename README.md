# docker_and_hdfs_setup
installation guide and setup

## install docker from official site for ubuntu
```

sudo apt-get update                                

sudo apt-get install \                             
    apt-transport-https \                          
    ca-certificates \                                 
    curl \
    gnupg \
    lsb-release



curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg



echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null



sudo apt-get update

 sudo apt-get install docker-ce docker-ce-cli containerd.io
```

### checking docker status

```
systemctl status docker

docker version
```

### creating switch/network bridge using docker
```

docker network create network_name --subnet ip_series

docker network create myhadoop_br --subnet 192.168.200.0/24
```
docker network list
```

docker network ls
```

#### creating containers
```

docker  run  -itd --name  namenode  --hostname namenode --network myhadoop_br --ip 192.168.200.100  oraclelinux:8.3  bash 

docker  run  -itd --name  datanode1  --hostname datanode1 --network myhadoop_br oraclelinux:8.3  bash
```

##### Now there are 2 options- either create each datanode manunally and install everything or create one image of datanode and use it for others

creating docker image

```
docker  commit -m  "datanode sample"   datanode1   hadoop:vv3 
```

checking docker image 
```
docker images
```

#### creating datanode container using image(do it after installing java and hadoop it will automatically install everything on image containers)
```

docker  run  -itd --name  datanode3  --hostname datanode3 --network myhadoop_br  hadoop:vv3  

docker  run  -itd --name  datanode4  --hostname datanode4 --network myhadoop_br  hadoop:vv3
```

#### installing jdk 8 in container and check after install

```
dnf  install  java-1.8.0-openjdk.x86_64  java-1.8.0-openjdk-devel.x86_64 -y

java -version

```
##### Adding java path to .bashrc
```
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.el8_4.x86_64
PATH=$PATH:$JAVA_HOME/bin
export PATH
```

#### downloading hadoop
```
wget  https://downloads.apache.org/hadoop/common/stable/hadoop-3.3.1.tar.gz

Downloading on Desktop and then we'll copy it to conatiners in one go!
```
#### copying hadoop file to containers
```
docker cp hadoop-3.3.1.tar.gz namenode:/

docker cp hadoop-3.3.1.tar.gz datanode1:/
```
#### decompressing hadoop tar file in each container after copy
```
dnf install tar -y

tar xvzf  hadoop-3.3.1.tar.gz 
```
##### removing tar unzipped nd moving hadoop to /hadoop3 folder
```
rm hadoop-3.3.1.tar.gz

mv  hadoop-3.3.1 /hadoop3
```
#### setting java path and hadoop path for all conatiners

```
[root@namenode ~]# cat /root/.bashrc 

/# .bashrc

/# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

/# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi


JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.el8_4.x86_64

HADOOP_HOME=/hadoop3

PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

export PATH
```
#### nodes configuration
Name node

```
[root@namenode /]# cd /hadoop3/etc/hadoop/
[root@namenode hadoop]# ls
capacity-scheduler.xml      hadoop-user-functions.sh.example  kms-log4j.properties        ssl-client.xml.example
configuration.xsl           hdfs-rbf-site.xml                 kms-site.xml                ssl-server.xml.example
container-executor.cfg      hdfs-site.xml                     log4j.properties            user_ec_policies.xml.template
core-site.xml               httpfs-env.sh                     mapred-env.cmd              workers
hadoop-env.cmd              httpfs-log4j.properties           mapred-env.sh               yarn-env.cmd
hadoop-env.sh               httpfs-site.xml                   mapred-queues.xml.template  yarn-env.sh
hadoop-metrics2.properties  kms-acls.xml                      mapred-site.xml             yarn-site.xml
hadoop-policy.xml           kms-env.sh                        shellprofile.d              yarnservice-log4j.properties

```
hadoop-env.sh     
```

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.el8_4.x86_64

export HADOOP_HOME=/hadoop3
```

core-site.xml
```
<configuration>
<property>
	<name>fs.default.name</name>
	<value>hdfs://namenode:9000</value>
</property>

</configuration>
```
hdfs-site.xml
```
<configuration>
<property>
	<name>dfs.namenode.name.dir</name>
	<value>/mynndata</value>
	<description>location where namenode will store its metadata </description>
</property>

<property>
	<name>dfs.replication</name>
	<value>3</value>
	<description> number of copy for each block or chunk </description>
</property>
</configuration>
```
Datanodes
hadoop-env.sh will be same as Namenode
core-site.xml will be same as namenode
hdfs-site.xml
```
<configuration>

<property>
	<name>dfs.datanode.data.dir</name>
	<value>/mydndata1</value>
	<description>location where datanode will store its data</description>
</property>
</configuration>
```
#### fomatting namenode

```
[root@namenode hadoop]# hdfs  namenode  -format 
```

