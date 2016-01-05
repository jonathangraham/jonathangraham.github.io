---
layout: post
title: Adding Local Jar Dependency in Maven
tagline:    
include_social: true
---
{% include JB/setup %}

I recently wrote a <a href= "https://github.com/jonathangraham/http_server">java http server</a>, which included specific routing so that it would pass the <a href="https://github.com/8thlight/cob_spec">8th Light cobspec</a> suite of tests. So that I could use my server to serve other apps, I wanted to pull it out as a separate jar. I could then include it as a dependency within my cobspec app, and any other web apps.

I had already built a clean interface, so separation of the code into two new projects, <a href="https://github.com/jonathangraham/java_server">java server</a> and <a href="https://github.com/jonathangraham/cobspec_app">cobspec app</a>, was fairly trivial. I used <a href="https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html">Maven</a> for the builds. The compiled server code, which holds an AppRouter interface, was successfully packaged into a jar using ```mvn package```, with all the associated server unit tests passing. The cobspec app implemented the AppRouter interface, and just needed to depend on the new server jar to function as before. Easy, right?

Maybe, but although there is plenty of information out there, in the form of docs, blogs, stackoverflows, etc, nothing I found actually gave me everything I needed. I have found this a lot with the java ecosystem, and a lot more than I did getting started with Ruby, Clojure and Scala. However, not having everything working straight away gives you a chance to understand what is really happening, and below is what I learned about adding a local jar as a dependency in a Maven project.  

With a Maven project, you can install the package into a local repository, for use as a dependency in other projects locally, using ```mvn install```. Running this put my server jar and pom files in ```Users/user/.m2/repository/com/jgraham/server/server/1.0-SNAPSHOT``` (the server Maven project was created with a groupID of com.jgraham.server, an artifactID of server, and version 1.0-SNAPSHOT). I assumed that I could then simply add a dependency to the server in my cobspec app pom.xml file:

      <dependency>
          <groupId>com.jgraham.server</groupId>
          <artifactId>server</artifactId>
          <version>1.0</version>
      </dependency>   

However, on running ```mvn package``` for the cobspec app, I got the error ```The POM for com.jgraham.server:server:jar:1.0 is missing, no dependency information available``` and the build failed.

Instead of installing the server to a local respoitory, I decided to ```deploy``` it for direct use in the cobspec app:

```mvn deploy:deploy-file -Durl=file:/<path_to_app>/app/lib/ -Dfile=server-1.0-SNAPSHOT.jar -DgroupId=com.jgraham.server -DartifactId=server -Dpackaging=jar -Dversion=1.0```

This put the jar and pom in a lib folder in my cobspec app.

To enable Maven to know where to find the server during the cobspec app build, I added the new lib repository to the POM file:

    <repositories>
        <repository>
            <id>lib</id>
            <url>file:${project.basedir}/lib</url>
        </repository>
    </repositories> 

With this in place, the cobspec app successfully built using ```mvn package```, with all the unit tests passing.

However, on running the jar (```java -cp target/cobspec-1.0-SNAPSHOT.jar com.jgraham.cobspec.Main```) I got the error ```Exception in thread "main" java.lang.NoClassDefFoundError: com/jgraham/server/Routers/iAppRouter```.

Although the server was available for the build, its classes were not put on the cobspec classpath for runtime.

To copy the dependencies into the classpath during the build, I used the ```maven-dependency-plugin``` by adding to the POM:

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>compile</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

This creates a lib folder within target, holding all the jars that cobspec depends on. These are still not available on the classpath, however, and I used the ```maven-jar-plugin``` to add the contents of the lib folder to the classpath:

      <plugin>
          <artifactId>maven-jar-plugin</artifactId>
          <configuration>
            <archive>
              <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
              </manifest>
            </archive>
          </configuration>
      </plugin>

After rebuilding with ```mvn package```, the cobspec jar can be run using ```java -cp target/cobspec-1.0-SNAPSHOT.jar com.jgraham.cobspec.Main```, which serves on ```localhost:5000``` by default.

By adding ```<mainClass>com.jgraham.cobspec.Main</mainClass>``` to the ```<manifest>``` the jar can be run directly with ```java -jar target/cobspec-1.0-SNAPSHOT.jar```, and everything works just as it should.

So, in retrospect it was easy to add a local jar as a dependency in a Maven project, even if the hours slipped away in the process!

You can find the full source code for the <a href="https://github.com/jonathangraham/java_server">java server</a> and <a href="https://github.com/jonathangraham/cobspec_app">cobspec app</a> on github.

