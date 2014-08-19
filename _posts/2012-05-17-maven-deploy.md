---
layout: post
title: maven deploy
description:
- nexus maven deploy
categories:
- maven
tags:
- maven
- nexus
---
####settings.xml
<pre class="prettyprint"><xmp><servers>
	<server>  
	       <id>nexus-releases</id>  
	       <username>admin</username>  
	       <password>admin123</password>  
	</server>  
	<server>  
	       <id>nexus-snapshots</id>  
	       <username>admin</username>  
	       <password>admin123</password>  
	</server>  
</servers> </xmp></pre>

http://localhost:8081/nexus/
security-users
有三个默认用户
admin
deployment
anonymous
密码用右键更改

####工程 pom.xml
<pre class="prettyprint"><xmp><distributionManagement>
  <repository>
      <id>nexus-releases</id>
      <name>Local Nexus Repository</name>
      <url>http://localhost:8081/content/repositories/releases</url>
  </repository>
  <snapshotRepository>
      <id>nexus-snapshots</id>
      <name>Local Nexus Repository</name>
      <url>http://localhost:8081/content/repositories/snapshots</url>
  </snapshotRepository>
</distributionManagement>
 </xmp></pre>
id 要相同