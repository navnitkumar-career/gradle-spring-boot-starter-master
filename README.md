This tutorial will walk you through creating a simple Spring Boot application integrated with Gradle. All of the code in the example is also included in this repository.

Install the Development Platform
--------------------------------
* Install [Java](http://www.oracle.com/technetwork/java/javase/overview/index.html)

Install the Build Tool
----------------------
[Gradle](https://gradle.org) is a tool that can be used to easily create repeatable builds. It will compile code, run tests, create deployment artifacts and much more.

Install it following the instructions here: https://gradle.org/install/

Create a directory for your project:

    mkdir gradle-spring-boot-starter
    cd gradle-spring-boot-starter

Initialize a new Gradle project
-------------------------------
Execute the following command to create a new Gradle project:

    gradle init

_Note that Gradle doesn't know how to do much on it's own, and will download lots of other jar files and plugins to execute various tasks._

This creates a few starter files & directories in your project:

    ├── build.gradle
    ├── gradle
    │   └── wrapper
    │       ├── gradle-wrapper.jar
    │       └── gradle-wrapper.properties
    ├── gradlew
    ├── gradlew.bat
    └── settings.gradle

`build.gradle` is the place that will describe how to compile your project, run tests and generate any required artifacts.

The `gradle` directory contains the [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html). The Gradle Wrapper is the preferred way of running Gradle, because it ensures that anyone using your project is using the exact same version of Gradle.

`settings.gradle` is a place to put various configuration settings used by the main `build.gradle` file. You'll notice that currently all it has in it is the name of the project

`gradlew` & `gradlew.bat` are the Gradle Wrapper runners for Unix & Windows respectively.

From now on, instead of typing `gradle <taskname>`, you'll run `./gradlew <taskname>`

Learn about Gradle Tasks
------------------------
Run this command to see what tasks are currently available with your project

    ./gradlew tasks

Notice that there are no tasks to actually build the project.

We can fix this by telling Gradle that this is going to be a java project. Add the following to the _top_ of your `build.gradle`:

    plugins {
        id 'java'
    }

You don't need to specify a version of this plugin, since it is considered a  [Core Plugin](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block)

Note that you may see other guides that use the `apply plugin: 'java'` syntax. This is the [legacy style](https://docs.gradle.org/current/userguide/plugins.html#sec:old_plugin_application) of applying plugins and can be used interchangeably with the newer syntax.

Now run `./gradlew tasks` again and notice the new tasks -- most notable `clean` and `build`


Build a Java artifact
---------------------
The deployable artifact we're going to be using is a [Jar file](https://docs.oracle.com/javase/tutorial/deployment/jar/basicsindex.html).

You can create this jar file by executing the following Gradle command:

    ./gradlew build

You'll now have a new directory called `build`, under which there is a `libs` directory containing your new jar file. Since you have no code yet, the jar file doesn't actually contain anything useful.

We can get rid of this directory by executing the following Gradle command:

    ./gradlew clean

From now on, we'll combine these two tasks to ensure we start with an empty build directory before building.

    ./gradlew clean build

Turn it into a Spring Boot project
----------------------------------
In order to quickly make a web service, we'll use [Spring Boot](https://projects.spring.io/spring-boot).

Tell Gradle that we're going to be using Spring Boot by adding the [Spring Boot Gradle Plugin](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/gradle-plugin/reference/html/#getting-started) to the `plugins` block of your `build.gradle` file.

        id 'org.springframework.boot' version '2.0.4.RELEASE'
        id 'io.spring.dependency-management' version '1.0.6.RELEASE'

If you run `./gradlew tasks` again, you'll notice that there is a new `bootRun` task. We'll use this to start the Spring Boot application in a few steps.

the `plugins` entry also tells Gradle to use the specified spring-boot version for any other dependencies that we add to the project.

Next, add the following to your `build.gradle` file:

    repositories {
        mavenCentral()
    }

    dependencies {
        compile 'org.springframework.boot:spring-boot-starter-web'
    }

The `repositories` section tells Gradle where to look in order to download any dependencies needed by the build process and/or your application. [Maven Central](https://search.maven.org) is a well known location that hosts many open source projects.

The `dependencies` section tells Gradle that we are going to add the specified dependency to the Java compilation classpath. The Maven infrastructure (which is leveraged by Gradle and many other build tools) is able to identify all artifacts by three items:
  * group id     - `org.springframework.boot`
  * artifact id  - `spring-boot-starter-web`
  * version      - Gradle resolves this automatically because of the `plugins` section from above.

All that is remaining to have a working Spring Boot application is to have an entry point for the application. This is done by creating a class with a `main` method. Create this class in `src/main/java/com/chikli/example/SpringBootApplicationExample.java`. By default, Gradle will look for source code under `src/main/java`:

    package com.chikli.example;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    @SpringBootApplication
    public class SpringBootApplicationExample {

        public static void main(String[] args) {
            SpringApplication.run(SpringBootApplicationExample.class, args);
        }
    }

The [`@SpringBootApplication` annotation](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html) will perform a bunch of sensible defaults based on the jars that it finds in your classpath.

You can now start the Spring Boot application by executing `./gradlew bootRun`. Spring will search your classpath for an application (i.e. a class with a `main` method and annotated with `@EnableAutoConfiguration` or `@SpringBootApplication`).

_Note: Gradle tasks have dependencies on other tasks, so executing `bootRun` for example will also execute `build`, thus, we really only need to execute the last task in the chain._

In the console output, you'll see that a Tomcat server will have started up on port 8080. The bad news is that your fancy new Spring Boot application doesn't do anything yet.

You can press `control-C` to kill the application.

Add a Web Service Endpoint
--------------------------
Our final step is to add a web service endpoint. The simplest way to do this is to just enhance our existing class to contain the Web Service functionality. In practice, you'd probably want to create a separate class to contain your controller.

Enhance the existing file so it looks like this:

    package com.chikli.example;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    @SpringBootApplication
    @RestController
    public class SpringBootApplicationExample {

        @RequestMapping("/cool-endpoint")
        public String coolWebServiceMethodName() {
           return "some cool web service data";
        }

        public static void main(String[] args) {
            SpringApplication.run(SpringBootApplicationExample.class, args);
        }
    }

The [`@RestController` annotation](https://docs.spring.io/spring/docs/current/javadoc-api/index.html?org/springframework/web/bind/annotation/RestController.html) tells spring that this class will have some methods that map to incoming requests.

The [`@RequestMapping` annotation](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html) simply maps an HTTP GET request to a java method. The content of the HTTP Response will be whatever the method returns.

Now execute `./gradlew bootRun` again and you'll see the mapping being created in the console output. Something like:

    2018-03-12 20:23:30.103  INFO 42096 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/cool-endpoint]}" onto public java.lang.String com.chikli.example.SpringBootApplicationExample.coolWebServiceMethodName()

Finally, you can open up your browser and go to http://localhost:8080/cool-endpoint .

You should now be in a great place to expand this application and conquer the world! Enjoy!
