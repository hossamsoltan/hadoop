
# Hadoop Cluster Setup
This project focuses on setting up Hadoop Cluster for learning environment.

## Setup
 ![alt text](image/Namenode-and-Datanode.png)

-- Namenode

-- 3 Datanode

## Prerequisites
Java => openjdk version "1.8.0_412"

Redhat => 9.2

Hadoop => 3.4.0

## Installation
This installation will be for learning so I will disable the firewall on all nodes
### 1- firewall
#### All Nodes
~~~bash 
systemctl stop firewalld.service #Stop firewall Service
~~~ 
~~~bash 
systemctl disable firewalld.service # Disable the Firewall service,
~~~
### 2- Hostname, DNS, and Hadoop User 
You can use differnt IPs based on your network
#### NamdeNode
Change the hostname for master node
~~~bash 
hostnamectl hostname master
~~~
Edit the hosts file to add DNS record for all nodes
~~~bash 
vi /etc/hosts
~~~
~~~dns 
  192.168.83.240  master  # Add this records to /etc/hosts
  192.168.83.241  node1   # Add this records to /etc/hosts
  192.168.83.242  node2   # Add this records to /etc/hosts
  192.168.83.243  node3   # Add this records to /etc/hosts
~~~

~~~bash 
useradd hadoop            # Create Hadoop USer
~~~
~~~bash
passwd hadoop             # Set Password
  :password
~~~
~~~bash
echo "ALL ALL=(ALL) ALL" > /etc/sudoers.d/hadoop  # Add the Hadoop User to superuser
~~~
~~~bash
su - hadoop   # Switch to hadoop user
~~~
#### DataNodes
Change the hostname for worker node which the n is the node number
~~~bash 
hostnamectl hostname nodeN
~~~
Edit the hosts file to add DNS record for all nodes
~~~bash 
vi /etc/hosts
~~~
~~~dns 
  192.168.83.240  master  # Add this records to /etc/hosts
  192.168.83.241  node1   # Add this records to /etc/hosts
  192.168.83.242  node2   # Add this records to /etc/hosts
  192.168.83.243  node3   # Add this records to /etc/hosts
~~~

~~~bash 
useradd hadoop            # Create Hadoop USer
~~~
~~~bash
passwd hadoop             # Set Password
  :password
~~~
~~~bash
echo "ALL ALL=(ALL) ALL" > /etc/sudoers.d/hadoop  # Add the Hadoop User to superuser
~~~
~~~bash
su - hadoop
~~~
### 3- SSH configuration
After Making sure the SSH serive is installed and enabled to all nodes please do below step on the master node
#### Master Node
~~~bash
ssh-keygen -t rsa
~~~
~~~bash
ssh-copy-id hdoop@master # Will ask to enter the password
ssh-copy-id hdoop@192.168.83.240
~~~
~~~bash
ssh-copy-id hdoop@node1 # Will ask to enter the password
ssh-copy-id hdoop@192.168.83.241
~~~
~~~bash
ssh-copy-id hdoop@node2 # Will ask to enter the password
ssh-copy-id hdoop@192.168.83.242
~~~
~~~bash
ssh-copy-id hdoop@node3 # Will ask to enter the password
ssh-copy-id hdoop@192.168.83.243
~~~
### 4- Java installation and Environment Variable
Do below Step for all nodes to install the Java
#### All Nodes
~~~bash  
 sudo dnf install -y java-1.8.0-openjdk-devel
~~~
~~~bash
sudo vi ~/.bashrc   # add below Env Variables  =fo Java to .bashrc

    # java
    export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
    export PATH=$JAVA_HOME/bin:$PATH
~~~
~~~bash
source ~/.bashrc
~~~

### 5- Download, Install Hadoop and configure Env Variable
#### All Nodes
~~~bash
sudo dnf install -y wget # Downloader
~~~
DownLoad Hadoop Package 3.4.0, if the URL is not working search on the internet and download it
~~~bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
~~~
~~~bash
tar -xzvf hadoop-3.4.0.tar.gz 
~~~
~~~bash
sudo mv hadoop-3.4.0 /usr/local/hadoop
~~~
~~~bash
sudo chown -R hadoop:hadoop /usr/local/hadoop
~~~
~~~bash
sudo vi ~/.bashrc   # add below Env Variables  for Java to .bashrc

    # hadoop
    export HADOOP_HOME=/home/jane/hadoop
    export HADOOP_INSTALL=$HADOOP_HOME
    export HADOOP_MAPRED_HOME=$HADOOP_HOME
    export HADOOP_COMMON_HOME=$HADOOP_HOME
    export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
    export HADOOP_HDFS_HOME=$HADOOP_HOME
    export YARN_HOME=$HADOOP_HOME
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
    export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
    export HADOOP_OPTS=-Djava.net.preferIOv4stack=true
    export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
~~~
~~~bash
source ~/.bashrc
~~~

### 6- Configure Hadoop
#### Namenode => Master

##### Configure Hadoop Daemon Variable

###### hadoop-env.sh
~~~bash
vi /usr/local/hadoop/etc/hadoop/hadoop-env.sh

    export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
    #allow a specific user to run the 
    export HDFS_NAMENODE_USER="hadoop"
    export HDFS_DATANODE_USER="hadoop"
    export YARN_RESOURCEMANAGER_USER="hadoop"
    export HDFS_SECONDARYNAMENODE_USER="hadoop"
~~~
##### Configuring the Hadoop Cluster
###### workers
~~~bash
vi /usr/local/hadoop/etc/hadoop/etc/hadoop/workers

    node1
    node2
    node3
~~~
##### Configuring the Hadoop Daemons
###### core-site.xml
~~~bash
vi /usr/local/hadoop/etc/hadoop/core-site.xml

    <configuration>
      <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
      </property>
    </configuration>
~~~
###### hdfs-site.xml
~~~bash
vi /usr/local/hadoop/etc/hadoop/hdfs-site.xml

    <configuration>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>~/hdfs/namenode</value>
        </property>
        <property>
            <name>dfs.replication</name>
            <value>3</value>
        </property>
    </configuration>

~~~
###### mapred-site.xml
~~~bash
vi /usr/local/hadoop/etc/hadoop/mapred-site.xml

    <configuration>
      <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>
      <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
      </property>
      <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
      </property>
      <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
      </property>
    </configuration>
~~~
###### yarn-site.xml
~~~bash
vi /usr/local/hadoop/etc/hadoop/yarn-site.xml

    <configuration>
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>master</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services.mapreduce.class</name>
            <value>org.hadoop.mapred.shufflehandler</value>
        </property>
        <property>
            <name>yarn.resourcemanager.webapp.address</name>
            <value>master:8088</value>
        </property>
        <property>
            <name>yarn.resourcemanager.resource-tracker.address</name>
            <value>master:8031</value>
        </property>
    </configuration>

~~~


#### DataNode => worker

##### Configure Hadoop Daemon Variable

###### hadoop-env.sh
~~~bash
vi /usr/local/hadoop/etc/hadoop/hadoop-env.sh

    export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
    #allow a specific user to run the 
    export HDFS_NAMENODE_USER="hadoop"
    export HDFS_DATANODE_USER="hadoop"
    export YARN_RESOURCEMANAGER_USER="hadoop"
    export HDFS_SECONDARYNAMENODE_USER="hadoop"
~~~
##### Configuring the Hadoop Cluster
###### workers
~~~bash
vi /usr/local/hadoop/etc/hadoop/etc/hadoop/workers

    node1
    node2
    node3
~~~
##### Configuring the Hadoop Daemons
###### core-site.xml
~~~bash
vi /usr/local/hadoop/etc/hadoop/core-site.xml

    <configuration>
      <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
      </property>
    </configuration>
~~~
###### hdfs-site.xml
~~~bash
vi /usr/local/hadoop/etc/hadoop/hdfs-site.xml

  <configuration>
      <property>
          <name>dfs.datanode.data.dir</name>
          <value>~/hdfs/datanode</value>
      </property>
  </configuration>
~~~
###### mapred-site.xml
~~~bash
vi /usr/local/hadoop/etc/hadoop/mapred-site.xml


    <configuration>
      <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>
    </configuration>
~~~
###### yarn-site.xml
~~~bash
vi /usr/local/hadoop/etc/hadoop/yarn-site.xml


    <configuration>
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>master</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services.mapreduce.class</name>
            <value>org.hadoop.mapred.shufflehandler</value>
        </property>
        <property>
            <name>yarn.resourcemanager.webapp.address</name>
            <value>master:8088</value>
        </property>
        <property>
            <name>yarn.resourcemanager.resource-tracker.address</name>
            <value>master:8031</value>
        </property>
    </configuration>
~~~

### 7- Hadoop Startup
#### Namenode => master
~~~bash
hdfs namenode -format
~~~
~~~bash
start-dfs.sh
~~~
~~~bash
start-yarn.sh
~~~

### 8- Verify Hadoop Installation
You can access the Hadoop web interface by navigating to http://<master_node>:9870/ in a web browser.

master node
~~~bash
hdfs dfsadmin -report
~~~

### 9- Example and Testing
####Step 1: Create a directory on HDFS
~~~bash
hdfs dfs -mkdir /input
~~~
#### Step 2: Copy a text file into the directory
~~~bash
echo "Hello World, this is a test file for Hadoop." > test.txt
hdfs dfs -put test.txt /input/
~~~
#### Step 3: Run the WordCount example
~~~bash
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.4.0.jar wordcount /input /output
~~~
#### Step 4: View the output
~~~bash
hdfs dfs -cat /output/part-r-00000
~~~
You should see output similar to:
~~~kotlin
Hello	1
World,	1
this	1
is	1
a	1
test	1
file	1
for	1
Hadoop.	1
~~~
