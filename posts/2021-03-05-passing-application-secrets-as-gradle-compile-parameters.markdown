---
title: Passing Application Secrets as Gradle Compile Parameters
---

## Intro

On the previous post, 
[Passing Applications Secrets as Maven Compile Parameters](https://steve-papadogiannis.github.io/posts/2021-03-04-passing-application-secrets-as-maven-compile-parameters.html),
a process to pass application secrets on **Maven** compile step was analyzed.
In this post, a resembling process will be elaborated for **Gradle** build tool.

The key concept, again, is to avoid making your application secrets 
(**private keys**, **API keys**, etc.) visible on a public repo, and hence,
not exploitable by others.

---

## Setup

The solution proposed here is to pass such information to the application 
at compile/build time, utilizing code or configuration of tools you use in your project.

To illustrate this, let\'s say we have a **Gradle** **Android** project 
which uses an **API Key** for authenticating/authorizing access 
to a **Google Service**, such as **Maps API**.
I have such a project at [dist-sys-client-android](https://github.com/steve-papadogiannis/dist-sys-client-android).

---

## Secrets

So the application uses two properties, `debug_api_key` and `release_api_key` 
to hold the values of two different keys, one for **Debug APK** and the other 
for **Release APK**. 

Producing **Debug/Release APK**s is a concept of **Android** programming,
where you produce different **artifacts** of your **Android** application,
relevant to the state of development you are working on. 
The **Debug APK** is used for **debugging/testing** purposes, whilst 
the **Release APK** is the signed one which can be officially published to 
**Google Store**, etc.

Let see how the values can be injected to the application at compile time in order to
not be present in the source code.

The steps are shown from a top to bottom perspective.

---

## Calling code

In our code, at some point we want to render a 
`com.google.android.gms.maps.SupportMapFragment` that utilizes **Maps API**
and requires an **API Key** to authenticate/authorize the access.

The configuration of the **API Key** that will be used 
by the maps library, is done **declaratively** inside the applcation\'s 
`AndroidManifest.xml` file like below:

<script src="https://gist.github.com/steve-papadogiannis/e19913f13517833800d4441a36662ce3.js"></script>

[comment]: <> (```xml)
[comment]: <> (<?xml version="1.0" encoding="utf-8"?>)
[comment]: <> (<manifest)
[comment]: <> (        xmlns:android="http://schemas.android.com/apk/res/android")
[comment]: <> (        package="gr.papadogiannis.stefanos.DistSysClientAndroid">)
[comment]: <> (    ...)
[comment]: <> (    <application)
[comment]: <> (    android:allowBackup="false")
[comment]: <> (    android:icon="@mipmap/ic_launcher")
[comment]: <> (    android:label="@string/app_name")
[comment]: <> (    android:roundIcon="@mipmap/ic_launcher_round")
[comment]: <> (    android:supportsRtl="true")
[comment]: <> (    android:theme="@style/AppTheme">)
[comment]: <> (        <meta-data)
[comment]: <> (        android:name="com.google.android.geo.API_KEY")
[comment]: <> (        android:value="@string/google_maps_key"/>)
[comment]: <> (        ...)
[comment]: <> (    </application>)
[comment]: <> (</manifest>)
[comment]: <> (```)

As we see the `com.google.android.geo.API_KEY` takes an **android:value** of type 
**string** that should be present in the **resources** of the application

---

## Artifact specific resources

Now for the interesting part..

**Android** framework allows you to define **resources** for your application
based on the artifact you are producing, meaning that you can have the aforementioned
value resolved either from:

<script src="https://gist.github.com/steve-papadogiannis/c221b3a0f33bebb24d6e09573009026e.js"></script>

[comment]: <> (```)
[comment]: <> (app)
[comment]: <> (|_ src)
[comment]: <> (   |_ debug)
[comment]: <> (      |_ res)
[comment]: <> (         |_ values)
[comment]: <> (            |_ google_maps_api.xml)
[comment]: <> (```)

or from:

<script src="https://gist.github.com/steve-papadogiannis/998ae1f301443ce4cd79dae5057bd6a5.js"></script>

[comment]: <> (```)
[comment]: <> (app)
[comment]: <> (|_ src)
[comment]: <> (   |_ release)
[comment]: <> (      |_ res)
[comment]: <> (         |_ values)
[comment]: <> (            |_ google_maps_api.xml)
[comment]: <> (```)

The contents of the files are:

<script src="https://gist.github.com/steve-papadogiannis/c31429a7b0003b459d4db0e2e7adf0bf.js"></script>

[comment]: <> (```xml)
[comment]: <> (<resources>)
[comment]: <> (    <string name="google_maps_key" templateMergeStrategy="preserve" translatable="false">@string/debug_api_key</string>)
[comment]: <> (</resources>)
[comment]: <> (```)

and:

<script src="https://gist.github.com/steve-papadogiannis/40a5ad1c31a49ae5520a868c55728332.js"></script>

[comment]: <> (```xml)
[comment]: <> (<resources>)
[comment]: <> (    <string name="google_maps_key" templateMergeStrategy="preserve" translatable="false">@string/release_api_key</string>)
[comment]: <> (</resources>)
[comment]: <> (```)

respectively. So the **@string/google_maps_key** would take the value of 
either the **@string/debug_api_key**, or the **@string/release_api_key**,
depending on the **artifact**.

---

## Gradle Properties

In this step, we have to define the values for **@string/debug_api_key**
and **@string/release_api_key**

In **Gradle** we can have (at least) two injection points. We can have 
values injected either from **local** **gitignored** `.properties` **files** 
or from **Environment variables** (the latter technique is more valuable on **CI**
systems that may not give you a way to configure the files of your **build** env
but enable you to define **Environment variables**).

The code snippet below, extracted from `app/build.gradle` 
gives a glimpse of the solution at hand:

<script src="https://gist.github.com/steve-papadogiannis/50291c95e30d0dd1a0d8ab1801e62352.js"></script>

[comment]: <> (```groovy)
[comment]: <> (...)
[comment]: <> (def getApiKey&#40;String fileName, String apiKeyProperty, String apiKeyEnvVar&#41; {)
[comment]: <> (    Properties apiKeyProperties = new Properties&#40;&#41;)
[comment]: <> (    def file = project.rootProject.file&#40;fileName&#41;)
[comment]: <> (    if &#40;file.exists&#40;&#41;&#41; {)
[comment]: <> (        apiKeyProperties.load&#40;file.newDataInputStream&#40;&#41;&#41;)
[comment]: <> (        return apiKeyProperties.getProperty&#40;apiKeyProperty&#41; ?: "Please define $apiKeyProperty in $fileName")
[comment]: <> (    } else {)
[comment]: <> (        return System.getenv&#40;apiKeyEnvVar&#41; ?: "Please define $apiKeyEnvVar environment variable")
[comment]: <> (    })
[comment]: <> (})
[comment]: <> (android {)
[comment]: <> (    ...)
[comment]: <> (    defaultConfig {)
[comment]: <> (        ...)
[comment]: <> (        resValue "string", "debug_api_key", getApiKey&#40;"debug-api-key.properties", "api.key", "DEBUG_API_KEY"&#41;)
[comment]: <> (        resValue "string", "release_api_key", getApiKey&#40;"release-api-key.properties", "api.key", "RELEASE_API_KEY"&#41;)
[comment]: <> (        ...)
[comment]: <> (    })
[comment]: <> (    ...)
[comment]: <> (})
[comment]: <> (...)
[comment]: <> (```)

A method is defined named **getApiKey** which takes three **arguments**
the **fileName** to look for, the **apiKeyProperty** to lookup in the **fileName**
`.properties` file, and the **apiKeyEnvVar** to lookup in the 
system environment variables when the **fileName** specified does not exist.

The resolution is pretty straightforward. If the file **fileName** exists,
proceed with extracting the **apiKeyProperty** value if defined or return 
a detailed message as the api key to enable easier debugging, otherwise,
proceed with extracting the **apiKeyEnvVar** value if defined or, again,
return a detailed message as the api key to enable easier debugging.

The invocation of method is also self documenting, and declares that 
we want to define two **string** value resources named **debug_api_key**
and **release_api_key** (remember the properties from the previous section)
which would be resolved from the use of corresponding triplets of arguments,
respectively.

So, for example, for the **debug_api_key**, the method tries to extract the 
value of the property **api.key** from a file with filename **debug-api-key.properties**
and if it fails to find the file, it tries to extract the value of 
the **Environment Variable** named **DEBUG_API_KEY**.

---

## Local Application `.properties` Files

If we chose to use the local application `.properties` files then we should 
have two such files, named **debug-api-key.properties** and **release-api-key.properties**
in the root directory of our project.

The main property of these files is the below one:

<script src="https://gist.github.com/steve-papadogiannis/91b4af16f86e04d3870983c3b4a16332.js"></script>

[comment]: <> (```properties)
[comment]: <> (api.key=<api.key.value>)
[comment]: <> (```)

Keep in mind that these file have to be pattern or exactly matched with at least
one rule in the `.gitignore` file of the project.

For example:

<script src="https://gist.github.com/steve-papadogiannis/8b3285a2f15176e484a11c9190960daa.js"></script>

[comment]: <> (```gitignore)
[comment]: <> (debug-api-key.properties)
[comment]: <> (release-api-key.properties)
[comment]: <> (```)

---

## Gradle Environment Variables

If we chose to use the **Gradle** environment variables then we can set the 
**DEBUG_API_KEY** environment variable like below:

<script src="https://gist.github.com/steve-papadogiannis/70b4156460369321d2088bc0bf6c3d88.js"></script>

[comment]: <> (```shell)
[comment]: <> (DEBUG_API_KEY=<api.key.value>)
[comment]: <> (export DEBUG_API_KEY)
[comment]: <> (```)

With that give, when the build command is issued:

<script src="https://gist.github.com/steve-papadogiannis/6bcb858080f3dfec3b35e302db860840.js"></script>

[comment]: <> (```shell)
[comment]: <> (./gradlew build connectedCheck)
[comment]: <> (```)

then the property\'s value will be
resolved to the environment variable\'s value

---

## Conclusion

In conclusion, as in the previous post, we have defined a sequence of steps 
that inject our secret info to our application
without introducing it in the source code.

![Fig. 1: Property Resolution](../images/property_resolution_gradle.svg)

This process assures us that our secret info of an application is safe,
and it would not be made public, when working in public repos.
