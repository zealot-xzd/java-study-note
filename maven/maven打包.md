目标：

将依赖的第三方jar包打进去

方法：

maven-assembly-plugin

环境：

IDEA 2016.3

JDK 1.8

遇到的问题：

此处耗时2天时间，遇到过的坑：

1.修改完pom.xml后，不生效。

--改pom.xml后，代码不生效，是因为对IDEA工具不熟，在修改完xml后，需要点工具右下角的import changes或者直接点auto-import就可以一劳永逸了。

2.生成jar后，idea可以执行，但是java -jar无法执行，报错Exception in thread "main" java.lang.NoClassDefFoundError

如果修改pom.xml中的mainClass生效了，说不定是mainClass传入的不对，使用mvn exec:java -Dexec.mainClass="com.delon.main.Test"可以尝试main方法是否正确。

如果想用编译Test.java文件，可以使用mvn clean compile exec:java -Dexec.mainClass="com.delon.main.Test"

3.生成jar后，idea可以执行，java -jar也可以执行，但是缺少相关依赖，报错Exception in thread "main" java.lang.NoClassDefFoundError: okhttp3/RequestBody

参考如下解决方式即可。

 

maven构建jar包的步骤：

1.执行可执行的class，代码内需要有入口main方法

2.通过mvn package来构建jar包

3.使用java -jar test.jar来执行jar包

 

一、包含依赖jar包

 maven的pom.xml配置文件

复制代码
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.xxx.delon</groupId>
    <artifactId>bugly</artifactId>
    <version>1.0-SNAPSHOT</version>

    
    <dependencies>
　　　　<!-- 工程所需jar包引用开始 -->
        <!-- 单元测试 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.7</version>
            <scope>test</scope>
        </dependency>

        <!-- 代码所需依赖 -->
        <dependency>
            <groupId>com.squareup.okhttp3</groupId>
            <artifactId>okhttp</artifactId>
            <version>3.3.0</version>
        </dependency>

        <!-- 代码所需依赖 -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.2.2</version>
        </dependency>
        <!-- 工程所需jar包引用开始 -->
    </dependencies>


    <build>
        <plugins>

            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <appendAssemblyId>false</appendAssemblyId>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <!-- 此处指定main方法入口的class -->
                            <mainClass>com.xxx.uploadFile</mainClass>
                        </manifest>
                    </archive>
               ta </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>assembly</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

        </plugins>
    </build>

</project>
复制代码
生成jar包，会生成在target目录下

mvn package
解压缩bugly-1.0-SNAPSHOT.jar->META-INF->MANIFEST.MF

Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Created-By: Apache Maven
Built-By: delon
Build-Jdk: 1.8.0_112
Main-Class: com.xxx.uploadFile
执行：

# 运行jar
java -jar test.jar
也可以通过如下命令

mvn assembly:assembly
#跳过测试 
mvn -Dmaven.test.skip=true  assembly:assembly
注意：在执行这个命令之前，必须先配置Maven的环境变量，检查是否配置可通过命令： mvn -version

如果上面的命令成功执行，那么在项目路径的target文件下就会有两个jar文件，一个是有jar包依赖的，一个是没jar包依赖的。

二、不包含依赖jar包

如果不想包含依赖的jar包，可以把<build>里面的代码替换成如下code：

复制代码
            <!-- 打包jar文件时，配置manifest文件，加入lib包的jar依赖 -->  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-jar-plugin</artifactId>  
                <configuration>  
                    <archive>  
                        <manifest>  
                            <addClasspath>true</addClasspath>  
                            <classpathPrefix>lib/</classpathPrefix>  
                            <mainClass>com.xxx.uploadFile</mainClass>  
                        </manifest>  
                    </archive>  
                </configuration>  
            </plugin>  
            <!-- 拷贝依赖的jar包到lib目录 -->  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-dependency-plugin</artifactId>  
                <executions>  
                    <execution>  
                        <id>copy</id>  
                        <phase>package</phase>  
                        <goals>  
                            <goal>copy-dependencies</goal>  
                        </goals>  
                        <configuration>  
                            <outputDirectory>  
                                ${project.build.directory}/lib  
                            </outputDirectory>  
                        </configuration>  
                    </execution>  
                </executions>  
            </plugin>  
复制代码
 

三、只包含部分依赖jar包

 如果想只包含部分依赖jar包

比如说，想做一个工具jar包，依赖公共jar和自己本地jar包，本地jar包需要解压成class打到jar包内，而依赖的公共jar包则不需要。

剔除公共jar包 可以用<scope>

<scope>的值的含义：
compile，缺省值，适用于所有阶段，会随着项目一起发布。
provided，类似compile，期望JDK、容器或使用者会提供这个依赖。如servlet.jar。
runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段。
test，只在测试时使用，用于编译和运行测试代码。不会随项目发布。
system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。

编译的时候采用 compile

复制代码
<dependency>
     <groupId>log4j</groupId>
     <artifactId>log4j</artifactId>
     <version>1.2.17</version>
     <scope>complie</scope>
     <optional>true</optional>
</dependency>
复制代码
在用package打包的时候，改成test，生成的jar包里就不会有该jar包的类了。

复制代码
<dependency>
     <groupId>log4j</groupId>
     <artifactId>log4j</artifactId>
     <version>1.2.17</version>
     <scope>test</scope>
     <optional>true</optional>
</dependency>
复制代码
build配置项，mainClass为空因为不是可执行jar。

复制代码
<build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <mainClass></mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
复制代码
