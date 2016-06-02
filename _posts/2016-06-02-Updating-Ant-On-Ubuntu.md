---
layout: post
title: Updating Ant On Ubuntu
---

1. Get source archive:

 From: http://ant.apache.org/srcdownload.cgi
 

2. Extract contents of the source archive: (assuming a tar.gz archive and version 1.9.7)

 ::

 > tar xvf apache-ant-1.9.7-src.tar.gz
 > cd apache-ant-1.9.7


3. Fetch dependencies:

 (This assumes you are updating and have an old version of ant installed).

 ::

 > ant -f fetch.xml -Ddest=user

 Details: http://ant.apache.org/manual/install.html#optionalTasks


4. Build:

 ::

 > sh build.sh -Ddist.dir=bin dist


5. Install:

 (Uninstall previous version of ant if any).

 ::

 > sudo mkdir /usr/local/ant
 > cd bin
 > sudo cp -r * /usr/local/ant
 

6. Setup Environment: (assuming bash)

 ::

 > export ANT_HOME=/usr/local/ant
 > export PATH=${PATH}:${ANT_HOME}/bin

 Also, setup JAVA_HOME if not already done.

7. Check:
 
 Open a new terminal::

 > ant -version
 > Apache Ant(TM) version 1.9.7 compiled on June 1 2016

All Set !!

