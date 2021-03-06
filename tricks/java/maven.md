# Maven

## Tips
```
-pl, --projects
        Build specified reactor projects instead of all projects
-am, --also-make
        If project list is specified, also build projects required by the list
```

```bash
# 可通过命令行修改仓库
mvn package -Dmaven.repo.remote=http://maven.aliyun.com/nexus/content/groups/public -Dmaven.repo.local="$HOME/repo"
# 下载单个
mvn dependency:get -DrepoUrl=http://maven.aliyun.com/nexus/content/groups/public -Dartifact=org.redisson:redisson:3.2.0

# 获取项目信息,在命令行下比较有用
mvn -q -Dexec.executable="echo" -Dexec.args='${project.artifactId}' --non-recursive org.codehaus.mojo:exec-maven-plugin:exec

# 强制更新 SNAPSHOT
mvn clean install -U
```

## maven-release-plugin
* Maven Release
* [Maven Release Plugin](http://maven.apache.org/maven-release/maven-release-plugin/)
* http://central.sonatype.org/pages/apache-maven.html
* Goals
  * release:clean Clean up after a release preparation.
  * release:prepare Prepare for a release in SCM.
  * release:prepare-with-pom Prepare for a release in SCM, and generate release POMs that record the fully resolved projects used.
  * release:rollback Rollback a previous release.
  * release:perform Perform a release from SCM.
  * release:stage Perform a release from SCM into a staging folder/repository.
  * release:branch Create a branch of the current project with all versions updated.
  * release:update-versions Update the versions in the POM(s).

```bash
mvn release:help

# 准备发布
mvn release:clean release:prepare
# 执行发布
# deploy & push
mvn release:perform
# 回滚操作
mvn release:rollback
```

### 中央仓库
* [Guide to uploading artifacts to the Central Repository](https://maven.apache.org/guides/mini/guide-central-repository-upload.html)
* [Deploy to Maven Central Repository](https://dzone.com/articles/deploy-maven-central)
* [OSSRH Guide](http://central.sonatype.org/pages/ossrh-guide.html)

### 第三方仓库

一般使用镜像有以下几种方式

* 在 POM 中添加仓库
  * 粘贴复制下就能使用
  * 会持久在项目中
    * 团队中其他人也不需要配置
  * 如果在镜像的仓库中找不到会在中央仓库找
* 在 setting 中添加镜像
  * 需要调整 setting.xml 相对麻烦一些
  * 在仓库中找不到会出错
  * 当项目中有多个模块时,使用镜像可能会出现找不到本地模块的问题
* 在 setting 中添加 profile

#### 阿里云

```xml
<!-- 可以在 POM 中加入如下仓库配置来使用 阿里云的 Maven 仓库镜像 -->
<!-- 注意: 阿里云 Maven 仓库镜像同步相对较慢,可能几天或者十几天才能同步 -->
<repositories>
    <repository>
        <id>aliyun</id>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>aliyun</id>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </pluginRepository>
</pluginRepositories>
```

#### 谷歌
```xml
<!-- 同步速度快,访问速度也很快,没有被封 -->
<repositories>
    <repository>
       <id>google</id>
       <url>https://maven-central.storage.googleapis.com</url>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>google</id>
        <url>https://maven-central.storage.googleapis.com</url>
    </pluginRepository>
</pluginRepositories>
```
#### setting 配置

```xml
<settings>
  <mirrors>

    <!-- Google -->
    <mirror>
      <id>google-maven-central</id>
      <name>Google Maven Central</name>
      <url>https://maven-central.storage.googleapis.com</url>
      <mirrorOf>central</mirrorOf>
    </mirror>

    <!-- OSChina -->
    <mirror>  
    		<id>nexus-osc</id>  
        <mirrorOf>*</mirrorOf>  
    	  <name>Nexusosc</name>  
    	  <url>http://maven.oschina.net/content/groups/public/</url>  
    </mirror>

    <!-- 阿里 -->
    <mirror>
         <id>aliyun</id>
         <mirrorOf>central</mirrorOf>
         <name>aliyun</name>
         <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
  </mirrors>
  <!-- 阿里 -->
        <profile>
            <id>nexus</id>
            <repositories>
                <repository>
                    <id>central</id>
                    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
                <repository>
                    <id>snapshots</id>
                    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>central</id>
                    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </pluginRepository>
                <pluginRepository>
                    <id>snapshots</id>
                    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>

    </profiles>
</settings>
```

## Plugins

### delombok
```xml
<plugin>
     <groupId>org.projectlombok</groupId>
     <artifactId>lombok-maven-plugin</artifactId>
     <version>${org.projectlombok.version}.0</version>
     <dependencies>
         <dependency>
             <groupId>sun.jdk</groupId>
             <artifactId>tools</artifactId>
             <version>1.7</version>
             <scope>system</scope>
             <systemPath>${java.home}/../lib/tools.jar</systemPath>
         </dependency>
     </dependencies>
     <executions>
         <execution>
             <id>delombok</id>
             <phase>generate-sources</phase>
             <goals>
                 <goal>delombok</goal>
             </goals>
             <configuration>
                 <encoding>UTF-8</encoding>
                 <addOutputDirectory>false</addOutputDirectory>
                 <sourceDirectory>target/generated-sources/lombok</sourceDirectory>
             </configuration>
         </execution>
         <execution>
             <id>test-delombok</id>
             <phase>generate-test-sources</phase>
             <goals>
                 <goal>testDelombok</goal>
             </goals>
             <configuration>
                 <encoding>UTF-8</encoding>
                 <addOutputDirectory>false</addOutputDirectory>
                 <sourceDirectory>target/generated-sources/lombok</sourceDirectory>
             </configuration>
         </execution>
     </executions>
 </plugin>
```

## 仓库管理

* [仓库管理](https://maven.apache.org/repository-management.html)
* [Artifactory 特性比较](https://www.jfrog.com/confluence/display/RTF/Artifactory+Comparison+Matrix)
* [Apache Archiva](https://archiva.apache.org)
* Nexus
  * 支持 bower,docker,maven2,npm,nuget,pypi,raw,rubygems
  * 3.x 支持 Docker

```bash
# ARCHIVA_BASE 可修改存储位置, 默认为 /var/archiva
# ARCHIVA_CONTEXT_PATH 可在反向代理是使用, 默认为 /
docker run -v $PWD/archiva:/var/archiva -p 8080:8080 -d ninjaben/archiva-docker
```

__settings.xml__
```xml

```


将 jar 安装到本地仓库
----
mvn install:install-file -Dfile=c:\kaptcha-{version}.jar -DgroupId=com.google.code -DartifactId=kaptcha -Dversion={version} -Dpackaging=jar

mvn install:install-file
-Dfile=<path-to-file>
-DgroupId=<group-id>
-DartifactId=<artifact-id>
-Dversion=<version>
-Dpackaging=<packaging>
-DgeneratePom=true

Where: <path-to-file>  the path to the file to load
   <group-id>      the group that the file should be registered under
   <artifact-id>   the artifact name for the file
   <version>       the version of the file
   <packaging>     the packaging of the file e.g. jar

打包 jar 的参数   
----
http://maven.apache.org/plugins/maven-jar-plugin/jar-mojo.html


准备好上传
---------
```cmd
:: 手册 https://docs.sonatype.org/display/Repository/Sonatype+OSS+Maven+Repository+Usage+Guide
cd ..
mvn source:jar javadoc:jar
cd target
cp ../pom.xml .
SET PROJ_NAME=IKAnalyzer-2012_u6
del *.asc
gpg -ab pom.xml
gpg -ab %PROJ_NAME%.jar
gpg -ab %PROJ_NAME%-sources.jar
gpg -ab %PROJ_NAME%-javadoc.jar
```

直接部署存在的
-------------
```
:: 具体参数参考 http://maven.apache.org/plugins/maven-gpg-plugin/sign-mojo.html
:: 可以直接在命令行上指定密码
PASSPHRASE=xxx
mvn gpg:sign-and-deploy-file -Dgpg.passphrase=%PASSPHRASE% ^
	-Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ -DrepositoryId=sonatype-nexus-staging -DpomFile=pom.xml -Dfile=IKAnalyzer-2012_u6.jar
mvn gpg:sign-and-deploy-file -Dgpg.passphrase=%PASSPHRASE% ^
	-Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ -DrepositoryId=sonatype-nexus-staging -DpomFile=pom.xml -Dfile=IKAnalyzer-2012_u6-sources.jar -Dclassifier=sources
mvn gpg:sign-and-deploy-file -Dgpg.passphrase=%PASSPHRASE% ^
	-Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ -DrepositoryId=sonatype-nexus-staging -DpomFile=pom.xml -Dfile=IKAnalyzer-2012_u6-javadoc.jar -Dclassifier=javadoc

```

上传时的 gpg认证
----------------
https://github.com/sevntu-checkstyle/dsm-maven-plugin/wiki/How-to-config-GPG-and-sign-artifact-with-it

```
gpg --gen-key
gpg --list-keys
gpg --keyserver hkp://pool.sks-keyservers.net --send-keys 09CB6FEF
gpg -ab artifact.jar
gpg --verify artifact.jar.asc
```

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-gpg-plugin</artifactId>
	<version>1.4</version>
	<executions>
		<execution>
			<id>sign-artifacts</id>
			<phase>verify</phase>
			<goals>
				<goal>sign</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

```bash
# maven 使用 socks 代理
export MAVEN_OPTS="-DsocksProxyHost=127.0.0.1 -DsocksProxyPort=8888"
export MAVEN_OPTS="-DhttpProxyHost=192.168.1.103 -DhttpProxyPort=8087 -DhttpsProxyHost=192.168.1.103 -DhttpsProxyPort=8087"

export MAVEN_OPTS="-DhttpProxyHost=127.0.0.1 -DhttpProxyPort=7777 -DhttpsProxyHost=127.0.0.1 -DhttpsProxyPort=7777"

# 不调用javadoc 插件
-Dmaven.javadoc.skip=true
# 不测试
-Dmaven.test.skip=true
# -Dmaven.test.skip prevents Maven building the test-jar artifact.
# If you'd like to skip tests but create artifacts as per a normal build use:
-Dmaven.test.skip.exec
```

# maven 代理设置
```xml
<proxy>
	<id>myproxy</id>
	<active>true</active>
	<protocol>http</protocol>
	<host>127.0.0.1</host>
	<port>8087</port>
	<nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
</proxy>
```
## Tomcat 配置

__pom.xml__
```xml
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.2</version>
    <configuration>
        <url>http://localhost:8080/manager/text</url>
        <server>tomcatserver</server>
        <path>/mycontext</path>
        <username>admin</username>
        <password>admin</password>
    </configuration>
</plugin>
```

__settings.xml__
```xml
<server>
    <id>tomcatserver</id>
    <username>admin</username>
    <password>admin</password>
</server>
```

# spring 的仓库
http://repo.spring.io/libs-milestone
https://code.lds.org/nexus/content/groups/main-repo
https://repository.jboss.org/nexus/content/repositories/releases
http://repo.typesafe.com/typesafereadonly/releases
# 添加其他的仓库
```xml
<repository>
	<id>nexus-osc</id>
	<url>http://maven.oschina.net/content/groups/public/</url>
</repository>
```
# maven 配置 conf/settings.xml
# 国内镜像

```xml

```

```xml
<profile>
	<id>jdk-1.7</id>  
	<activation>  
		<jdk>1.7</jdk>  
	</activation>  
	<repositories>  
		<repository>  
			<id>nexus</id>  
			<name>local private nexus</name>  
			<url>http://maven.oschina.net/content/groups/public/</url>  
			<releases>  
				<enabled>true</enabled>  
			</releases>  
			<snapshots>  
				<enabled>false</enabled>  
			</snapshots>  
		</repository>  
	</repositories>  
	<pluginRepositories>  
		<pluginRepository>  
			<id>nexus</id>  
			<name>local private nexus</name>  
			<url>http://maven.oschina.net/content/groups/public/</url>  
			<releases>  
				<enabled>true</enabled>  
			</releases>  
			<snapshots>  
				<enabled>false</enabled>  
			</snapshots>  
		</pluginRepository>  
	</pluginRepositories>  
</profile>
```
