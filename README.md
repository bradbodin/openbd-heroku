<<<<<<< HEAD
# Deploying an OpenBD (Open BlueDragon) Web Application on Heroku via Jetty Runner

### Preface
OpenBD is a lightweight ColdFusion compatible application server. It is designed to run
on any J2EE server. Heroku now supports deploying and running Java apps, but it uses a 
deployment model that is somewhat different from what most OpenBD developers expect. 
This document presents a workable method for deploying OpenBD on Heroku on top of Jetty.
There are a few ways we could do this, but we are opting for a very simple approach that
doesn't require you to actually use maven at all, we just give Heroku the stuff it needs 
to do it on their side.

## Prerequisites

* [Java VM](http://www.java.com/en/download/index.jsp)
* [Heroku Toolbelt](https://toolbelt.heroku.com/) (includes Git and Foreman)
* [OpenBD Desktop](http://openbd.org/downloads/)
* Basic knowledge of Git is assumed. If you need help, go [here](http://githowto.com/)

## The Easy Way

Clone this repo. 

    $ git clone git@github.com:heathprovost/openbd-heroku.git


Download OpenBD Desktop and point it at the project folder and check
off "Provision Web App with OpenBD latest Engine". You can also check off "Enable Admin Console"
if you want that (you generally do). Now hit "Start Web App" and your server should be running
at http://localhost:8080/. Stop your application before proceeding further.

## Deploy to Heroku

The first thing to do is commit your project into git if you haven't already.

    $ git init
    $ git add .
    $ git commit -m "Ready to deploy"

Create the Heroku App (using the Cedar stack):

    $ heroku create --stack cedar my-app-name
    Creating my-app-name... done, stack is cedar
    http://my-app-name.herokuapp.com/ | git@heroku.com:my-app-name.git
    Git remote heroku added

Deploy your code:

    $ git push heroku master
    Counting objects: xxx, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (xxx/xxx), done.
    Writing objects: 100% (xxx/xxx), xxx.xx KiB, done.
    Total xxx (delta xx), reused xxx (delta xx)

    -----> Heroku receiving push
    -----> Java app detected
    -----> Installing Maven 3.0.3..... done
    -----> Installing settings.xml..... done
    -----> executing .maven/bin/mvn -B -Duser.home=/tmp/build_xxx -s .m2/settings.xml -DskipTests=true clean install
           [INFO] Scanning for projects...
           [INFO]                                                                         
           [INFO] ------------------------------------------------------------------------
           [INFO] Building xxx 0.1.0.BUILD-SNAPSHOT
           [INFO] ------------------------------------------------------------------------
           ...
           [INFO] ------------------------------------------------------------------------
           [INFO] BUILD SUCCESS
           [INFO] ------------------------------------------------------------------------
           [INFO] Total time: xx.xxxs
           [INFO] Finished at: Mon Jan 1 00:00:00 UTC 2013
           [INFO] Final Memory: xxM/xxxM
           [INFO] ------------------------------------------------------------------------
    -----> Discovering process types
           Procfile declares types -> web
    -----> Compiled slug size is xx.xMB
    -----> Launching... done, v1
           http://my-app-name.herokuapp.com deployed to Heroku


Your app should be up and running on Heroku. Open it in your browser with:

    $ heroku open

## The Hard Way

If you prefer to do it yourself from scratch. You will need Maven 3 installed to do this.
If you run on OSX you probably already have it. Otherwise you can get it [here](http://maven.apache.org/download.cgi).

Heroku uses Maven for doing Java deployments. In order to do this, it looks for a file
called pom.xml in the root of your project. The approach we use here is to have Heroku
use maven to copy Jetty Runner into your project in order to bootstrap OpenBD. Jetty 
Runner is essentially a generalized, pre-configured Jetty Server packaged in a way that
allows you to simply and easily launch Jetty and run a specified war file, in this case
OpenBD itself. So if you do something like this: 

    $ java -jar jetty-runner.jar application.war

Jetty Runner will launch a Jetty instance with the given war deployed to it.

## Getting Started

The first thing you want to do is use Maven to initialize a new project:

    $ mvn archetype:generate -DarchetypeArtifactId=maven-archetype-webapp
    ...
    [INFO] Generating project in Interactive mode
    Define value for property 'groupId': : com.example
    Define value for property 'artifactId': : openbd-heroku
    
(you can pick any groupId or artifactId). You now have a complete Java web app in the `openbd-heroku` directory.
The first thing you want to do is delete the boilerplate code that maven generated in src/main/webapp - you want
this folder to be empty before proceeding.

## Add Jetty Runner to your pom.xml file

The next step is to modify the generated pom.xml file to copy Jetty Runner into your project. As of right now
OpenBD is not published as a maven artifact, so we won't be able to use maven to download and configure OpenBD,
but hopefully in the future this will be possible. For now, add the following to the build section of your pom.xml file:

    <build>
	    <plugins>
	      <plugin>
	        <groupId>org.apache.maven.plugins</groupId>
	        <artifactId>maven-dependency-plugin</artifactId>
	        <version>2.3</version>
	        <executions>
	          <execution>
	            <phase>package</phase>
	            <goals>
	              <goal>copy</goal>
	            </goals>
	            <configuration>
	              <artifactItems>
	                <artifactItem>
	                  <groupId>org.mortbay.jetty</groupId>
	                  <artifactId>jetty-runner</artifactId>
	                  <version>7.5.4.v20111024</version>
	                  <destFileName>jetty-runner.jar</destFileName>
	                </artifactItem>
	              </artifactItems>
	            </configuration>
	            </execution>
	        </executions>
	      </plugin>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>2.4.1</version>
          <configuration>
            <filesets>
              <fileset>
                <directory>target/</directory>
                <includes>
                  <include>**/*</include>
                </includes>
                <excludes>
                  <exclude>dependency/*.jar</exclude>
                  <exclude>*.war</exclude>
                </excludes>
                <followSymlinks>false</followSymlinks>
              </fileset>
            </filesets>
            <excludeDefaultDirectories>true</excludeDefaultDirectories>
          </configuration>
          <executions>
            <execution>
              <id>auto-clean</id>
              <phase>install</phase>
              <goals>
                <goal>clean</goal>
              </goals>
            </execution>
          </executions>
        </plugin>      
      </plugins>
    </build>

Note: There is some additional customization in the pom.xml file in this repo to make it easier to use for developers who are not going to use maven at all for their dependency management, and just want a way to
deploy to Heroku. Primarily these modifications collapse the directory structure so that maven sees the
root of the project folder as src/main/webapp, with some exclusions to make the build work correctly. If 
you actually plan on using maven, I don't recommend doing any of that - just do something like the above.

## Add a Procfile

Heroku uses Foreman to launch application. You will need to create a Procfile to the root of your project in
order to tell Heroku how to run your application. The contents of this file should be:

		web: java $JAVA_OPTS -jar target/dependency/jetty-runner.jar --port $PORT target/*.war

## Add a .gitignore file

Maven will create a folder called target that holds all your build dependencies and whatnot. You generally
don't want this in revision control, so create a .gitignore file in the root of your project containing the
following

		target

## Add OpenBD and Build your App

Since OpenBD itself is not available as a maven artifact, you will have to add it manually. To do this, download and unzip the OpenBD Standard J2EE WAR file (it has a war extension, but it is in fact a zip file). After
decompressing it, you will see something like this:

		openbd
		-- bluedragon
		-- manual
		-- WEB-INF
		-- index.cfm

You will want to copy the contents of the openbd folder to src/main/webapp. You should also either delete the
manual folder, or add it to your .gitignore so that is doesn't get deployed to heroku later on.

To build your application simply run:

    $ mvn package

And then run your app using the java command:

    $ java -jar target/dependency/jetty-runner.jar target/*.war

That's it. Your application should be running locally on port 8080. 

Alternatively, you can download and run OpenBD Desktop in order to run your application for testing locally. All you have to do is point it to your webapp folder. I would actually recommend this method, as if you use OpenBD Desktop you no longer need to use maven or jetty-runner at all locally, it is only needed to allow running on Heroku. Once you have the basic skeleton generated, you can just use it as a project template for each app you
build. Heroku needs your pom.xml and Procfile in order to deploy and run your app, but you don't actually need them for local development.

# Deploy your Application to Heroku

Read the "easy" instructions at the top of this file.
=======
openbd-heroku
=============
>>>>>>> b8f947423ed71822a027bc56353c08d7492940b3
