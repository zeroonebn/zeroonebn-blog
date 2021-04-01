---
layout: post
title:  "Hello world!"
date:   2021-03-29 15:28:45 -0400
author: Damien Ferrand
---
Welcome to Zero One B N blog.

The goal of this blog is to help IBM i developers (especially ILE RPG) learn how to create web applications.

While "green screen" application development tools have been stable and straightforward for the last 30 years, the web application world is the exact opposite: constantly changing and diverse. There are multiple ways to create web applications on IBM i, this blog will focus on one way to do it. The main tools, languages and frameworks we'll use are:
* IntelliJ IDEA integrated development environment
* The java virtual machine
* The Kotlin programming language
* Spring Boot
* Vaadin

All those tools are open source free software. 

# Prerequisites

## Development computer

The only prerequisite for the development computer is being able to run IntelliJ IDEA. Most reasonably recent version of Windows, macOS or Linux will be supported. You can check on IntelliJ IDEA [system requirements page](https://www.jetbrains.com/help/idea/installation-guide.html#requirements).

You can get IntelliJ IDEA from this [download page](https://www.jetbrains.com/idea/download/).

There are two editions of IntelliJ IDEA: Community and Ultimate. The former is free and open-source while the latter is commercial. Community edition is fine for our use case. Ultimate edition has a very nice Spring integration (among a lot of other features) but you can do a lot without it.

## Production server

Our application can run on IBM i or any other system capable of run java 8 or later.

On IBM i you need version 7.1 minimum: this is licensed program 5761JV1 (for IBM i 7.1) or 5770JV1 (for IBM i 7.2+) option 16 or 17. On IBM i 7.4, the freshly announced option 19 (Java 11 64bits) should work too.

# Our first web application

## Creating the application

Our first web application will be the traditional hello world, we won't be taking over Google today.

The wizard created a few files and directories, we can see them in the top left part of IntelliJ window:

All those files are related to Gradle which will be our build tool. The build tool will have the following responsibilities:
* Downloading the dependencies
* Invoking the compiler
* Packaging the application

In traditional IBM i development you might do those operations manually, have in-house tools or third-party software.

The file build.gradle.kts which is the build definition file should be open on the top right part of IntelliJ IDEA.

We will replace it's content with the following:
```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
    id("org.springframework.boot") version "2.4.4"
    id("com.vaadin") version "0.14.3.7"
    kotlin("jvm") version "1.4.31"
    kotlin("plugin.spring") version "1.4.31"
    kotlin("plugin.jpa") version "1.4.31"
}

repositories {
    mavenCentral()
}

dependencies {
    implementation(kotlin("stdlib"))
    implementation("xyz.zeroonebn:vaadin-spring-boot-starter:0.0.1")
}

tasks.withType<KotlinCompile> {
    kotlinOptions {
        freeCompilerArgs = listOf("-Xjsr305=strict")
        jvmTarget = "1.8"
    }
}
```

Click on the Gradle button on the right of the window and then on the "Reload All Gradle Projects" button:

This will reload the build.gradle.kts file and download the dependencies.

## Creating the entry point class

Every Java (or Kotlin) application needs an entry class which will be called when we launch the application.

In the project area, expand src then main and right click on kotlin.

Enter the name "com.example.HelloWordApplication" and select Class.

This will open the file HelloWorldApplication.kt in the editor, replace its content with following:
```kotlin
package com.example

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class HelloWorldApplication

fun main(args: Array<String>) {
    runApplication<HelloWorldApplication>(*args)
}
```

## Creating the view

The view in charge of describing the user interface and it's behaviour. In a traditional application, this is the display file and the part of the RPG program handling the display.

Using the same procedure, create a Class named "com.example.MainView".

Replace the content of the created file with the following:
```kotlin
package com.example

import com.github.mvysny.karibudsl.v10.label
import com.vaadin.flow.component.orderedlayout.VerticalLayout
import com.vaadin.flow.router.Route

@Route
class MainView: VerticalLayout() {
    
    init {
        label("Hello world!")
    }
}
```

## Running the application on the development machine

To run the application on the development machine, simply open the Gradle button on the right of the screen and expand Tasks then application, then double click the bootRun task.

After a few seconds (likely 20 to 30 seconds), the application will be running.

Simply enter the following url in your web browser: http://localhost:8080.

A simple page containing "Hello World! should appear".

## Building and deploying on IBM i

Building the application will compile our classes and package them with all the required libraries into a single jar file.

Open a command line prompt and navigate to the project directory.

On Windows, run the following command:
```gradle clean build -Pvaadin.productionMode```

On MacOS or Linux, run the following command:
```./gradlew clean build -Pvaadin.productionMode```

This will create the file hello-world.jar in build/libs directory.

Transfer this file in the IFS of the IBM i, you can use Netserver, NFS, ftp, scp, sftp or IBM navigator for i. If using ftp, be sure to use binary mode.

In this example, we'll consider the file was transfered into /tmp.

To run the application we must use JDK 1.8 or higher. Depending on your IBM i version and JDK options installed, a lower version JDK might be the default. To force the use of 64 bit JDK 8 in your job, run the following:
```ADDENVVAR ENVVAR(JAVA_HOME) VALUE('/QOpenSys/QIBM/ProdData/JavaVM/jdk80/64bit’)```

To run you application, run the following command: 
```JAVA CLASS('/tmp/hello-world.jar')```

When the application is running you can display the Hello world page by directing your web browser to the IP of your IBM i on port 8080 (http://ibmi_ip:8080)

If a server is already listening on port 8080 on your IBM i, your application will refuse to start, you can instruct the application to listen on a different port, for example to use port 8081:
```JAVA CLASS('/tmp/hello-world.jar') PARM('-Dserver.port=8081')```

