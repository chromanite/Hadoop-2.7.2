# Basic set up of Hadoop 2.7.2 using CENTos 7
Hadoop software allows for the distributed processing of large data sets across clusters of computer using simple programming models The Apache Hadoop. It is known for reliability, scalability and distributed computing. 

Create an account for Hadoop 
- #useradd –d /usr/hadoop hadoop 
- #chmod 755 /usr/hadoop 
- #passwd hadoop

Enter the Hadoop account that we had created earlier, fire up terminal, enter as root and disable firewall. Reason behind it, is to enable all port connection so as to ensure that when Hadoop is connecting to all nodes, there is not disruption. 

- $su 
- Password: 
- #systemctl stop firewalld 
- #systemctl disable firewalld 

.As Hadoop runs mostly under Java, we will install the latest Java environment. Remove the pre-installed Java, and we will proceed to install the latest Java Environment 

- #yum remove java-1.6.0-openjdk 
- #yum remove java-1.7.0-openjdk 

Download Java. I am using Java 1.8.0_111 build at the time I’m using. Head over to: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads2133151.html.
We will move the .tar.gz file to /opt/ path. 
- #cd /opt 
- #tar xzf jdk-8u45-linux-x64.tar.gz

Install Java using alternatives.

- #cd /opt/jdk1.8.0_111/ 
- #alternatives -–install /usr/bin/java java /opt/jdk1.8.0_111/bin/java 2 
- #alternatives --config java 

   
Enter the following command to install & set Jar as well as Java C: 
 
- #alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_111/bin/jar 2 
- #alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_111/bin/javac 2 
- #alternatives --set jar /opt/jdk1.8.0_111/bin/jar 
- #alternatives --set javac /opt/jdk1.8.0_111/bin/javac

Checked the installed Java version 
- #java –version 

Java version “1.8.0_111” 

Java(TM) SE Runtime Environment (build 1.8.0_111-b14) 

Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)

Exit root and set up the global environment variables for JAVA_HOME, JRE_HOME and PATH in ~/.bashrc. 
- $export JAVA_HOME=/opt/jdk1.8.0_111 
- $export JRE_HOME=/opt/jdk1.8.0_111/jre 
- $export PATH=$PATH:/opt/jdk1.8.0_111/bin:/opt/jdk1.8.0_111/jre/bin 

Exit the .Bashrc file and source it 
- $source ~/.bashrc   

Setup machine alias in host files. Do a ifconfig command. Note down the NameNodes & DataNodes IP address. Below IP Address are to vary depending on your IP that you received.  Enter as root so that you can edit the hosts file.
- #vi /etc/hosts 

127.0.0.0.1 localhost 

================================

172.16.241.001 DN1 

172.16.241.002 DN2 

172.16.241.224 GANGLIA  

*Ganglia is the monitoring tool’s IP address that we will set up in the project later.

Hadoop control scripts rely on SSH to perform cluster-wide operations. To work seamlessly, SSH needs to be setup for a password-less & passphrase-less login for root/hadoop user from machines in the cluster. Generate private/public key pair. 
- $ ssh-keygen 
- $ ssh-copy-id localhost 
- $ ssh-copy-id datanode 

Download the Hadoop stable release. In time of writing, 2.7.2 version appears to be the stable in comparison with the version 3 Alpha. It will be installing under /usr/hadoop. We will also create a few additional directories, namenode for hadoop to store all namenode information and namesecondary to store the checkpoint images. DO check for latest version of Hadoop & stability. https://www.hadoop.apache.org/releases.html 

- $cd /usr/hadoop 
- $curl –O http://www.us.apache.org/dist/hadoop/common/hadoop-2.7.2/hadoop-2.7.2.ta r.gz 
- $tar zxvf hadoop-2.7.2.tar.gz –C /usr/Hadoop –strip-components 1 
- $mkdir ~/namenode <<DO NOT INCLUDE THIS FILE IN DATANODE!
- $mkdir ~/datanode

Now open up ~/.bash_profile and insert the lines at the end of the page. 
- export HADOOP_HOME=/usr/hadoop 
- export HADOOP_COMMON_HOME=$HADOOP_HOME 
- export HADOOP_HDFS_HOME=$HADOOP_HOME 
- export HADOOP_MAPRED_HOME=$HADOOP_HOME 
- export HADOOP_YARN_HOME=$HADOOP_HOME 
- export HADOOP_OPTS=”-Djava.library.path=$HADOOP_HOME/lib/native” 
- export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native 
- export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin 

Exit, save and source the ~/.bash_profile 
- $source ~/.bash_profile 

Seting up JAVA_HOME for Hadoop environment. Enter this command and add the 3 lines in
- $ vi ~/etc/hadoop/hadoop-env.sh 
---------------------------------------------------------------------------------- 
 
- #the java implementation use 
- export JAVA_HOME=/opt/jdk1.8.0_111   
- export JRE_HOME=/opt/jdk1.8.0_111/jre 
- export PATH=$PATH:/opt/jdk1.8.0_111/bin:/opt/jdk1.8.0_111/jre/bin

Once all is done, we will proceed to set up the xml files configuration. These configurations will determine how well Hadoop will perform various tasks. 
# core-site.xml

- name:fs.defaultFS
- value:hdfs://NameNode:8020/

- name:io.file.buffer.size
- value:131072

# hdfs-site.xml 

name:dfs.replication
value:1

name:dfs.datanode.data.dir
value:file:///usr/hadoop/datanode

name:dfs.namenode.name.dir
value:file:///usr/hadoop/namenode
 
name:dfs.blocksize
value:268435456

name:dfs.namenode.handler.count
value:100

# mapred-site.xml

name:mapreduce.framework.name 
value:yarn 

name:mapreduce.jobhistory.webapp.address
value:0.0.0.0:19888<

name:mapreduce.jobhistory.address
value:0.0.0.0:10020

# yarn-site.xml
name:yarn.resourcemanager.bind-host
value:0.0.0.0

name:yarn.nodemanager.bind-host
value:0.0.0.0

name:yarn.nodemanager.hostname
value:0.0.0.0< 

name:yarn.nodemanager.address
value:0.0.0.0:0

name:yarn.nodemanager.aux-services
value:mapreduce_shuffle

name:yarn.nodemanager.aux-services.mapreduce_shuffle.class
value:org.apache.hadoop.mapred.ShuffleHandler 

name:yarn.resourcemanager.scheduler.class
value:org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacitySc heduler

name:yarn.resourcemanager.scheduler.address
value:namenode:8030
 
name:yarn.resourcemanager.resource-tracker.address
value:namenode:8031 

name:yarn.resourcemanager.address
value:namenode:8032

name:yarn.resourcemanager.webapp.address
value:namenode:8088

After inserting those properties in, we will now edit the slaves file. The slave files will tell the namenode which computer the correct datanode/slave. Insert all the datanode in it

- $vi ~/etc/Hadoop/slaves

datanode1

datanode2

...

We will next edit the master file. The master file is to let namenode and datanode to recognise which is the namenode/master. Enter to namenode and type in these commands. Inside the file, insert ONLY the Namenode and save it.

- $vi ~/etc/Hadoop/masters 

namenode

Once done, format the namenode and start the services
- $hdfs namenode –format 
- $start-dfs.sh;start-yarn.sh 

To check if Hadoop processers are running, simply enter in jps to show the current running processers.
- Namenode
- SecondaryNameNode
- ResourceManager
- JobHistoryServer
- Jps

To test if Hadoop is working, enter this command. The command will shows a variety of programs it can run
hadoop jar ~/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar 

Also, when Hadoop has been successfully set up, open up a web browser and enter the IP address of the namenode with the following port number. Port 50070: Hadoop Administration WebUI Port 19888: Hadoop JobHistory WebUI Port 8088: Hadoop All Application
