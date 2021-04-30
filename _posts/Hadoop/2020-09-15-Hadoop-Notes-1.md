---
layout:     post
title:      "Hadoop Learning Notes - 1"
subtitle:   " \"Hadoop - How to deploy multi-nodes Hadoop cluster in Amazon Web Service\""
date:       2020.09.15 15:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 环境搭建
    - Hadoop



---

> *"Keep Learning Hadoop"*

### 0.1. The  integrated processes  to deploy Hadoop in AWS

#### 01.01 Install Hadoop on Amazon AWS EC2 Instances

1. Setup EC2 Instances
2. Setup SSH and Configuration files
3. Configure Putty and WinSCP
4. Install [Hadoop as a Single Node cluster](https://klasserom.azurewebsites.net/CourseStrands/Binder/3380) on each node
5. Test HDFS
6. Configure [Hadoop as a Multiple Node cluster](https://klasserom.azurewebsites.net/CourseStrands/Binder/3385)

Linked Resources

- [Simplify Your Life With an SSH Config File - http://nerderati.com/2011/03/17/simplify-your-life...](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file)

#### 01.02 Setup Amazon AWS EC2 Instances

##### 01.02.01 Build EC2 Instances

###### Choose Instance Type

**Ubuntu Server 16.04 LTS (HVM), SSD Volume Type** - ami-aa2ea6d0Ubuntu Server 16.04 LTS (HVM),EBS General Purpose (SSD) Volume Type. Support available from Canonical (http://www.ubuntu.com/cloud/services).Root device type: ebs Virtualization type: hvm ENA Enabled: Yes

###### Configure Instance Details

Select the following Instance type

| Type                                                         |
| ------------------------------------------------------------ |
| t2.micro (Variable ECUs, 1 vCPUs, 2.5 GHz, Intel Xeon Family, 1 GiB memory, EBS only) |

###### Volumes and Storage

| Size        | 8Gb                 |
| ----------- | ------------------- |
| Volume Type | General Purpose SSD |

###### Add Tags

Add a tag with the following settings:

| Name | NameNode |
| ---- | -------- |
| Key  | Value    |

###### Configure Security Group

| Security group name | hadoop-cluster             |
| ------------------- | -------------------------- |
| Type                | All Traffic                |
| Protocal            | All                        |
| Port Range          | 0 - 65535                  |
| Source              | Anywhere (0.0.0.0/0, ::/0) |

##### 01.02.02 Generate Public and Private Key Pairs

###### Create a new key pair

| Key pair name        | hadoop-clusterkeypair |
| -------------------- | --------------------- |
| Defined by your self | `...`end with `.pem`  |

###### Download Key Pair

- Save the Key Pair file to a location on your local computer
- Name the file `hadoop-clusterkeypair.pem`

##### 01.02.03 Launch Instances

Select the Launch Instances button. This will bring you to a summary of your new Virtual Machine.

Linked Resources

- [Simplify Your Life With an SSH Config File - http://nerderati.com/2011/03/17/simplify-your-life...](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file)

#### 01.03 Setup WinSCP and puTTY

You will need to install WinSCP and Putty to manage the setup and configuration of your Amazon instances. 

Watch this view to assist you with the setup and configuration of WinSCP and Putty. The video covers several parts of the setup process including SSH configuration. Follow the parts related to installation of WinSCP and Putty.

Additional links to WinSCP and Putty are provided below.

Linked Resources

- [puTTY Download - http://www.chiark.greenend.org.uk/~sgtatham/putty/...](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)
- [WinSCP Download - https://winscp.net/eng/download.php](https://winscp.net/eng/download.php)

#### 01.04 Common Environment Variables for Big Data

The Big Data technologies that we use often require Environment Variables to be set that are common for all user of your environment. 

In general these include

- Java - `JAVA_HOME`
- Hadoop - `HADOOP_HOME`, Hive - `HIVE_HOME`, Pig - `PIG_HOME`

We will take advantage of some features of the Ubuntu operating system. In particular `etc/profile.d`. This is a directory that contains shell script files that run during start up and general install features for all users.

We will begin by creating a file named `bigdata.sh` to use used in later installations:

**NOTE**: This command may ask for a password.

```shell
sudo touch /etc/profile.d/bigdata.sh
```

Execute the following to set permission and initialize the file:

```shell
sudo chmod +x /etc/profile.d/bigdata.sh
sudo echo -e '#!/bin/\n# Environment Variables for Big Data tools\n' | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null
```

Reboot your system

```shell
sudo reboot
```

#### 01.04b Map Amazon EC2 Environment Variables

In this section we will add Environment Variables that we can use throughout our clusters to refer to Nodes.

Collect the Public DNS and "Internal" IP addresses from your Amazon EC2 Instances and add the following to your `bigdata.sh` file:

```shell
export NameNodeDNS="ec2-34-227-205-116.compute-1.amazonaws.com"
export DataNode001DNS="ec2-52-90-160-54.compute-1.amazonaws.com"
export DataNode002DNS="ec2-54-175-210-82.compute-1.amazonaws.com"
export DataNode003DNS="ec2-54-86-10-108.compute-1.amazonaws.com"

export NameNodeIP="172.31.31.127"
export DataNode001IP="172.31.30.70"
export DataNode002IP="172.31.24.96"
export DataNode003IP="172.31.88.102"

export IdentityFile="~/.ssh/hadoop-clusterkeypair.pem"
```

Execute the following to add the schema to the `bigdata.sh` file:

```shell
export NameNodeDNS="ec2-18-218-165-62.us-east-2.compute.amazonaws.com"
export DataNode001DNS="ec2-13-58-51-176.us-east-2.compute.amazonaws.com"
export DataNode002DNS="ec2-18-217-252-152.us-east-2.compute.amazonaws.com"
export DataNode003DNS="ec2-18-218-96-134.us-east-2.compute.amazonaws.com"
export NameNodeIP="172.31.31.127"
export DataNode001IP="172.31.30.70"
export DataNode002IP="172.31.24.96"
export DataNode003IP="172.31.88.102"
export IdentityFile="~/.ssh/hadoop-clusterkeypair.pem"

echo "# AmazonEC2 Variables START" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo "export NameNodeDNS=\"${NameNodeDNS}\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo "export DataNode001DNS=\"${DataNode001DNS}\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo "export DataNode002DNS=\"${DataNode002DNS}\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo "export DataNode003DNS=\"${DataNode003DNS}\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo "" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo -e "export NameNodeIP=\"${NameNodeIP}\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo -e "export DataNode001IP=\"${DataNode001IP}\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo -e "export DataNode002IP=\"${DataNode002IP}\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo -e "export DataNode003IP=\"${DataNode003IP}\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo -e "export IdentityFile=\"${IdentityFile}\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo -e "# AmazonEC2 Variables END" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null
```

Reboot your system

```shell
sudo reboot 
```

#### 01.05 Map Nodes in `etc/hosts` File on Amazon EC2 Instances

The `etc/hosts` file manages direct links to servers through the use of IP address and Host name. In this section we will map our Hadoop Cluster Nodes. 

Go to each Node and get the IP address.

Run the following command on each Node and record the return value:

```shell
hostname
```

The result should look similar to this:

```shell
ip-172-31-31-127
```

**NOTE**: Change the DNS entry to match your Public **Amazon EC2** Instance DNS

Execute the following code to create a local variable to store your Public Amazon EC2 Instance DNS:

```shell
publichost=ec2-54-86-10-108.compute-1.amazonaws.com
```

Or, use the Environment Variables we setup earlier:

**NOTE**: Do not execute all of these entries at once. One at a time on the matching instance

```shell
publichost=${NameNodeDNS}
publichost=${DataNode001DNS}
publichost=${DataNode002DNS}
publichost=${DataNode003DNS}
```

Execute the following code to change the local hostname of your Amazon EC2 Instance from the local DNS to the Public DNS:

```shell
sudo hostname ${publichost}

sudo rm -rf /etc/hostname
echo -e "${publichost}" | sudo tee --append /etc/hostname > /dev/null
sudo chown root /etc/hostname
```

##### Configure the /etc/hosts file

Execute the following code to create a local variable to store your Public Amazon EC2 Instance DNS:

```shell
publichost=ec2-54-86-10-108.compute-1.amazonaws.com
```

Once you have completed the IP addresses, execute the code:

```shell
sudo rm -rf /etc/hosts
echo -e "127.0.0.1\tlocalhost" | sudo tee --append /etc/hosts > /dev/null
echo -e "127.0.1.1\t${publichost}" | sudo tee --append /etc/hosts > /dev/null
echo -e "${NameNodeIP}\thadoop-master" | sudo tee --append /etc/hosts > /dev/null
echo -e "${DataNode001IP}\tDataNode001" | sudo tee --append /etc/hosts > /dev/null
echo -e "${DataNode002IP}\tDataNode002" | sudo tee --append /etc/hosts > /dev/null
echo -e "${DataNode003IP}\tDataNode003" | sudo tee --append /etc/hosts > /dev/null
echo -e "\n# The following lines are desirable for IPv6 capable hosts" | sudo tee --append /etc/hosts > /dev/null
echo -e "::1 ip6-localhost ip6-loopback" | sudo tee --append /etc/hosts > /dev/null
echo -e "fe00::0 ip6-localnet" | sudo tee --append /etc/hosts > /dev/null
echo -e "ff00::0 ip6-mcastprefix" | sudo tee --append /etc/hosts > /dev/null
echo -e "ff02::1 ip6-allnodes" | sudo tee --append /etc/hosts > /dev/null
echo -e "ff02::2 ip6-allrouters" | sudo tee --append /etc/hosts > /dev/null
echo -e "ff02::3 ip6-allhosts" | sudo tee --append /etc/hosts > /dev/null
sudo chown root /etc/hosts
```

Reboot your Amazon EC2 Instances to initialize your new `etc/hosts` file.

```shell
sudo reboot
```

#### 01.06 Connect and Install Passwordless SSH 

We are using `~/.ssh/.config`, PEM and PPK files to connect and install Passwordless SSH Connect and Install Passwordless SSH using PEM and PPK files. And then, we convert Public Key to Private Key using puTTY and finally, we build an `SSH .config` file.

For some variables, we explain them as:

- **HostName** - entry is the Public DNS IPV4 value for your Amazon EC2 instance.
- **User** - the default user name for your Amazon EC2 Ubuntu instance is `ubuntu`

- **IdentityFile** - the Key Pair (.pem) file you created while setting up your instances

Open a text editor and enter the following:

Modify the HostName for each entry in your .config file to match your Amazon EC2 Instances

```shell
Host 0.0.0.0
  HostName ec2-34-227-205-116.compute-1.amazonaws.com
  User ubuntu
  IdentityFile ~/.ssh/hadoop-clusterkeypair.pem
Host hadoop-master
  HostName ec2-34-227-205-116.compute-1.amazonaws.com
  User ubuntu
  IdentityFile ~/.ssh/hadoop-clusterkeypair.pem
Host DataNode001
  HostName ec2-34-207-108-211.compute-1.amazonaws.com
  User ubuntu
  IdentityFile ~/.ssh/hadoop-clusterkeypair.pem
Host DataNode002
  HostName ec2-52-91-241-213.compute-1.amazonaws.com
  User ubuntu
  IdentityFile ~/.ssh/hadoop-clusterkeypair.pem
Host DataNode003
  HostName ec2-52-55-203-83.compute-1.amazonaws.com
  User ubuntu
  IdentityFile ~/.ssh/hadoop-clusterkeypair.pem
```

Save the file as "`config`" with no file extension.

 the `config` file and your `hadoop-clusterkeypair.pem` file to your `~/.ssh` folder on each of your Nodes using [WinSCP](https://winscp.net/eng/download.php).

If you choose, execute this code with your modified `HostName` entries:

```shell
sudo rm -rf ~/.ssh/config

echo -e "Host 0.0.0.0" | tee --append ~/.ssh/config > /dev/null
echo -e "  HostName ${NameNodeDNS}" | tee --append ~/.ssh/config > /dev/null
echo -e "  User ubuntu" | tee --append ~/.ssh/config > /dev/null
echo -e "  IdentityFile ${IdentityFile}" | tee --append ~/.ssh/config > /dev/null
echo -e "Host hadoop-master" | tee --append ~/.ssh/config > /dev/null
echo -e "  HostName ${NameNodeDNS}" | tee --append ~/.ssh/config > /dev/null
echo -e "  User ubuntu" | tee --append ~/.ssh/config > /dev/null
echo -e "  IdentityFile ${IdentityFile}" | tee --append ~/.ssh/config > /dev/null
echo -e "Host DataNode001" | tee --append ~/.ssh/config > /dev/null
echo -e "  HostName ${DataNode001DNS}" | tee --append ~/.ssh/config > /dev/null
echo -e "  User ubuntu" | tee --append ~/.ssh/config > /dev/null
echo -e "  IdentityFile ${IdentityFile}" | tee --append ~/.ssh/config > /dev/null
echo -e "Host DataNode002" | tee --append ~/.ssh/config > /dev/null
echo -e "  HostName ${DataNode002DNS}" | tee --append ~/.ssh/config > /dev/null
echo -e "  User ubuntu" | tee --append ~/.ssh/config > /dev/null
echo -e "  IdentityFile ${IdentityFile}" | tee --append ~/.ssh/config > /dev/null
echo -e "Host DataNode003" | tee --append ~/.ssh/config > /dev/null
echo -e "  HostName ${DataNode003DNS}" | tee --append ~/.ssh/config > /dev/null
echo -e "  User ubuntu" | tee --append ~/.ssh/config > /dev/null
echo -e "  IdentityFile ${IdentityFile}" | tee --append ~/.ssh/config > /dev/null

sudo chmod 0400 ~/.ssh/config
sudo chmod 0400 ${IdentityFile}
```

Once you have copied the files you'll need to set a permission level of `0400` to both of these files on each of the Nodes:

```shell
sudo chmod 0400 ~/.ssh/config
sudo chmod 0400 ~/.ssh/hadoop-clusterkeypair.pem
```

Attempt to ssh into each Node from NameNode

> NOTE: *Remember to exit from each Node after executing this command*

```shell
ssh -o StrictHostKeyChecking=no hadoop-master
```

```shell
ssh -o StrictHostKeyChecking=no DataNode001
```

```shell
ssh -o StrictHostKeyChecking=no DataNode002
```

```shell
ssh -o StrictHostKeyChecking=no DataNode003
```

The result of this command will look something like the following:

```shell
The authenticity of host 'ec2-34-227-205-116.compute-1.amazonaws.com (172.31.31.127)' can't be established.
ECDSA key fingerprint is SHA256:BvMyZDBxj5zZ8rldAr4ws1WaC/i6kc+xdUnJqcm1SVY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ec2-34-227-205-116.compute-1.amazonaws.com,172.31.31.127' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-1044-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

21 packages can be updated.
0 updates are security updates.


Last login: Mon Jan  8 00:53:27 2018 from 99.255.142.148
```

Linked Resources

- [Connecting to Your Linux Instance from Windows Using PuTTY - http://docs.aws.amazon.com/AWSEC2/latest/UserGuide...](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html?icmpid=docs_ec2_console#putty-prereqs)
- [Hadoop on Amazon AWS Part 2: Setup puTTY and WinSCP to connect to Amazon Hadoop Cluster - YouTube - https://youtu.be/lRQqR0Fm1oE?list=PLWsYJ2ygHmWhOvt...](https://youtu.be/lRQqR0Fm1oE?list=PLWsYJ2ygHmWhOvtHIPxGJDtki9SJlINCw)
- [Hadoop on Amazon AWS Part 3: Setup Passwordless SSH - YouTube - https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvt...](https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvtHIPxGJDtki9SJlINCw)
- [Hadoop Part 3 - Install Passwordless SSH on VMWare - YouTube - https://youtu.be/Ynt4FNSoS-c](https://youtu.be/Ynt4FNSoS-c)
- [puTTY Download - http://www.chiark.greenend.org.uk/~sgtatham/putty/...](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)
- [Simplify Your Life With an SSH Config File - http://nerderati.com/2011/03/17/simplify-your-life...](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file)
- [Spinning Up a Free Hadoop Cluster on Amazon AWS: Step by Step - https://blog.insightdatascience.com/spinning-up-a-...](https://blog.insightdatascience.com/spinning-up-a-free-hadoop-cluster-step-by-step-c406d56bae42#.v9rl8wrle)
- [ssh_config(5): OpenSSH SSH client config files - Linux man page - http://linux.die.net/man/5/ssh_config](http://linux.die.net/man/5/ssh_config)

#### 01.07 Configure Passwordless SSH on Amazon EC2 Instances

Review the [Passwordless SSH Single Node installation](http://klasserom.azurewebsites.net/Lessons/Binder/1939#CourseStrand_3379) instructions before going further. 

The following commands will  the Public Key to each of the Data Nodes.

Execute the following commands from the Master Node to generate a Public Key:

```shell
sudo rm -rf ~/.ssh/id_rsa*
sudo rm -rf ~/.ssh/known_hosts

ssh-keygen -f ~/.ssh/id_rsa -t rsa -P ""
```

Set the security permissions on the `~/.ssh/id_rsa.pub` file

```shell
sudo chmod 0600 ~/.ssh/id_rsa.pub
```

 the key file to each of the NameNodes:

```shell
sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

 the keys for the Node the fast way:

> NOTE:  this code to a text file and modify the **IP addresses** to match your cluster!

```shell
hosts=0.0.0.0,127.0.0.1,127.0.1.1,hadoop-master,DataNode001,DataNode002,DataNode003
ssh-keyscan -H ${hosts} >> ~/.ssh/known_hosts
```

 security keys to each of the Nodes in your cluster:

```shell
sudo cat ~/.ssh/id_rsa.pub | ssh -o StrictHostKeyChecking=no DataNode001 'cat >> ~/.ssh/authorized_keys'
sudo cat ~/.ssh/id_rsa.pub | ssh -o StrictHostKeyChecking=no DataNode002 'cat >> ~/.ssh/authorized_keys'
sudo cat ~/.ssh/id_rsa.pub | ssh -o StrictHostKeyChecking=no DataNode003 'cat >> ~/.ssh/authorized_keys'
```

Attempt to SSH into each of your Nodes:

> NOTE: *Remember to **exit** from each Node after executing these commands*

```shell
ssh -o StrictHostKeyChecking=no localhost
exit
```

```shell
ssh -o StrictHostKeyChecking=no hadoop-master
exit
```

```shell
ssh -o StrictHostKeyChecking=no DataNode001
exit
```

```shell
ssh -o StrictHostKeyChecking=no DataNode002
exit
```

```shell
ssh -o StrictHostKeyChecking=no DataNode003
exit
```

If something went wrong during your SSH setup, run the following code to reset SSH and start over:

```shell
filename=~/.ssh/authorized_keys
line=$(head -1 ${filename})
echo $line

sudo rm -rf ${filename}
echo -e "${line}" | tee --append ${filename} > /dev/null
sudo chmod 0600 ${filename}

sudo rm -rf ~/.ssh/id_rsa*
sudo rm -rf ~/.ssh/known_hosts
ssh-keygen  -f ~/.ssh/id_rsa -t rsa -P ""
sudo chmod 0600 ~/.ssh/id_rsa.pub
sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
sudo chmod 0600 ~/.ssh/authorized_keys
sudo chmod 0400 ~/.ssh/config
sudo chmod 0400 ~/.ssh/hadoop-clusterkeypair.pem

hosts=0.0.0.0,127.0.0.1,127.0.1.1,hadoop-master,DataNode001,DataNode002,DataNode003
ssh-keyscan -H ${hosts} >> ~/.ssh/known_hosts

sudo service ssh restart 
```

Linked Resources

- [Hadoop Cluster Setup - http://hadoop.apache.org/docs/current/hadoop-proje...](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)
- [Hadoop on Amazon AWS Part 3: Setup Passwordless SSH - YouTube - https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvt...](https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvtHIPxGJDtki9SJlINCw)
- [Public-key cryptography - https://en.wikipedia.org/wiki/Public-key_cryptogra...](https://en.wikipedia.org/wiki/Public-key_cryptography)
- [Spinning Up a Free Hadoop Cluster on Amazon AWS: Step by Step - https://blog.insightdatascience.com/spinning-up-a-...](https://blog.insightdatascience.com/spinning-up-a-free-hadoop-cluster-step-by-step-c406d56bae42#.v9rl8wrle)

#### 01.08 Install Java Developers Kit (JDK)

##### Update packages on the server

```shell
sudo apt-get -y update
```

##### Install JDK

```shell
sudo apt-get -y install default-jdk
```

##### Notice

If we install JDK using the method above, after we deploy Hadoop and start it, it will show some WARNING message such as `Illegal access...`, it means that your JDK version is too high, you have to delete them clearly and install JDK-8 or other old version.

About how to delete JDK clearly, you should clear them using the codes shown below:

```shell
$ sudo apt-get update 
$ sudo apt-cachesearch java | awk '{print($1)}' | grep -E -e '^(ia32-)?(sun|oracle)-java' -e'^openjdk-' -e '^icedtea' -e '^(default|gcj)-j(re|dk)' -e '^gcj-(.*)-j(re|dk)'-e 'java-common' | xargs sudo apt-get -y remove
$ sudo apt-get -yautoremove // delete JDK

$ dpkg -l | grep ^rc | awk '{print($2)}' |xargs
$ sudo apt-get -y purge // delete configuration information

$ bash -c 'ls -d /home/*/.java' | xargs
$ sudo rm -rf // delete JAVA configurations and caches

$ rm -rf /usr/lib/jvm/* // delete files under JVM
```

##### Confirm the Java install location and  the location

This command will install Java into the /usr/lib/jvm/java-x-openjdk-amd64. A link folder is also created that references the Java install folder. You will use the link folder location to set up the Environment Variables. 

Navigate to the folder location using:

```shell
cd /usr/lib/jvm/
```

Here you should see a folder that looks something like this:

```shell
/usr/lib/jvm/default-java
```

##### Add Environment Variables to /etc/profile.d/bigdata.sh

Open the file /etc/profile.d/bigdata.sh

```shell
sudo gedit /etc/profile.d/bigdata.sh
```

Add these lines to the end of the file:

```shell
export JAVA_HOME=/usr/lib/jvm/default-java
PATH=$PATH:$JAVA_HOME/bin
```

Or, push with code:

```shell
echo "# JAVA Variables START" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo "export JAVA_HOME=/usr/lib/jvm/default-java" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo "PATH=\$PATH:\$JAVA_HOME/bin" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

echo "# JAVA Variables END" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null
```

Confirm that your Java variables were added, open the /etc/profile.d/bigdata.sh file:

```shell
sudo gedit /etc/profile.d/bigdata.sh
```

##### Reboot to instantiate to environment variables

```shell
sudo reboot
```

##### Confirm the Java Version and Environment Variables

##### Java Version Check

```shell
java -version
```

Displays something similar to this:

```shell
openjdk version "1.8.0_131"

OpenJDK Runtime Environment (build 1.8.0_131-8u131-b11-2ubuntu1.16.04.3-b11)

OpenJDK 64-Bit Server VM (build 25.131-b11, mixed mode)
```

##### Environment Variable Check

```shell
echo $JAVA_HOME 
```

Displays something similar to this:

```shell
/usr/lib/jvm/default-java 
```

#### 01.09 Disable IPv6

Some versions of Hadoop require the IPv6 is disabled. Currently 2.9.0 does not require this.

Open `hadoop-env.sh` for editing we'll add the following lines:

```shell
sudo gedit $HADOOP_CONF_DIR/hadoop-env.sh
```

For more information on IPv6 see this link https://en.wikipedia.org/wiki/IPv6

Add the following line at the bottom of the file to disable `IPv6`:

```shell
HADOOP_OPTS=-Djava.net.preferIPv4Stack=true
```

Save the file and close.

##### Edit /etc/sysctl.conf to add addition configuration for disabling IPv6

```shell
sudo chown ubuntu /etc/sysctl.conf
```

```shell
sudo gedit /etc/sysctl.conf
```

Add these lines to the end of the file:

```shell
# IPv6 configuration
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

```shell
sudo chown root /etc/sysctl.conf
```

Or, execute the following script:

```shell
echo "# IPv6 configuration" | sudo tee --append /etc/sysctl.conf > /dev/null
echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee --append /etc/sysctl.conf > /dev/null
echo "net.ipv6.conf.default.disable_ipv6 = 1" | sudo tee --append /etc/sysctl.conf > /dev/null
echo "net.ipv6.conf.lo.disable_ipv6 = 1" | sudo tee --append /etc/sysctl.conf > /dev/null
```

##### Confirm that IPv6 is disabled:

Reload configuration for `sysctl.conf`

```shell
sudo sysctl -p
```

Check IPv6 is disabled typing

```shell
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
```

Response:

0 – means that IPv6 is enabled.
1 – means that IPv6 is disable. *This is what we expect.*

#### 01.10 Install and Configure Hadoop on Single Node Cluster

Consider using this link to find the appropriate Hadoop download http://apache.forsale.plus/hadoop/common/

> The current version may be higher, this tutorial was tested on 2.9.0.

```shell
wget http://apache.forsale.plus/hadoop/common/hadoop-2.9.0/hadoop-2.9.0.tar.gz -P ~/Downloads/Hadoop
```

##### Uncompress the Hadoop tar file into the `/usr/local` folder

```shell
sudo tar -zxvf ~/Downloads/Hadoop/hadoop-*.tar.gz -C /usr/local
```

##### Move all Hadoop related file from `/usr/local` to `/usr/local/hadoop`

```shell
sudo mv /usr/local/hadoop-* /usr/local/hadoop
```

##### Set Environment Variables

Use this command to edit the `/etc/profile.d/bigdata.sh` file:

```shell
sudo gedit /etc/profile.d/bigdata.sh
```

Paste this code into your `/etc/profile.d/bigdata.sh` file:

```shell
export HADOOP_HOME="/usr/local/hadoop"

export HADOOP_CONF_DIR="${HADOOP_HOME}/etc/hadoop"

export HADOOP_DATA_HOME="${HOME}/hadoop_data/hdfs"

PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

Or, run this script from the command line:

```shell
sudo echo -e "# HADOOP Variables START" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

sudo echo -e "export HADOOP_HOME='/usr/local/hadoop'" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

sudo echo -e "export HADOOP_CONF_DIR=\"\${HADOOP_HOME}/etc/hadoop\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

sudo echo -e "export HADOOP_DATA_HOME=\"\${HOME}/hadoop_data/hdfs\"" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

sudo echo -e "PATH=\$PATH:\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null

sudo echo -e "# HADOOP Variables END" | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null
```

##### Instantiate Environment Variables

The following command will instantiate the new variables available immediately. You can use this method to instantiate variable in any of the modified files in this tutorial:

```shell
source /etc/profile.d/bigdata.sh
```

##### Test the Variables

```shell
echo $HADOOP_HOME
```

This command should result in the following output:

```shell
/usr/local/hadoop
```

Reboot your system for changes to take effect

```shell
sudo reboot
```

**NOTE**: For **Amazon EC2**, Reboot your Instances

##### Login as your Hadoop User

Once you have rebooted your virtual machine, login with the new `hduser` account or the User account you have set up for Hadoop usage.

NOTE: On **Amazon EC2** this user is `ubuntu`.

##### Test Environment Variables

```shell
echo $HADOOP_HOME
```

Should produce a result that looks something like this

```shell
/usr/local/hadoop
```

##### Test Hadoop Version

```
hadoop version 
```

This should produce a result similar to this:

```shell
Hadoop 2.9.0
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r 756ebc8394e473ac25feac05fa493f6d612e6c50
Compiled by arsuresh on 2017-11-13T23:15Z
Compiled with protoc 2.5.0
From source with checksum 0a76a9a32a5257331741f8d5932f183
This command was run using /usr/local/hadoop/share/hadoop/common/hadoop-common-2.9.0.jar
```

##### Build the Hadoop data directories

```shell
mkdir -p $HADOOP_DATA_HOME/datanode

mkdir -p $HADOOP_DATA_HOME/namenode

mkdir -p $HADOOP_DATA_HOME/tmp
```

##### Hadoop Configuration Files

Modify permissions on the `HADOOP_CONF_DIR` to allow you to edit the configuration files:

**NOTE**: You do not need to execute this on Amazon EC2 Instances

```shell
sudo chown root -R $HADOOP_HOME
```

Then set permissions to Read / Write

NOTE: You do not need to execute this on Amazon EC2 Instances

```shell
sudo chmod 777 -R $HADOOP_HOME
```

##### Add JAVA_HOME Variable to $HADOOP_CONF_DIR/hadoop-env.sh

Use `gedit` or the text editor in WinSCP to edit the file `$HADOOP_CONF_DIR/hadoop-env.sh`

NOTE: If you are running Amazon EC2 Instance, open this file using the WinSCP text editor

```shell
sudo gedit $HADOOP_CONF_DIR/hadoop-env.sh
```

Locate the area that indicates the current `JAVA_HOME` variable. It should look something like this:

```shell
export JAVA_HOME=${JAVA_HOME}
```

Change the variable to look like this statement, this location should be the same as your `JAVA_HOME` environment variable:

```shell
export JAVA_HOME=/usr/lib/jvm/default-java
```

> NOTE: You can find your `JAVA_HOME` variable location using this command:

```shell
echo $JAVA_HOME
```

##### Get your Hostname (Computer Name or DNS)

Your Hostname identifies your computer to the network. It may be acceptable at times to use the keyword localhost to refer to your computer name but more often in big systems it is preferable to use the DNS. You will use this DNS to refer to your computer in many of the Hadoop configuration files.

To find your local DNS use the following command:

```shell
echo $(hostname)
```

Should display something like:

```shell
ubuntu
```

##### Modify $HADOOP_CONF_DIR/core-site.xml

Use this command to open the `core-site.xml` file

```shell
sudo gedit $HADOOP_CONF_DIR/core-site.xml
```

Add the following lines to the configuration section of the `core-site.xml` file. 

```markdown
<configuration>
  <!--Custom Properties-->

  <property>

    <name>thisnamenode</name>

    <value>localhost</value>

    <description>localhost may be replaced with a DNS that points to the NameNode.</description>

  </property>

  <property>

    <name>homefolder</name>

    <value>/home/${user.name}</value>

  </property>

  <!--Hadoop Properties-->

  <property>

    <name>hadoop.tmp.dir</name>

    <value>${homefolder}/hadoop_data/hdfs/tmp</value>

    <description>A base for other temporary directories.</description>

  </property>

  <property>

    <name>fs.defaultFS</name>

    <value>hdfs://${thisnamenode}:9000</value>

    <description>localhost may be replaced with a DNS that points to the NameNode.</description>

  </property>

  <property> 

    <name>dfs.permissions</name> 

    <value>false</value> 

  </property>

</configuration>
```

##### Modify $HADOOP_CONF_DIR/yarn-site.xml

Use this command to open the `yarn-site.xml` file

```shell
sudo gedit $HADOOP_CONF_DIR/yarn-site.xml
```

Add the following lines to the configuration section of the `yarn-site.xml` file. 

```markdown
<configuration>

  <property>

    <name>yarn.nodemanager.aux-services</name>

    <value>mapreduce_shuffle</value>

  </property> 

  <property>

    <name>mapred.job.tracker</name>

    <value>${thisnamenode}:9001</value>

  </property>

  <!--<property>

    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>

    <value>org.apache.hadoop.mapred.ShuffleHandler</value>

  </property>-->

</configuration>
```

##### Modify $HADOOP_CONF_DIR/mapred-site.xml

 the `mapred-site.xml` template and rename the new file `mapred-site.xml` edit configuration element

```shell
sudo cp $HADOOP_CONF_DIR/mapred-site.xml.template $HADOOP_CONF_DIR/mapred-site.xml
```

You'll need to set the permissions of `mapred-site.xml` to `0644`

```shell
sudo chmod 644 $HADOOP_CONF_DIR/mapred-site.xml
```

> NOTE: On Amazon EC2 execute the following command to return ownership of the file:
>
> Note the username, `ubuntu`. On a Virtual Machine you may be using `root`.

```shell
sudo chown ubuntu $HADOOP_CONF_DIR/mapred-site.xml
```

Use this command to open the `mapred-site.xml` file

```shell
sudo gedit $HADOOP_CONF_DIR/mapred-site.xml
```

Edit the configuration element in `mapred-site.xml`.

```markdown
<configuration>

  <property>

    <name>mapreduce.jobtracker.address</name>

    <value>local</value>

  </property>

  <property>

    <name>mapreduce.framework.name</name>

    <value>yarn</value>

  </property>

</configuration>
```

##### Modify $HADOOP_CONF_DIR/hdfs-site.xml

Use this command to open the `hdfs-site.xml` file

```shell
sudo gedit $HADOOP_CONF_DIR/hdfs-site.xml
```

Edit the configuration element in `hdfs-site.xml`.

```markdown
<configuration>

  <property>

    <name>dfs.replication</name>

    <value>1</value>

    <description>Default block replication.

The actual number of replications can be specified when the file is created.

The default is used if replication is not specified in create time.

    </description>

  </property>

  <property>

    <name>dfs.namenode.name.dir</name>

    <value>file://${homefolder}/hadoop_data/hdfs/namenode</value>

  </property>

  <property>

    <name>dfs.datanode.data.dir</name>

    <value>file://${homefolder}/hadoop_data/hdfs/datanode</value>

  </property>

  <property>

    <name>dfs.permissions.enabled</name>

    <value>false</value>

    <description>If "true", enable permission checking in HDFS. If "false", permission checking is turned off, but all other behavior is unchanged. Switching from one parameter value to the other does not change the mode, owner or group of files or directories.</description>

  </property>

</configuration>
```

##### Return Ownership of the $HADOOP_HOME folder to root

We'll return the ownership back to root on all files in `$HADOOP_HOME`. We've make changes so let's just be sure:

NOTE: You do not need to execute this on Amazon EC2 Instances

```shell
sudo chown root -R $HADOOP_HOME
```

##### Allow all users to have Read / Write access to your $HADOOP_HOME folder

Note: This is not a suggested Production practice.

**NOTE**: You do not need to execute this on Amazon EC2 Instances

```shell
sudo chmod 777 -R $HADOOP_HOME
```

##### Format the HDFS

```shell
hdfs namenode -format
```

##### Reboot

```shell
sudo reboot
```

Linked Resources

- [Configuration (Apache Hadoop Main 2.9.0 API) - https://hadoop.apache.org/docs/r2.9.0/api/org/apac...](https://hadoop.apache.org/docs/r2.9.0/api/org/apache/hadoop/conf/Configuration.html)

#### 01.11 Install Hadoop on all Slave Nodes

Follow these instructions the setup [Hadoop on Single Node Cluster](http://klasserom.azurewebsites.net/CourseStrands/Binder/3380). Ensure that you have a fully functional Single-Node Hadoop cluster running on each Node.

Linked Resources

- [Hadoop Cluster Setup - http://hadoop.apache.org/docs/current/hadoop-proje...](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)
- [Hadoop on Amazon AWS Part 3: Setup Passwordless SSH - YouTube - https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvt...](https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvtHIPxGJDtki9SJlINCw)
- [Public-key cryptography - https://en.wikipedia.org/wiki/Public-key_cryptogra...](https://en.wikipedia.org/wiki/Public-key_cryptography)
- [Spinning Up a Free Hadoop Cluster on Amazon AWS: Step by Step - https://blog.insightdatascience.com/spinning-up-a-...](https://blog.insightdatascience.com/spinning-up-a-free-hadoop-cluster-step-by-step-c406d56bae42#.v9rl8wrle)

#### 01.12 Configure Hadoop on Multiple Node Cluster

Confirm that the following configuration settings are available on your NameNode. Once you have completed these steps, run the `scp` command at the bottom of this section to  the configuration files to each Node.

##### Modify $HADOOP_CONF_DIR/core-site.xml

Use this command to open the `core-site.xml` file

```shell
sudo gedit $HADOOP_CONF_DIR/core-site.xml
```

Add the following lines to the configuration section of the `core-site.xml` file. 

```markdown
  <property>

    <name>thisnamenode</name>

    <value>hadoop-master</value>

    <description>NameNode is the hostname specified in the config file and etc/hosts file. It may be replaced with a DNS that points to your NameNode.</description>

  </property>
```

##### Modify $HADOOP_CONF_DIR/yarn-site.xml

Use this command to open the `yarn-site.xml` file

```shell
sudo gedit $HADOOP_CONF_DIR/yarn-site.xml
```

Add the following lines to the configuration section of the `yarn-site.xml` file. 

```markdown
  <property>

    <name>mapred.job.tracker</name>

    <value>${thisnamenode}:9001</value>

  </property>
```

##### Modify $HADOOP_CONF_DIR/hdfs-site.xml

Use this command to open the `hdfs-site.xml` file

```shell
sudo gedit $HADOOP_CONF_DIR/hdfs-site.xml
```

Edit the configuration element in `hdfs-site.xml`.

```markdown
  <property>

    <name>dfs.replication</name>

    <value>3</value>

    <description>Default block replication.

The actual number of replications can be specified when the file is created.

The default is used if replication is not specified in create time.

    </description>

  </property>
```

##### Return Ownership of the `$HADOOP_HOME` folder to root

We'll return the ownership back to root on all files in `$HADOOP_HOME`. We've make changes so let's just be sure:

```shell
sudo chown root -R $HADOOP_HOME
```

> NOTE: This is not needed on Amazon EC2 instances

##### Allow all users to have Read / Write access to your `$HADOOP_HOME` folder

**Note**: This is not a suggested Production practice.

```shell
sudo chmod 777 -R $HADOOP_HOME
```

#####  configuration files to all Nodes

```shell
cd $HADOOP_CONF_DIR
scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml DataNode001:$HADOOP_CONF_DIR
scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml DataNode002:$HADOOP_CONF_DIR
scp hadoop-env.sh core-site.xml hdfs-site.xml mapred-site.xml DataNode003:$HADOOP_CONF_DIR
```

Linked Resources

- [Hadoop Cluster Setup - http://hadoop.apache.org/docs/current/hadoop-proje...](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)
- [Spinning Up a Free Hadoop Cluster on Amazon AWS: Step by Step - https://blog.insightdatascience.com/spinning-up-a-...](https://blog.insightdatascience.com/spinning-up-a-free-hadoop-cluster-step-by-step-c406d56bae42#.v9rl8wrle)
- [Tutorialspoint - Hadoop - Multi Node Cluster - https://www.tutorialspoint.com/hadoop/hadoop_multi...](https://www.tutorialspoint.com/hadoop/hadoop_multi_node_cluster.htm)

#### 01.13 Configure `.masters` File

Create the `masters` file if it does not exist. *Only on your NameNode and SecondaryNameNode, not Slaves*.

```shell
sudo touch $HADOOP_CONF_DIR/masters
```

Add the DNS of your Master Node

```
hadoop-master
DataNode001
```

> NOTE: On Amazon EC2 this is `NameNode`. These entries will ensure that hadoop-master (NameNode) and DataNode001 (SecondaryNameNode) are master nodes.

Execute this code to add the entry for you:

```shell
sudo rm -rf $HADOOP_CONF_DIR/masters

echo -e "hadoop-master" | sudo tee --append $HADOOP_CONF_DIR/masters > /dev/null
echo -e "DataNode001" | sudo tee --append $HADOOP_CONF_DIR/masters > /dev/null
```

Set ownership and permissions to the root (`ubuntu` on Amazon EC2 or `root` on VMWare) owner:

```shell
sudo chown ubuntu $HADOOP_CONF_DIR/masters
sudo chmod 0644 $HADOOP_CONF_DIR/masters
```

 the `masters` file to your SecondaryNameNode

**NOTE**: We are using `DataNode001` as our SecondaryNameNode

```
scp $HADOOP_CONF_DIR/masters DataNode001:$HADOOP_CONF_DIR
```

Linked Resources

- [Hadoop Cluster Setup - http://hadoop.apache.org/docs/current/hadoop-proje...](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)
- [Hadoop on Amazon AWS Part 3: Setup Passwordless SSH - YouTube - https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvt...](https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvtHIPxGJDtki9SJlINCw)
- [Public-key cryptography - https://en.wikipedia.org/wiki/Public-key_cryptogra...](https://en.wikipedia.org/wiki/Public-key_cryptography)
- [Spinning Up a Free Hadoop Cluster on Amazon AWS: Step by Step - https://blog.insightdatascience.com/spinning-up-a-...](https://blog.insightdatascience.com/spinning-up-a-free-hadoop-cluster-step-by-step-c406d56bae42#.v9rl8wrle)

#### 01.14 Configure `.slaves` File

Edit the `slaves `file and  it to each DataNode in your cluster.

Open the slaves file for editing:

```shell
sudo gedit $HADOOP_CONF_DIR/slaves
```

Add the entries for the Data Nodes:

```
DataNode001
DataNode002
DataNode003
```

Or, execute this following script:

> NOTE: Confirm the entries after executing this script

```shell
sudo rm -rf $HADOOP_CONF_DIR/slaves
echo -e "DataNode001" | sudo tee --append $HADOOP_CONF_DIR/slaves > /dev/null
echo -e "DataNode002" | sudo tee --append $HADOOP_CONF_DIR/slaves > /dev/null
echo -e "DataNode003" | sudo tee --append $HADOOP_CONF_DIR/slaves > /dev/null
```

Set ownership and permissions

```shell
sudo chown ubuntu $HADOOP_CONF_DIR/slaves
sudo chmod 0644 $HADOOP_CONF_DIR/slaves
```

 the slaves file to each Node in your cluster

```shell
scp $HADOOP_CONF_DIR/slaves DataNode001:$HADOOP_CONF_DIR
scp $HADOOP_CONF_DIR/slaves DataNode002:$HADOOP_CONF_DIR
scp $HADOOP_CONF_DIR/slaves DataNode003:$HADOOP_CONF_DIR
```

Linked Resources

- [Hadoop Cluster Setup - http://hadoop.apache.org/docs/current/hadoop-proje...](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html)
- [Hadoop on Amazon AWS Part 3: Setup Passwordless SSH - YouTube - https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvt...](https://youtu.be/u7fIf_R-gaM?list=PLWsYJ2ygHmWhOvtHIPxGJDtki9SJlINCw)
- [Public-key cryptography - https://en.wikipedia.org/wiki/Public-key_cryptogra...](https://en.wikipedia.org/wiki/Public-key_cryptography)
- [Spinning Up a Free Hadoop Cluster on Amazon AWS: Step by Step - https://blog.insightdatascience.com/spinning-up-a-...](https://blog.insightdatascience.com/spinning-up-a-free-hadoop-cluster-step-by-step-c406d56bae42#.v9rl8wrle)

#### 01.15 Format Your Multi-Node Hadoop Cluster

Recall that you have already formatted your Hadoop Cluster as a Single Node Cluster. This means that the HDFS on each Node will not match with the new Cluster. You will need to remove the hadoop_data directory from each Node:

```shell
sudo rm -rf $HADOOP_DATA_HOME
```

It is a good idea to reset your log directory too:

```shell
sudo rm -rf $HADOOP_HOME/logs
```

On your NameNode execute the following code to reformat your Hadoop Cluster:

```shell
hdfs namenode -format
```

Confirm from the output that there are no errors. Review logs to ensure your cluster is formatted as expected `/usr/local/hadoop/logs`.

#### 01.16 Reset all settings in your cluster

If something has gone terribly wrong, use the following code to reset parts of your cluster:

##### Reset SSH

```shell
filename=~/.ssh/authorized_keys
line=$(head -1 ${filename})
echo $line

sudo rm -rf ${filename}
echo -e "${line}" | tee --append ${filename} > /dev/null
sudo chmod 0600 ${filename}

sudo rm -rf ~/.ssh/id_rsa*
sudo rm -rf ~/.ssh/known_hosts
ssh-keygen  -f ~/.ssh/id_rsa -t rsa -P ""
sudo chmod 0600 ~/.ssh/id_rsa.pub
sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
sudo chmod 0600 ~/.ssh/authorized_keys
sudo chmod 0400 ~/.ssh/config
sudo chmod 0400 ~/.ssh/hadoop-clusterkeypair.pem

hosts=0.0.0.0,127.0.0.1,127.0.1.1,hadoop-master,DataNode001,DataNode002,DataNode003
ssh-keyscan -H ${hosts} >> ~/.ssh/known_hosts

sudo cat ~/.ssh/id_rsa.pub | ssh -o StrictHostKeyChecking=no DataNode001 'cat >> ~/.ssh/authorized_keys'
sudo cat ~/.ssh/id_rsa.pub | ssh -o StrictHostKeyChecking=no DataNode002 'cat >> ~/.ssh/authorized_keys'
sudo cat ~/.ssh/id_rsa.pub | ssh -o StrictHostKeyChecking=no DataNode003 'cat >> ~/.ssh/authorized_keys'
```

##### Reset Hadoop Installation Cluster Files and Directories

```shell
sudo rm -rf $HADOOP_DATA_HOME
sudo rm -rf $HADOOP_HOME/hadoop_data
sudo rm -rf $HADOOP_HOME/logs
mkdir -p $HADOOP_DATA_HOME/namenode
mkdir -p $HADOOP_DATA_HOME/datanode
mkdir -p $HADOOP_DATA_HOME/tmp
chmod -R 755 $HADOOP_DATA_HOME
```

Delete Hadoop

```shell
sudo rm -rf $HADOOP_HOME
```

##### Reset bigdata.sh

```shell
sudo rm -rf /etc/profile.d/bigdata.sh
sudo touch /etc/profile.d/bigdata.sh

sudo chmod +x /etc/profile.d/bigdata.sh
sudo echo -e '# Environment Variables for Big Data tools\n' | sudo tee --append /etc/profile.d/bigdata.sh > /dev/null 
```

#### 01.17 Start up your Hadoop Cluster

Execute the following code from you NameNode only.

##### Start All Hadoop Services

This command will start all Hadoop services however, it will be deprecated soon.

```shell
start-all.sh
```

##### Start the DFS service

If you choose to start the DFS separately, use this command:

```
start-dfs.sh
```

Browse the NameNode in your Web Browser, change localhost to your NameNode DNS or IP Address

```shell
http://localhost:50070
```

Try it now [http://localhost:50070](http://localhost:50070/)

On Amazon EC2 Instances your URL my look like this:

```shell
http://ec2-54-158-115-185.compute-1.amazonaws.com:50070
```

##### Start YARN on NameNode

Start yarn:

```
start-yarn.sh
```

Start the History Server

```
mr-jobhistory-daemon.sh start historyserver
```

Browse the ResourceManager in your Web Browser, change localhost to your NameNode DNS or IP Address

```
http://localhost:8088
```

Try it now [http://localhost:8088](http://localhost:8088/)

On Amazon EC2 Instances your URL my look like this:

```
http://ec2-54-158-115-185.compute-1.amazonaws.com:8088
```

##### Confirm that Hadoop services are running

Run this command to display the currently running Java processes

```
jps
```

Should produce a result similar this:

```
8065 SecondaryNameNode
8243 ResourceManager
8676 JobHistoryServer
7877 DataNode
7765 NameNode
8717 Jps
```

Check all ports used by Java

```shell
sudo netstat -plten | grep java
```

This should produce an output similar to the following:

```shell
tcp        0      0 0.0.0.0:50070           0.0.0.0:*               LISTEN      1000       18803       1814/java
tcp        0      0 0.0.0.0:50010           0.0.0.0:*               LISTEN      1000       19934       1972/java
tcp        0      0 0.0.0.0:50075           0.0.0.0:*               LISTEN      1000       20222       1972/java
tcp        0      0 0.0.0.0:50020           0.0.0.0:*               LISTEN      1000       20225       1972/java
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      1000       18812       1814/java
tcp        0      0 0.0.0.0:50090           0.0.0.0:*               LISTEN      1000       21596       2167/java
tcp        0      0 127.0.0.1:39787         0.0.0.0:*               LISTEN      1000       19941       1972/java
tcp6       0      0 :::45523                :::*                    LISTEN      1000       28034       2448/java
tcp6       0      0 :::8088                 :::*                    LISTEN      1000       28033       2322/java
tcp6       0      0 :::8030                 :::*                    LISTEN      1000       23241       2322/java
tcp6       0      0 :::8031                 :::*                    LISTEN      1000       22685       2322/java
tcp6       0      0 :::8032                 :::*                    LISTEN      1000       23247       2322/java
tcp6       0      0 :::8040                 :::*                    LISTEN      1000       29126       2448/java
```

Generate a Report from your Cluster

```shell
hdfs dfsadmin -report
```

The result should look similar to the following:

```shell
Configured Capacity: 16518029312 (15.38 GB)
Present Capacity: 10704764928 (9.97 GB)
DFS Remaining: 10704715776 (9.97 GB)
DFS Used: 49152 (48 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0
Pending deletion blocks: 0

-------------------------------------------------
Live datanodes (2):

Name: 172.31.24.96:50010 (ip-172-31-24-96.ec2.internal)
Hostname: ip-172-31-24-96.ec2.internal
Decommission Status : Normal
Configured Capacity: 8259014656 (7.69 GB)
DFS Used: 24576 (24 KB)
Non DFS Used: 2889826304 (2.69 GB)
DFS Remaining: 5352386560 (4.98 GB)
DFS Used%: 0.00%
DFS Remaining%: 64.81%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Jan 08 00:15:34 UTC 2018
Last Block Report: Mon Jan 08 00:15:07 UTC 2018


Name: 172.31.30.70:50010 (ip-172-31-30-70.ec2.internal)
Hostname: ip-172-31-30-70.ec2.internal
Decommission Status : Normal
Configured Capacity: 8259014656 (7.69 GB)
DFS Used: 24576 (24 KB)
Non DFS Used: 2889883648 (2.69 GB)
DFS Remaining: 5352329216 (4.98 GB)
DFS Used%: 0.00%
DFS Remaining%: 64.81%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Jan 08 00:15:34 UTC 2018
Last Block Report: Mon Jan 08 00:15:07 UTC 2018
```

#### 01.18 Test the HDFS

Create txt file in your local home folder

```shell
echo "Hello this will be my first distributed and fault-tolerant data set\!" | cat >> my_file.txt
```

List the hdfs files

```shell
hdfs dfs -ls /
```

Make a folder named user

```shell
hdfs dfs -mkdir /user
```

List the hdfs files

```shell
hdfs dfs -ls /
```

 the created file a few times

```shell
hdfs dfs -FromLocal ~/my_file.txt /user

hdfs dfs -FromLocal ~/my_file.txt /user/my_file2.txt

hdfs dfs -FromLocal ~/my_file.txt /user/my_file3.txt
```

List the files in the new folder

```shell
hdfs dfs -ls /user
```

Remove the files with a name starting with `my_file`

```shell
hdfs dfs -rm /user/my_file*
```

Remove the folder you

```shell
hdfs dfs -rmdir /user
```

Linked Resources

- [HDFS Command Line - https://hadoop.apache.org/docs/r2.7.1/hadoop-proje...](https://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/FileSystemShell.html)

#### 01.19 Example MapReduce on Hadoop

Test that Hadoop can run MapReduce jobs:

```shell
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.0.jar
```

This will produce a result similar to this:

```
An example program must be given as the first argument.
Valid program names are:
  aggregatewordcount: An Aggregate based map/reduce program that counts the words in the input files.
  aggregatewordhist: An Aggregate based map/reduce program that computes the histogram of the words in the input files.
  bbp: A map/reduce program that uses Bailey-Borwein-Plouffe to compute exact digits of Pi.
  dbcount: An example job that count the pageview counts from a database.
  distbbp: A map/reduce program that uses a BBP-type formula to compute exact bits of Pi.
  grep: A map/reduce program that counts the matches of a regex in the input.
  join: A job that effects a join over sorted, equally partitioned datasets
  multifilewc: A job that counts words from several files.
  pentomino: A map/reduce tile laying program to find solutions to pentomino problems.
  pi: A map/reduce program that estimates Pi using a quasi-Monte Carlo method.
  randomtextwriter: A map/reduce program that writes 10GB of random textual data per node.
  randomwriter: A map/reduce program that writes 10GB of random data per node.
  secondarysort: An example defining a secondary sort to the reduce.
  sort: A map/reduce program that sorts the data written by the random writer.
  sudoku: A sudoku solver.
  teragen: Generate data for the terasort
  terasort: Run the terasort
  teravalidate: Checking results of terasort
  wordcount: A map/reduce program that counts the words in the input files.
  wordmean: A map/reduce program that counts the average length of the words in the input files.
  wordmedian: A map/reduce program that counts the median length of the words in the input files.
  wordstandarddeviation: A map/reduce program that counts the standard deviation of the length of the words in the input files.
```

Start Hadoop HDFS and Resource Manager

```
start-all.sh
```

Setup the environment for your application:

```shell
hdfs dfs -mkdir -p /user/hduser/input
hdfs dfs -chmod -R 777 /user
hdfs dfs -put $HADOOP_HOME/*.txt /user/hduser/input
hdfs dfs -ls /user/hduser/input
```

This will produce a result similar to this:

```shell
Found 3 items
-rw-r--r--   1 hduser supergroup     106210 2017-12-06 13:58 /user/hduser/input/LICENSE.txt
-rw-r--r--   1 hduser supergroup      15915 2017-12-06 13:58 /user/hduser/input/NOTICE.txt
-rw-r--r--   1 hduser supergroup       1366 2017-12-06 13:58 /user/hduser/input/README.txt
```

Run the Wordcount MapReduce job:

```shell
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.0.jar  wordcount /user/hduser/input /user/hduser/output
```

The should produce a result that looks similar to the following:

```
17/12/06 14:06:20 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
17/12/06 14:06:24 INFO input.FileInputFormat: Total input files to process : 3
17/12/06 14:06:24 INFO mapreduce.JobSubmitter: number of splits:3
17/12/06 14:06:25 INFO Configuration.deprecation: yarn.resourcemanager.system-metrics-publisher.enabled is deprecated. Instead, use yarn.system-metrics-publisher.enabled
17/12/06 14:06:25 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1512596939837_0002
17/12/06 14:06:29 INFO impl.YarnClientImpl: Submitted application application_1512596939837_0002
17/12/06 14:06:30 INFO mapreduce.Job: The url to track the job: http://ubuntu:8088/proxy/application_1512596939837_0002/
17/12/06 14:06:30 INFO mapreduce.Job: Running job: job_1512596939837_0002
17/12/06 14:07:10 INFO mapreduce.Job: Job job_1512596939837_0002 running in uber mode : false
17/12/06 14:07:10 INFO mapreduce.Job:  map 0% reduce 0%
17/12/06 14:08:16 INFO mapreduce.Job:  map 33% reduce 0%
17/12/06 14:08:19 INFO mapreduce.Job:  map 67% reduce 0%
17/12/06 14:08:20 INFO mapreduce.Job:  map 100% reduce 0%
17/12/06 14:08:35 INFO mapreduce.Job:  map 100% reduce 100%
17/12/06 14:08:37 INFO mapreduce.Job: Job job_1512596939837_0002 completed successfully
17/12/06 14:08:38 INFO mapreduce.Job: Counters: 50
	File System Counters
		FILE: Number of bytes read=50653
		FILE: Number of bytes written=908579
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=123837
		HDFS: Number of bytes written=36040
		HDFS: Number of read operations=12
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters 
		Killed map tasks=1
		Launched map tasks=3
		Launched reduce tasks=1
		Data-local map tasks=3
		Total time spent by all maps in occupied slots (ms)=194319
		Total time spent by all reduces in occupied slots (ms)=15197
		Total time spent by all map tasks (ms)=194319
		Total time spent by all reduce tasks (ms)=15197
		Total vcore-milliseconds taken by all map tasks=194319
		Total vcore-milliseconds taken by all reduce tasks=15197
		Total megabyte-milliseconds taken by all map tasks=198982656
		Total megabyte-milliseconds taken by all reduce tasks=15561728
	Map-Reduce Framework
		Map input records=2461
		Map output records=17424
		Map output bytes=190556
		Map output materialized bytes=50665
		Input split bytes=346
		Combine input records=17424
		Combine output records=3125
		Reduce input groups=2851
		Reduce shuffle bytes=50665
		Reduce input records=3125
		Reduce output records=2851
		Spilled Records=6250
		Shuffled Maps =3
		Failed Shuffles=0
		Merged Map outputs=3
		GC time elapsed (ms)=4999
		CPU time spent (ms)=15100
		Physical memory (bytes) snapshot=877674496
		Virtual memory (bytes) snapshot=7798611968
		Total committed heap usage (bytes)=603783168
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=123491
	File Output Format Counters 
		Bytes Written=36040
```

Display the resulting output files

```shell
hdfs dfs -cat /user/hduser/output/*
```

The should produce a result that looks similar to the following:

```
""AS	2
"AS	22
"RIGHTS	1
"Collective	1
"Contribution"	2
"Contributor"	2
"Derivative	2
"French	1
"GCC	1
"LICENSE").	1
"Legal	1
"License"	1
"License");	3
"Licensed	1
```

Linked Resources

- [Apache Hadoop 3.0.0-beta1 – MapReduce Tutorial - http://hadoop.apache.org/docs/current/hadoop-mapre...](http://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)
- [Eclipse Download - http://www.eclipse.org/downloads](http://www.eclipse.org/downloads)
- [Hadoop - MapReduce WordCount - https://youtu.be/gmSc9KwViFs?list=PLWsYJ2ygHmWhOvt...](https://youtu.be/gmSc9KwViFs?list=PLWsYJ2ygHmWhOvtHIPxGJDtki9SJlINCw)