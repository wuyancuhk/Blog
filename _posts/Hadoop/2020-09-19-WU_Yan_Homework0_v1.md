---
layout:     post
title:      "Hadoop Learning Notes - 2"
subtitle:   " \"Hadoop - a simple example - Word Count\""
date:       2020.09.19 23:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 作业
    - Hadoop




---

> *"Keep Learning Hadoop"*

# Homework 0 of IERG4300-Big Data Technology and Applications 

# a. Single-node Hadoop Setup

## i. The  namenode web page(attached in the end)

You can also check the print page at the end of this PDF file

![image-20200919232053995](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919232054.png)

## ii. The Terasort example running process 

### 1. Run the commands in Putty

![image-20200917222337559](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917222337.png)

![image-20200917223006488](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917223006.png)

![image-20200917223418147](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917223418.png)

### 2. Check the logs in jobhistory web page

![image-20200919232540064](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919232540.png)

![image-20200919232623166](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919232623.png)

![image-20200919232648397](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919232648.png)

# b. Multi-node Hadoop Cluster Setup

## i. "jps" command on 4 VMs

![image-20200917223826009](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917223826.png)

![image-20200917223849847](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917223849.png)

![image-20200917223910977](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917223911.png)

![image-20200917223931537](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917223931.png)

## ii. Run Terasort example with 2GB data

### check the time comsumed(about 27 minutes)

![image-20200919232742485](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919232742.png)

## iii. Run Terasort example with 20 GB data

### 1. Run commands in Putty(the same as above)

![image-20200919171922759](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919171922.png)

![image-20200919171837210](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919171844.png)

![image-20200919171944271](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919171944.png)

### 2. Check the time consumed(about 10 minutes)

![image-20200919233047116](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919233047.png)

# c. Running the Python Code on Hadoop

## i. Necessary modifications

### 1. Download the Lemon Juice's artical

![image-20200919223310785](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919223310.png)

### 2. Create a directory on HDFS

![image-20200919223737383](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919223737.png)

![image-20200919223351839](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919223351.png)

### 3. Modify the .py files' authorty and format

```shell
$ chmod +x mapper.py

$ chmod +x reducer.py
```

`add on the second line : `# -*-coding:utf-8 -*``

![image-20200919223451037](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919223451.png)

### 4. Run the following codes

```shell
$ hadoop jar hadoop-streaming-2.9.2.jar -file /usr/local/hadoop/share/hadoop/mapreduce/mapper.py -mapper /usr/local/hadoop/share/hadoop/mapreduce/mapper.py -file /usr/local/hadoop/share/hadoop/mapreduce/reducer.py -reducer /usr/local/hadoop/share/hadoop/mapreduce/reducer.py -input /tmp/input/* -output output4
```

### 5. Check the result

![image-20200919224021737](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919224021.png)

### 6. Cat the file to VM and check the result

```shell
$ hadoop fs -cat /user/ubuntu/output4/part-00000 > resultofLemonJuice.txt
```

![](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200920001324.png)

# d. Compiling the Java WordCount program for MapReduce

## i. Make a preparation for running this job

### 1. Enter these path variables into $HADOOP_HOME/etc/hadoop/hadoop-env.sh

```shell
export PATH=${JAVA_HOME}/bin:${PATH}
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
```

### 2. Compile `WordCount.java` and create a jar

```shell
$ hadoop com.sun.tools.javac.Main WordCount.java
$ jar cf wc.jar WordCount*.class
```

![image-20200919235549322](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919235549.png)

### 3. Create input directory in HDFS and upload the shakespear_basket1/2 onto it

```shell
$ hdfs dfs -put shakespeare-basket1 shakespeare_basket2 /user/ubuntu/wordcount/input
```

![image-20200919235727901](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200919235728.png)

### 4. Run the application

```shell
$ hadoop jar wc.jar org.apache.hadoop.examples.WordCount /user/ubuntu/wordcount/input /user/ubuntu/wordcount/output
```

![image-20200920000032687](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200920000032.png)

### 5. Duplicate the output into a .txt file

```shell
$ hadoop fs -cat /user/ubuntu/wordcount/output/part-r-00000 > resultofshakespear.txt
```

![image-20200920000514114](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200920000514.png)

### 6. Compare the running time with task c(almost the same)

![image-20200920000917833](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200920000917.png)

