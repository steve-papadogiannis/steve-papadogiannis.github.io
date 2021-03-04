---
title: Passing Application Secrets as Maven Compile Parameters
---

## Intro

When you have a project that is stored on a public repo, you have to have a process to not accidentally commit and push
your application's secrets (`private keys`, `API keys`, etc.)
and have them visible to the public eye, unless you want to be exploited for good.

## Setup

One solution would be to pass such information to the application at compile/build time, utilizing code or configuration
of tools you use in your project.

Let say we have a `Maven` `Java` project which uses an `API Key` for authenticating/authorizing access to
a `Google Service`, such as `Directions API`. I have such a project at
https://github.com/steve-papadogiannis/dist-sys-server-java which gave me the idea of the topic of this post.

## Secrets

So our application uses two properties, `api.key` and `test.api.key` to hold the values of two different keys, one for
runtime and the other for testing purposes.

Let see how the values can be injected to the application at compile time in order to not be present in the source code.

The steps are shown from a top to bottom perspective.

## Calling code

In our code, at some point we want to perform an invocation to the `Directions API`
endpoint to get the possible routes set from a start to an end point.

In order to perform the call we authenticate/authorize our code via the `API Key`

The code to fetch the `API Key` could be something like the below:

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

The 