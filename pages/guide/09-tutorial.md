---
layout: guide
group: guide
title: Tutorial - GeoTwitter
---

In this tutorial, we will build a JSON API web service called GeoTwitter. GeoTwitter is a backend for an app that shares your location with your followers.

In this tutorial you will learn:

- How to standup an Elide instance using Dropwizard
- How to enable user authentication in Elide
- How to use Elide annotations to implement user authorization


### Overview

GeoTwitter works a lot like Twitter.

A user can:

- Create an account
- Update their location
- Modify their information
- Follow other users
- Find the location of other users

A user cannot:

- Modify another user's information

We will build GeoTwitter using the following technologies:

- [Elide](http://elide.io/)
- [Hibernate](http://hibernate.org/)
- [MySQL](https://www.mysql.com/)
- [Dropwizard](http://www.dropwizard.io/)


### Bootstrap

Throughout our tutorial, we will use IntelliJ. Let's get started by launching IntelliJ and select "Create New Project":
![Welcome to IntelliJ IDEA]({{ site.url }}/assets/screenshots/welcome-to-intellij-idea.png)

Select "Maven" and click "Next":
![IntelliJ New Maven Project]({{ site.url }}/assets/screenshots/intellij-new-maven-project.png)

For GroupId, we will use: `io.elide.example` and for ArtifactId: `geotwitter`. Click "Next":
![Maven GroupId and ArtifactId]({{ site.url }}/assets/screenshots/maven-group-artifact-id.png)

Let's use `GeoTwitter` for the "Project name". Click "Finish":
![Maven GroupId and ArtifactId]({{ site.url }}/assets/screenshots/maven-project-name.png)


### Dependencies

We will be adding the dependencies for Dropwizard, Elide and MySQL. The `dropwizard-elide` dependency provides a Dropwizard bundle which makes working with Dropwizard a little nicer. Add the following dependencies to the pom.xml:

```xml
<dependencies>
    <!-- Dropwizard -->
    <dependency>
        <groupId>io.dropwizard</groupId>
        <artifactId>dropwizard-core</artifactId>
        <version>0.9.2</version>
    </dependency>
    <dependency>
        <groupId>io.dropwizard</groupId>
        <artifactId>dropwizard-db</artifactId>
        <version>0.9.2</version>
    </dependency>

    <!-- Elide -->
    <dependency>
        <groupId>com.yahoo.elide</groupId>
        <artifactId>dropwizard-elide</artifactId>
        <version>RELEASE</version>
    </dependency>

    <!-- MySQL -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.38</version>
    </dependency>
</dependencies>
```

Let's also configure the maven compiler plugin to use Java 8. Add this after the dependencies in the `pom.xml`:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.5.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```


### Dropwizard Configuration

The next step is to create the Dropwizard configuration YAML. This configuration file enables us to configure our application. It will also describe how to connect to the database. So let's create the file `src/main/resources/geotwitter.yml` with the following contents:

```yaml
database:
  driverClass: com.mysql.jdbc.Driver
  url: jdbc:mysql://localhost:3306/geotwitter
  user: geotwitter
  password: geotwitter123

  properties:
    hibernate.current_session_context_class: thread
    hibernate.hbm2ddl.auto: create
```

Here, we've configured our app to connect to a MySQL instance running on port 3306. The name of the database is `geotwitter`, and the user that will connect to the database is also `geotwitter` and it will connect using the password `geotwitter123`.

**At this point, you should stand up a local MySQL instance, create the database `geotwitter` and grant the user `geotwitter` full access to the database.** We logged in to our MySQL instance as root and ran the following commands:

```
mysql> CREATE DATABASE `geotwitter`;
Query OK, 1 row affected (0.00 sec)

mysql> GRANT ALL ON `geotwitter`.* TO 'geotwitter'@'localhost' IDENTIFIED BY 'geotwitter123';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

Now, we need to create the class that will read our configuration YAML. Let's start by creating the Java package `io.elide.geotwitter.application`. Add a Java class to the package called `GeotwitterConfiguration`. Our `GeotwitterConfiguration` class extends Dropwizard's `Configuration` class and looks like this:

```java
public class GeotwitterConfiguration extends Configuration {
    private DataSourceFactory database = new DataSourceFactory();

    @JsonProperty("database")
    public DataSourceFactory getDatabase() {
        return database;
    }

    @JsonProperty("database")
    public void setDatabase(DataSourceFactory database) {
        this.database = database;
    }
}
```

This configuration class basically instantiates a DataSourceFactory that reads its configuration from geotwitter.yml.


### Dropwizard Application

Let's create the application class. Add a new Java class to the `application` package called `GeotwitterApplication`. This class extends Dropwizard's `Application` class and is parameterized by our configuration class. Let's also add a main method that instantiates the class and calls the run method with the args:

```java
public class GeotwitterApplication extends Application<GeotwitterConfiguration> {

    @Override
    public void run(GeotwitterConfiguration geotwitterConfiguration, Environment environment) throws Exception {

    }

    public static void main(String[] args) throws Exception {
        new GeotwitterApplication().run(args);
    }
}
```

Here's what our project looks like so far:
![GeoTwitter Application and Configuration]({{ site.url }}/assets/screenshots/geotwitter-app-and-config.png)

If you run `GeotwitterApplication`, you can see a nice Dropwizard help message:

```
usage: java -jar project.jar [-h] [-v] {server,check} ...

positional arguments:
  {server,check}         available commands

optional arguments:
  -h, --help             show this help message and exit
  -v, --version          show the application version and exit
```

To start our app, we need to pass it two arguments. We need to tell it to run the `server`, and we need to tell Dropwizard which configuration file to use. Edit your `GeotwitterApplication` run configuration and set `Program arguments` to `server src/main/resources/geotwitter.yml`:
![IntelliJ Run Configuration]({{ site.url }}/assets/screenshots/intellij-run-configuration.png)

Now, if you run your application again, the server will start up and will listen on port 8080. Verify that the application launches successfully and then stop it because it doesn't do anything yet.


### User Model

The next step 
