---
layout: post
title: Updating Ant On Ubuntu
---


### 1. Get source archive

 From: [ant download page](http://ant.apache.org/srcdownload.cgi)
 

### 2. Extract contents of the source archive (assuming a tar.gz archive and version 1.9.7)

 {% highlight bash %}
 > tar xvf apache-ant-1.9.7-src.tar.gz
 > cd apache-ant-1.9.7
 {% endhighlight %}


### 3. Fetch dependencies

 {% highlight bash %}
 > ant -f fetch.xml -Ddest=user
 {% endhighlight %}

 [more details here]( http://ant.apache.org/manual/install.html#optionalTasks)


### 4. Build

 {% highlight bash %}
 > sh build.sh -Ddist.dir=bin dist
 {% endhighlight %}


### 5. Install

 Uninstall the previous version of ant.

 {% highlight bash %}
 > sudo mkdir /usr/local/ant
 > cd bin
 > sudo cp -r * /usr/local/ant
 {% endhighlight %}
 

### 6. Setup Environment (assuming bash)

 {% highlight bash %}
 > export ANT_HOME=/usr/local/ant
 > export PATH=${PATH}:${ANT_HOME}/bin
 {% endhighlight %}

 Also, setup JAVA_HOME if not already done.

### 7. Check
 
 Open a new terminal.

 {% highlight bash %}
 > ant -version
 > Apache Ant(TM) version 1.9.7 compiled on June 1 2016
 {% endhighlight %}

All Set !!

