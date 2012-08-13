AWS Auto Discovery for JGroups
==============================
Overview
--------
This package provides auto discovery for other custer members on AWS using both tag matching and filters.  It is
a drop in replacement for TCPPING, allowing you to remove the definition of your initial members from your configuraiton
file.

Usage
-----
To use AWS auto discovery, you need to add a dependency to this package in your pom:
```
    <dependency>
      <groupId>com.meltmedia.jgroups.aws</groupId>
      <artifactId>jgroups-aws</artifactId>
      <version>1.0.0</version>
    </dependency>
```
and then replace TCPPING in your stack with com.meltmedia.jgroups.aws.AWS_PING:
```
    <com.meltmedia.jgroups.aws.AWS_PING
         timeout="3000"
         port_number="7800"
         tags="TAG1,TAG2"
         filters="NAME1=VALUE1,VALUE2;NAME2=VALUE3"
         access_key="AWS_ACCESS_KEY"
         secret_key="AWS_SECRET_KEY"/>
```
see the configuration section for information.  You can find an example stack in conf/aws_ping.xml.

This implementation will only work from inside EC2, since it uses environment information to auto wire itself.  See the
Setting Up EC2 section for more information.

Configuration Options
---------------------
* timeout - the timeout in milliseconds
* port_number - the port number that the nodes will communicate over.  This needs to be the same on all nodes.  The default is 7800.
* tags - A comma delimited list of EC2 node tag names.  The current nodes values are matched against other nodes to find
cluster members.
* filters - A colon delimited list of filters.  Each filter defines a name and a comma delimited list of possible values.
All filters mutch mach a node for it to be a cluster member.
* access_key (required) - the access key for an AWS user with permission to the "ec2:Describe*" action.
* secret_key (required) - the secret key for the AWS user.

Setting Up EC2
--------------
You will need to setup the following in EC2, before using this package:
* You will need to create a user in the IAM console.  You will need the API access key and secret key for the user.  You will also 
need to grant the user permission to the "ec2:Describe*" action.
* In the EC2 console, you will need to create a security group for your instances.  This security group will need a TCP_ALL rule,
with itself as the source (put the security groups name in the source filed.)  This will allow all of the nodes in the security
group to communicate with each other.
* Create 2 EC2 nodes, making sure to include the security group granting TCP communication.
* If you are going to use the tag matching feature, then define a few tags on the nodes with matching values.

Setting up JGroups Chat Demo
----------------------------
The JGroups project provides a chat application that is great for testing your configuration.  To set up the chat application,
first create 2 EC2 nodes, following the Setting Up EC2 instructions.  Once the nodes are created, SSH into each machine and
install the java 6 JDK, Maven 3, and Git.
```
sudo apt-get install openjdk-6-jdk
wget http://www.carfab.com/apachesoftware/maven/binaries/apache-maven-3.0.4-bin.tar.gz
sudo tar xzf apache-maven-3.0.4-bin.tar.gz /opt
ln -s /opt/maven /opt/apache-maven-3.0.4
echo "export PATH=/opt/maven/bin:$PATH" >> .profile
. ~/.profile
sudo apt-get install git
mkdir ~/git
```
Now your machine is ready to compile maven projects.

Next, clone and build this project.
```
cd ~/git
git clone git://github.com/meltmedia/jgroups-aws.git
cd jgroups-aws
mvn clean install
```
Finally, it is time to run the project.  You will need to edit the configuration in conf/aws-ping.xml.  Add values for the
tags, access_key and secret_key attributes.  Remove the filters attribute.  Then execute the following:
```
mvn exec:java -Dexec.mainClass="org.jgroups.demos.Chat" -Dexec.args="-props conf/aws-ping.xml"
```


