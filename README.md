hadoop-unit
====================

Hadoop-Unit is a project which allow testing projects which need hadoop ecosysteme like kafka, solr, hdfs, hive, hbase, ...

Moreover, it provide a standalone component which can be run locally and which simulate a hadoop cluster.

hadoop-unit-standalone:
[![Maven Central - hadoop-unit-standalone](https://maven-badges.herokuapp.com/maven-central/fr.jetoile.hadoop/hadoop-unit-standalone/badge.svg)](https://maven-badges.herokuapp.com/maven-central/fr.jetoile.hadoop/hadoop-unit-standalone)

[![Build Status](https://travis-ci.org/jetoile/hadoop-unit.svg?branch=master)](https://travis-ci.org/jetoile/hadoop-unit)

#Build

For windows users, you need to download a hadoop distribution, to unzip it and to define the system environment variable `HADOOP_HOME`. You can also define the path into files `hadoop-unit-default.properties` (warning: there are a lot...).

Hadoop Unit is using java 8 so set up your environment with it.

To build, launch the command:
```bash
mvn package
```

#Compatibility matrix

| Hadoop Unit version  | Hadoop mini cluster version | HDP version |
| ------------- | ------------- | ------------- |
| 2.2 | 0.1.11 | HDP 2.5.3.0 |
| 2.1 | 0.1.11 | HDP 2.5.3.0 |
| 2.0 | 0.1.9 | HDP 2.5.3.0 |
| 1.5 | 0.1.8 | HDP 2.5.0.0 |
| 1.4 | 0.1.7 | HDP 2.4.2.0 |
| 1.3 | 0.1.6 | HDP 2.4.0.0 |
| 1.2 | 0.1.5 | HDP 2.4.0.0 |


#Usage

When Hadoop Unit is started, it should display stuff like that:
```bash
           ______  __      _________                         _____  __      __________
           ___  / / /_____ ______  /___________________      __  / / /_________(_)_  /_ 1.3
           __  /_/ /_  __ `/  __  /_  __ \  __ \__  __ \     _  / / /__  __ \_  /_  __/
           _  __  / / /_/ // /_/ / / /_/ / /_/ /_  /_/ /     / /_/ / _  / / /  / / /_
           /_/ /_/  \__,_/ \__,_/  \____/\____/_  .___/      \____/  /_/ /_//_/  \__/
                                               /_/
 - ZOOKEEPER [host:127.0.0.1, port:22010]
 - HDFS [port:20112]
 - HIVEMETA [port:20102]
 - HIVESERVER2 [port:20103]
 - KAFKA [host:127.0.0.1, port:20111]
 - HBASE [port:25111]
 - SOLRCLOUD [zh:127.0.0.1:22010, port:8983, collection:collection1]

```

The available components are:
* HDFS
* ZOOKEEPER
* HIVEMETA
* HIVESERVER2
* SOLR
* SOLRCLOUD
* OOZIE
* KAFKA
* HBASE
* MONGODB
* CASSANDRA
* ELASTICSEARCH
* NEO4J
* KNOX
* ALLUXIO

##Integration testing (will start each component present into classpath)
With maven, add dependencies of components which are needed

Sample:
```xml
<dependency>
    <groupId>fr.jetoile.hadoop</groupId>
    <artifactId>hadoop-unit-hdfs</artifactId>
    <version>2.1</version>
    <scope>test</scope>
</dependency>
```

In test do:
```java
@BeforeClass
public static void setup() {
    HadoopBootstrap.INSTANCE.startAll();
}

@AfterClass
public static void tearDown() {
    HadoopBootstrap.INSTANCE.stopAll();
}
```

##Integration testing v2 (with specific component)
With maven, add dependencies of components which are needed

Sample:
```xml
<dependency>
    <groupId>fr.jetoile.hadoop</groupId>
    <artifactId>hadoop-unit-hdfs</artifactId>
    <version>2.1</version>
    <scope>test</scope>
</dependency>
```

In test do:
```java
@BeforeClass
public static void setup() throws NotFoundServiceException {
    HadoopBootstrap.INSTANCE
        .start(Component.ZOOKEEPER)
        .start(Component.HDFS)
        .start(Component.HIVEMETA)
        .start(Component.HIVESERVER2)
        .startAll();
}

@AfterClass
public static void tearDown() throws NotFoundServiceException {
    HadoopBootstrap.INSTANCE
        .stopAll();
}
```

##Standalone mode

Unzip `hadoop-unit-standalone-<version>.tar.gz`
Change `conf/hadoop-unit-default.properties`
Change `conf/hadoop.properties`

Start in fg with:
```bash
./bin/hadoop-unit-standalone console
```

Start in bg with:
```bash
./bin/hadoop-unit-standalone start
```

Stop with:
```bash
./bin/hadoop-unit-standalone stop
```

Because of the use of aether, `hadoop-unit-default.properties` has to know where your maven local repo is. In the same way, if a proxy manager like nexus or artifactory is used, it has to be indicated :
```bash
maven.central.repo=https://repo.maven.apache.org/maven2/
maven.local.repo=/home/khanh/.m2/repository
```


##Shell Usage
Hadoop-unit can be used with common tools such as:

* hbase shell
* kafka-console command
* hdfs command
* hive shell

###Kafka-console command

* Download and unzip kafka
* From directory `KAFKA_HOME/bin` (or `KAFKA_HOME/bin/windows` for windows), execute command: 
```bash
kafka-console-consumer --zookeeper localhost:22010 --topic topic
```

###HBase Shell

* Download and unzip HBase
* set variable `HBASE_HOME`
* edit file `HBASE_HOME/conf/hbase-site.xml`:
```xml
<configuration>
	<property>
		<name>hbase.zookeeper.quorum</name>
		<value>127.0.0.1:22010</value>
	</property>
	<property>
		<name>zookeeper.znode.parent</name>
		<value>/hbase-unsecure</value>
	</property>
</configuration>
```

* From directory `HBASE_HOME/bin`, execute command: 
```bash
hbase shell
```

###HDFS command

* From directory `HADOOP_HOME/bin`, execute command: 
```bash
hdfs dfs -ls hdfs://localhost:20112/
```
###Hive Shell

* Download and unzip Hive
* edit file `HIVE_HOME/conf/hive-site.xml`:
```xml
<configuration>
	<property>
		<name>hive.metastore.uris</name>
		<value>thrift://127.0.0.1:20102</value>
	</property>
</configuration>
```
* From directory `HIVE_HOME/bin`, execute command: 
```bash
hive
```

###Alluxio Shell

* Download and unzip alluxio
* edit file `ALLUXIO_HOME/conf/alluxio-env.sh`:
```properties
ALLUXIO_MASTER_HOSTNAME=localhost
```
* edit file `ALLUXIO_HOME/conf/alluxio-site.properties`:
```properties
alluxio.master.port=14001
```
* From directory `ALLUXIO_HOME/bin`, execute command: 
```bash
./alluxio fs ls <path>
```

#Sample
See `hadoop-unit-standalone/src/test/java/fr/jetoile/hadoopunit/integrationtest`

See `sample`

#Maven Plugin usage
A maven plugin is provided for integration test only.

##Embedded mode

To use it, add into the pom project stuff like that:
```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>

     <dependency>
        <groupId>fr.jetoile.hadoop</groupId>
        <artifactId>hadoop-unit-client-hdfs</artifactId>
        <version>2.1</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>fr.jetoile.hadoop</groupId>
        <artifactId>hadoop-unit-client-hive</artifactId>
        <version>2.1</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>fr.jetoile.hadoop</groupId>
        <artifactId>hadoop-unit-client-spark</artifactId>
        <version>2.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>**/*IntegrationTest.java</exclude>
                </excludes>
            </configuration>
            <executions>
                <execution>
                    <id>integration-test</id>
                    <goals>
                        <goal>test</goal>
                    </goals>
                    <phase>integration-test</phase>
                    <configuration>
                        <excludes>
                            <exclude>none</exclude>
                        </excludes>
                        <includes>
                            <include>**/*IntegrationTest.java</include>
                        </includes>
                    </configuration>
                </execution>
            </executions>
        </plugin>

        <plugin>
            <artifactId>hadoop-unit-maven-plugin</artifactId>
            <groupId>fr.jetoile.hadoop</groupId>
            <version>${hadoop-unit.version}</version>
            <executions>
                <execution>
                    <id>start</id>
                    <goals>
                        <goal>embedded-start</goal>
                    </goals>
                    <phase>pre-integration-test</phase>
                </execution>
            </executions>
            <configuration>
                <components>
                    <componentArtifact implementation="fr.jetoile.hadoopunit.ComponentArtifact">
                        <componentName>HDFS</componentName>
                        <artifact>fr.jetoile.hadoop:hadoop-unit-hdfs:2.1</artifact>
                    </componentArtifact>
                    <componentArtifact implementation="fr.jetoile.hadoopunit.ComponentArtifact">
                        <componentName>ZOOKEEPER</componentName>
                        <artifact>fr.jetoile.hadoop:hadoop-unit-zookeeper:2.1</artifact>
                    </componentArtifact>
                    <componentArtifact implementation="fr.jetoile.hadoopunit.ComponentArtifact">
                        <componentName>HIVEMETA</componentName>
                        <artifact>fr.jetoile.hadoop:hadoop-unit-hive:2.1</artifact>
                    </componentArtifact>
                    <componentArtifact implementation="fr.jetoile.hadoopunit.ComponentArtifact">
                        <componentName>HIVESERVER2</componentName>
                        <artifact>fr.jetoile.hadoop:hadoop-unit-hive:2.1</artifact>
                    </componentArtifact>
                    <componentArtifact implementation="fr.jetoile.hadoopunit.ComponentArtifact">
                        <componentName>SOLRCLOUD</componentName>
                        <artifact>fr.jetoile.hadoop:hadoop-unit-solrcloud:${hadoop-unit.version}</artifact>
                        <properties>
                            <solr.dir>file://${project.basedir}/src/test/resources/solr</solr.dir>
                        </properties>
                    </componentArtifact>
                </components>

            </configuration>

        </plugin>

    </plugins>
</build>
```
Values can be:
* HDFS
* ZOOKEEPER
* HIVEMETA
* HIVESERVER2
* SOLR
* SOLRCLOUD
* OOZIE
* KAFKA
* HBASE
* MONGODB
* CASSANDRA
* ELASTICSEARCH
* NEO4J
* KNOX
* ALLUXIO

It is also possible to override configurations with a list of `properties` which accept a map (ie. `<key>value</key>` and where `key` is a property from the file `hadoop-unit-default.properties`). 

For solrcloud, it is mandatory to indicate where is the solr config:

```xml
    <componentArtifact implementation="fr.jetoile.hadoopunit.ComponentArtifact">
        <componentName>SOLRCLOUD</componentName>
        <artifact>fr.jetoile.hadoop:hadoop-unit-solrcloud:${hadoop-unit.version}</artifact>
        <properties>
            <solr.dir>file://${project.basedir}/src/test/resources/solr</solr.dir>
        </properties>
    </componentArtifact>
```

Here is a sample integration test:
```java
public class HdfsBootstrapIntegrationTest {

    static private Configuration configuration;


    @BeforeClass
    public static void setup() throws BootstrapException {
        try {
            configuration = new PropertiesConfiguration("hadoop-unit-default.properties");
        } catch (ConfigurationException e) {
            throw new BootstrapException("bad config", e);
        }
    }


    @Test
    public void hdfsShouldStart() throws Exception {

        FileSystem hdfsFsHandle = HdfsUtils.INSTANCE.getFileSystem();


        FSDataOutputStream writer = hdfsFsHandle.create(new Path(configuration.getString(HadoopUnitConfig.HDFS_TEST_FILE_KEY)));
        writer.writeUTF(configuration.getString(HadoopUnitConfig.HDFS_TEST_STRING_KEY));
        writer.close();

        // Read the file and compare to test string
        FSDataInputStream reader = hdfsFsHandle.open(new Path(configuration.getString(HadoopUnitConfig.HDFS_TEST_FILE_KEY)));
        assertEquals(reader.readUTF(), configuration.getString(HadoopUnitConfig.HDFS_TEST_STRING_KEY));
        reader.close();
        hdfsFsHandle.close();

        URL url = new URL(
                String.format( "http://localhost:%s/webhdfs/v1?op=GETHOMEDIRECTORY&user.name=guest",
                        configuration.getInt( HadoopUnitConfig.HDFS_NAMENODE_HTTP_PORT_KEY ) ) );
        URLConnection connection = url.openConnection();
        connection.setRequestProperty( "Accept-Charset", "UTF-8" );
        BufferedReader response = new BufferedReader( new InputStreamReader( connection.getInputStream() ) );
        String line = response.readLine();
        response.close();
        assertThat("{\"Path\":\"/user/guest\"}").isEqualTo(line);
    }
}
```

##Remote mode
This plugin start/stop a remote local hadoop-unit-standalone.

To use it, add into the pom project stuff like that:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <excludes>
            <exclude>**/*IntegrationTest.java</exclude>
        </excludes>
    </configuration>
    <executions>
        <execution>
            <id>integration-test</id>
            <goals>
                <goal>test</goal>
            </goals>
            <phase>integration-test</phase>
            <configuration>
                <excludes>
                    <exclude>none</exclude>
                </excludes>
                <includes>
                    <include>**/*IntegrationTest.java</include>
                </includes>
            </configuration>
        </execution>
    </executions>
</plugin>

<plugin>
    <artifactId>hadoop-unit-maven-plugin</artifactId>
    <groupId>fr.jetoile.hadoop</groupId>
    <version>2.1</version>
    <executions>
        <execution>
            <id>start</id>
            <goals>
                <goal>start</goal>
            </goals>
            <phase>pre-integration-test</phase>
        </execution>
    </executions>
    <configuration>
        <hadoopUnitPath>/home/khanh/tools/hadoop-unit-standalone</hadoopUnitPath>
        <exec>./hadoop-unit-standalone</exec>
        <values>
            <value>ZOOKEEPER</value>
            <value>HDFS</value>
            <value>HIVEMETA</value>
            <value>HIVESERVER2</value>            
        </values>
        <outputFile>/tmp/toto.txt</outputFile>
    </configuration>

</plugin>

<plugin>
    <artifactId>hadoop-unit-maven-plugin</artifactId>
    <groupId>fr.jetoile.hadoop</groupId>
    <version>2.1</version>
    <executions>
        <execution>
            <id>stop</id>
            <goals>
                <goal>stop</goal>
            </goals>
            <phase>post-integration-test</phase>
        </execution>
    </executions>
    <configuration>
        <hadoopUnitPath>/home/khanh/tools/hadoop-unit-standalone</hadoopUnitPath>
        <exec>./hadoop-unit-standalone</exec>
        <outputFile>/tmp/toto.txt</outputFile>
    </configuration>

</plugin>
```
Values can be:
* HDFS
* ZOOKEEPER
* HIVEMETA
* HIVESERVER2
* SOLR
* SOLRCLOUD
* OOZIE
* KAFKA
* HBASE
* MONGODB
* CASSANDRA
* ELASTICSEARCH
* NEO4J
* KNOX
* ALLUXIO

hadoopUnitPath is not mandatory but system enviroment variable HADOOP_UNIT_HOME must be defined. 

exec variable is optional.

If both are set, HADOOP_UNIT_HOME override hadoopUnitPath.

Warning: This plugin will modify hadoop.properties and delete hadoop unit logs.

Here is a sample integration test:
```java
public class HdfsBootstrapIntegrationTest {

    static private Configuration configuration;


    @BeforeClass
    public static void setup() throws BootstrapException {
        try {
            configuration = new PropertiesConfiguration("hadoop-unit-default.properties");
        } catch (ConfigurationException e) {
            throw new BootstrapException("bad config", e);
        }
    }


    @Test
    public void hdfsShouldStart() throws Exception {

        FileSystem hdfsFsHandle = HdfsUtils.INSTANCE.getFileSystem();


        FSDataOutputStream writer = hdfsFsHandle.create(new Path(configuration.getString(HadoopUnitConfig.HDFS_TEST_FILE_KEY)));
        writer.writeUTF(configuration.getString(HadoopUnitConfig.HDFS_TEST_STRING_KEY));
        writer.close();

        // Read the file and compare to test string
        FSDataInputStream reader = hdfsFsHandle.open(new Path(configuration.getString(HadoopUnitConfig.HDFS_TEST_FILE_KEY)));
        assertEquals(reader.readUTF(), configuration.getString(HadoopUnitConfig.HDFS_TEST_STRING_KEY));
        reader.close();
        hdfsFsHandle.close();

        URL url = new URL(
                String.format( "http://localhost:%s/webhdfs/v1?op=GETHOMEDIRECTORY&user.name=guest",
                        configuration.getInt( HadoopUnitConfig.HDFS_NAMENODE_HTTP_PORT_KEY ) ) );
        URLConnection connection = url.openConnection();
        connection.setRequestProperty( "Accept-Charset", "UTF-8" );
        BufferedReader response = new BufferedReader( new InputStreamReader( connection.getInputStream() ) );
        String line = response.readLine();
        response.close();
        assertThat("{\"Path\":\"/user/guest\"}").isEqualTo(line);
    }
}
```

#Component available

* SolrCloud 6.1.0
* Kafka 
* Hive (metastore and server2)
* Hdfs
* Zookeeper
* Oozie (WIP)
* HBase
* Knox
* MongoDB
* Cassandra 3.4
* ElasticSearch 5.0-alpha4
* Neo4j 3.0.3
* Alluxio 1.4.0

Built on:
* [hadoop-mini-cluster-0.1.11](https://github.com/sakserv/hadoop-mini-clusters) (aka. HDP 2.5.3.0)
* [achilles-embedded-4.2.0](https://github.com/doanduyhai/Achilles)
* [maven aether](https://github.com/apache/maven-resolver/)

Use: 
* download and unzip hadoop
* [for oozie only] download and unzip oozie (http://s3.amazonaws.com/public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.3.4.0/tars/oozie-4.2.0.2.3.4.0-3485-distro.tar.gz)
* edit `hadoop-unit-default.properties` and indicate `HADOOP_HOME` or set your `HADOOP_HOME` environment variable
* edit `hadoop-unit-default.properties` and indicate `oozie.sharelib.path`

Issues:
* oozie does not work on windows 7 (see http://stackoverflow.com/questions/25790319/getting-access-denied-error-while-running-hadoop-2-3-mapreduce-jobs-in-windows-7)
* integrate phoenix
* can only manage one solr collection
* better docs ;)

# License

This software is licensed under the Apache License, version 2 ("ALv2"), quoted below.

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License. You may obtain a copy of
the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
License for the specific language governing permissions and limitations under
the License.
