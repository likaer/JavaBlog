---
title: 浅谈settings.xml中的几个关键节点如何配置
date: 2018-09-17 19:00:32
tags: Maven
---

今天在公司进行项目构建， 遇见了几个坑，排完坑做个总结

## 0. 配之前先了解maven的Repository概念和逻辑

![Maven repo](images/maven_repo.png)


1. maven默认会先从local repository查看是否存在dependency
2. 不存在的情况下会从central repository获取dependency
3. 不存在的情况下会从remote repository获取dependency
4. 无论是central还是remote repository都会走到mirror做一层转换

![Mirror](images/mirror.png)



## 1. 首先看一下根路径下的节点有哪些

		<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
	      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
	                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
	      <localRepository/>
	      <interactiveMode/>
	      <usePluginRegistry/>
	      <offline/>
	      <pluginGroups/>
	      <servers/>
	      <mirrors/>
	      <proxies/>
	      <profiles/>
	      <activeProfiles/>
	    </settings>


## 2. [Mirrors](https://maven.apache.org/guides/mini/guide-mirror-settings.html)

Mirror是Repository的映射，例如公司有自己的Repository， 那就可以把Central Repository映射到公司的仓库


		<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
		  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
		                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
		  ...
		  <mirrors>
		    <mirror>
		      <id>planetmirror.com</id>
		      <name>PlanetMirror Australia</name>
		      <url>http://downloads.planetmirror.com/pub/maven2</url>
		      <mirrorOf>central</mirrorOf>
		    </mirror>
		  </mirrors>
		  ...
		</settings>

以上是一个常见的Mirror， id是唯一标识符，name描述你的mirror，url是你的mirror地址。

常用的mirrorOf有几种：
1. `*` : 永远只从当前的repository下载dependency，**会导致pom引用的第三方repository仓库失效** 需要自己上传第三方dependency到你的公司仓库
2. `repo,repo1` :  映射id=repo和repo1的仓库到该镜像
3. `*,!repo1` : 映射除了repo1的所有仓库到该镜像

## 2. servers

servers配置了远端仓库的安全验证信息

		<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
		  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
		                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
		  ...
		  <servers>
		    <server>
		      <id>server001</id>
		      <username>my_login</username>
		      <password>my_password</password>
		      <privateKey>${user.home}/.ssh/id_dsa</privateKey>
		      <passphrase>some_passphrase</passphrase>
		      <filePermissions>664</filePermissions>
		      <directoryPermissions>775</directoryPermissions>
		      <configuration></configuration>
		    </server>
		  </servers>
		  ...
		</settings>


## 3. Repositories

Repositories定义了你的默认远端仓库


		<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
		  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
		                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
		  ...
		  <profiles>
		    <profile>
		      ...
		      <repositories>
		        <repository>
		          <id>codehausSnapshots</id>
		          <name>Codehaus Snapshots</name>
		          <releases>
		            <enabled>false</enabled>
		            <updatePolicy>always</updatePolicy>
		            <checksumPolicy>warn</checksumPolicy>
		          </releases>
		          <snapshots>
		            <enabled>true</enabled>
		            <updatePolicy>never</updatePolicy>
		            <checksumPolicy>fail</checksumPolicy>
		          </snapshots>
		          <url>http://snapshots.maven.codehaus.org/maven2</url>
		          <layout>default</layout>
		        </repository>
		      </repositories>
		      <pluginRepositories>
		        ...
		      </pluginRepositories>
		      ...
		    </profile>
		  </profiles>
		  ...
		</settings>


updatePolicy: 定义dependency的更新频率，always(实时), daily (每天，默认值), interval:X (X分钟更新一次), never(从不).


## 4. Active Profiles

profiles下有多个profile， active profiles代表当前profile是否生效，所以不是配了profiles就行。


		<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
		  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
		                      https://maven.apache.org/xsd/settings-1.0.0.xsd">
		  ...
		  <activeProfiles>
		    <activeProfile>env-test</activeProfile>
		  </activeProfiles>
		</settings>


## 5. 一个建议的settings.xml模板

		<settings>
		  <mirrors>
		    <mirror>
		      <id>nexus</id>
		      <mirrorOf>central</mirrorOf>
		      <!-- 永远只从当前的repository下载dependency使用* -->
		      <!-- mirrorOf>*</mirrorOf -->
		      <url>http://localhost:8081/nexus/content/groups/public/</url>
		    </mirror>
		  </mirrors>
		  <profiles>
		    <profile>
		      <id>profile1</id>
		      <!--Enable snapshots for the built in central repo to direct -->
		      <!--all requests to nexus via the mirror -->
		      <repositories>
		        <repository>
		          <id>repo1</id>
		          <url>http://central</url>
		          <releases>
		          	<enabled>true</enabled>
	          	  </releases>
		          <snapshots>
		            <enabled>true</enabled>
		            <updatePolicy>always</updatePolicy>
		          </snapshots>
		        </repository>
		      </repositories>
		     <pluginRepositories>
		        <pluginRepository>
		          <id>repo1</id>
		          <url>http://central</url>
		          <releases><enabled>true</enabled></releases>
		          <snapshots><enabled>true</enabled></snapshots>
		        </pluginRepository>
		      </pluginRepositories>
		    </profile>
		  </profiles>
		  <activeProfiles>
		    <!--make the profile active all the time -->
		    <activeProfile>profile1</activeProfile>
		  </activeProfiles>
		  <servers>
		    <!-- Possible authentications for access to nexus -->
		    <server>
		      <id>snapshots</id>
		      <username>xxx</username>
		      <password>xxx</password>
		    </server>
		    <server>
		      <id>releases</id>
		      <username>xxx</username>
		      <password>xxx</password>
		    </server>
		  </servers>
		</settings>