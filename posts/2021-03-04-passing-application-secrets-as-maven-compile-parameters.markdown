---
title: Passing Application Secrets as Maven Compile Parameters
---

## Intro

When you have a project that is stored on a public repo, you have to have a process to 
not accidentally commit and push the secrets of your application (**private keys**, **API keys**, etc)
and have them visible to the public eye, unless you want to be exploited for good.

## Setup

One solution would be to pass such information to the application at compile/build time, 
utilizing code or configuration of tools you use in your project.

Let say we have a **Maven** **Java** project which uses an **API Key** for 
authenticating/authorizing access to a **Google Service**, such as **Directions API**. 
I have such a project at [dist-sys-server-java](https://github.com/steve-papadogiannis/dist-sys-server-java)
which gave me the idea of the topic of this post.

## Secrets

So our application uses two properties, `api.key` and `test.api.key` to hold the values of 
two different keys, one for runtime and the other for testing purposes.

Let see how the values can be injected to the application at compile time in order to 
not be present in the source code.

The steps are shown from a top to bottom perspective.

## Calling code

In our code, at some point we want to perform an invocation to the **Directions API**
endpoint to get the possible routes set from a start to an end point.

In order to perform the call we authenticate/authorize our code via the **API Key**

The code to fetch the **API Key** could be something like the below:

```java
    private static final String APPLICATION_PROPERTIES_FILE_NAME = "application.properties";
    private static final String API_KEY_PROPERTY_KEY = "api.key";
    
    ...
    
    private String getApiKey() {
        try {
            final Properties props = new Properties();
            props.load(Server.class.getClassLoader().getResourceAsStream(APPLICATION_PROPERTIES_FILE_NAME));
            return props.getProperty(API_KEY_PROPERTY_KEY);
        } catch (IOException ioException) {
            LOGGER.severe(ioException.toString());
            return "";
        }
    }
```

In the above code snippet we load the content of a `.properties` file
to a `Properties` object in order to retrieve the value of a specific
property named `api.key`

A same snippet would fetch our `test.api.key` for our test suites:

```java
    private static final String APPLICATION_PROPERTIES_FILE_NAME = "application-test.properties";
    private static final String API_KEY_PROPERTY_KEY = "test.api.key";
    
    ...

    private String getApiKey() {
        try {
            final Properties props = new Properties();
            props.load(ServerTest.class.getClassLoader().getResourceAsStream(APPLICATION_PROPERTIES_FILE_NAME));
            return props.getProperty(API_KEY_PROPERTY_KEY);
        } catch (IOException ioException) {
            LOGGER.severe(ioException.toString());
            return "";
        }
    }
```

## Properties files

The `.properties` files are a good way to organize configurable aspects of your application
and used by various frameworks and tools.

In our case we have two such files named `application.properties` and `application-test.properties`
to hold the separate keys of runtime and testing, respectively.

These files are best placed at:

```
src
|_ main
   |_ resources
```

and

```
src
|_ test
   |_ resources
```

respectively.

Well, now, what may these files contain? 

The concept of these files are to have key value pairs such as:
```properties
api.key=value
```

For the project at hand, in `application.properties` we have:

```properties
api.key=${api.key}
```

What that means is that the `api.key` would take the value of another property (also named `api.key`)
when resolved (thus put in `${...}`). The same also holds for the `application-test.properties`.

## Pom project properties

Well, so we have keys that take the values of other keys when resolved, but where these other keys are 
defined?

The answer comes from the below lines of the `pom.xml`:

````xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    ...    
    
    <properties>
        ...
        <api.key>${api.key}</api.key>
        <test.api.key>${test.api.key}</test.api.key>
    </properties>

    ...

    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
        <testResources>
            <testResource>
                <directory>src/test/resources</directory>
                <filtering>true</filtering>
            </testResource>
        </testResources>
    
        ...
    
    </build>
    
</project>
````

In the file, we define that we have two properties named `api.key` and `test.api.key` that are
the ones that we resolve in the `.properties` files. The resolving is enabled by the 
configuration inside the `<build>` element of the **pom**, which states that the files inside the
two `resources` directories should have their included `${...}` expressions resolved.

Thus far we are good, but what are the values of the properties in the **pom**?

Well, they are `${api.key}` and `${test.api.key}`..

If this is not like [Inception](https://www.imdb.com/title/tt1375666/), then what is?

Ok, so once more we have two properties that take the values of two other properties when resolved.

These final properties are the ones injected as `Java` system properties when we invoke the 
**Maven compile lifecycle step**:

```
mvn compile -Dapi.key=<api.key.value> -Dtest.api.key=<test.api.key.value>
```

The `<api.key.value>` and `<test.api.key.value>` should be replaced with the real one in this step.

## Conclusion

In conclusion, we have the below sequence of steps in order to inject our secret info to our application
without introducing it in the source code.

![Property Resolution](../images/property_resolution.svg)

This process assures us that our secret info of an application is safe,
and it would not be made public, when working in public repos.

