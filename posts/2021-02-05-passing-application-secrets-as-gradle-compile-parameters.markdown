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

## Setup

The solution proposed here is to pass such information to the application 
at compile/build time, utilizing code or configuration of tools you use in your project.

To illustrate this, let\'s say we have a **Gradle** **Android** project 
which uses an **API Key** for authenticating/authorizing access 
to a **Google Service**, such as **Maps API**.
I have such a project at [dist-sys-client-android](https://github.com/steve-papadogiannis/dist-sys-client-android).

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

## Calling code

In our code, at some point we want to render a 
`com.google.android.gms.maps.SupportMapFragment` that utilizes **Maps API**
and requires an **API Key** to authenticate/authorize the access.

The configuration of the **API Key** that will be used 
by the maps library, is done **declaratively** inside the applcation\'s 
`AndroidManifest.xml` file like below:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
        xmlns:android="http://schemas.android.com/apk/res/android"
        package="gr.papadogiannis.stefanos.DistSysClientAndroid">

    ...

    <application
            android:allowBackup="false"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:roundIcon="@mipmap/ic_launcher_round"
            android:supportsRtl="true"
            android:theme="@style/AppTheme">

        <meta-data
                android:name="com.google.android.geo.API_KEY"
                android:value="@string/google_maps_key"/>

        ...

    </application>

</manifest>
```

As we see the `com.google.android.geo.API_KEY` takes an **android:value** of type 
**string** that should be present in the **resources** of the application

## Artifact specific resources

Now for the interesting part..

**Android** framework allows you to define **resources** for your application
based on the artifact you are producing, meaning that you can have the aforementioned
value resolved either from:

```
app
|_ src
   |_ debug
      |_ res
         |_ values
            |_ google_maps_api.xml
```

or from:

```
app
|_ src
   |_ release
      |_ res
         |_ values
            |_ google_maps_api.xml
```

The contents of the files are:

```xml
<resources>
    <string name="google_maps_key" templateMergeStrategy="preserve" translatable="false">@string/debug_api_key</string>
</resources>
```

and:

```xml
<resources>
    <string name="google_maps_key" templateMergeStrategy="preserve" translatable="false">@string/release_api_key</string>
</resources>
```

respectively. So the **@string/google_maps_key** would take the value of 
either the **@string/debug_api_key**, or the **@string/release_api_key**,
depending on the **artifact**.

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

```groovy

...

def getApiKey(String fileName, String apiKeyProperty, String apiKeyEnvVar) {
    Properties apiKeyProperties = new Properties()
    def file = project.rootProject.file(fileName)
    if (file.exists()) {
        apiKeyProperties.load(file.newDataInputStream())
        return apiKeyProperties.getProperty(apiKeyProperty) ?: "Please define $apiKeyProperty in $fileName"
    } else {
        return System.getenv(apiKeyEnvVar) ?: "Please define $apiKeyEnvVar environment variable"
    }
}

android {

    ...

    defaultConfig {

        ...

        resValue "string", "debug_api_key", getApiKey("debug-api-key.properties", "api.key", "DEBUG_API_KEY")
        resValue "string", "release_api_key", getApiKey("release-api-key.properties", "api.key", "RELEASE_API_KEY")

        ...

    }

    ...

}

...
```

A method is defined named **getApiKey** which takes three **arguments**
the **fileName** to look for, the **apiKeyProperty** to lookup in the **fileName**
`.proeperties` file and the **apiKeyEnvVar** to lookup in the 
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
and if it fails to find the file tries to extract the value of 
the **Environment Variable** named **DEBUG_API_KEY**.

## Local Application `.properties` Files


## Gradle Environment Variables

## Conclusion


