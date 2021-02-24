---
title: Introduction into C++ Builds with Gradle
description: 
date: 2019-04-10
tags:
  - c++
  - build-automation
  - gradle
  - testing
  - tool
layout: layouts/post.njk
image: /img/build.png
---

Welcome back to a new post on thoughts-on-cpp.com. In today's post, I would like to give an introduction to the build system Gradle and how we can use it to build native applications and libraries. Gradle is originally coming from the Java world, but it's also supporting native build toolchains for quite a while. Gradle is becoming more and more popular in the Java world and is on a good way to rule out the old bull Maven. This is because of two features which we can also benefit from in the native (C/C++, Objective-C/C++, Assembly, and Windows resources) build world. These features are Gradle's easy to maintain and very expressive Groovy (or Kotlin if preferred) based [DSL][1], and it's capabilities of dependency resolving via online and on-premise library providers (such as maven-central, Artifactory, Bintray, etc.) or local repositories.

![Hero Image: Just build](/img/build.png)

Let's start with a [multi-project example][2] we already know from my post [Introduction into an Automated C++ Build Setup with Jenkins and CMake][3]. I have just slightly changed the example hello world application. This time the main function is printing out "Hello World!" onto console using a shared library, called greeter. The Greeter class itself is utilizing the external library [{fmt}][4] to print "Hello World" onto the screen. If you've wondered about the Gradle directory and files, those are provided by the Gradle wrapper which facilitates us to build the project without even installing Gradle upfront.

```bash
.
├── app
│   ├── build.gradle
│   └── src
│       └── main
│           └── cpp
│               └── main.cpp
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
├── greeter
│   ├── build.gradle
│   └── src
│       └── main
│           ├── cpp
│           │   └── greeter.cpp
│           └── public
│               └── greeter.hpp
├── LICENSE
├── README.md
├── settings.gradle
└── testLib
    ├── build.gradle
    └── src
        └── test
            └── cpp
                └── greetertest.cpp
```

The Gradle native build plugin is quite straight forward to configure. Every Gradle project needs a build.gradle file at its root directory as an entry point and one at each subproject. In most cases, we will do general configurations in a build.gradle file located at the root directory. But there is no need, it can also be empty. By default, Gradle is looking for sources in the directory `src/main/cpp`. For libraries, public headers are defined in `src/main/public` and in case they should be used only library internal (private) the default directory is `src/main/headers`. In case we define the headers also in `src/main/cpp`, the headers are treated as private as well. If we like to overwrite the default source directories we just need to define them according to this [example][5]. To be able to resolve dependencies between subprojects, we need to define a settings.gradle file which is including our subprojects `include 'app', 'greeter', 'testLib'`.

```groovy
components.withType(ProductionCppComponent) {
    //By convention, source files are located in the root directory/Sources/
    source.from rootProject.file("Sources/${subproject.name.capitalize()}")
    privateHeaders.from rootProject.file("Sources/${subproject.name.capitalize()}/include")
}
components.withType(CppLibrary) {
    //By convention, public header files are located in the root directory/Sources//include
    publicHeaders.from rootProject.file("Sources/${subproject.name.capitalize()}/include")
}
```

As build definition of our root directory we just simply define IDE support for each subproject. [CLion][6], for example, has native Gradle support, so importing Gradle projects works smooth as silk. Therefore our root build.gradle file looks like the following.

```groovy
allprojects {
    apply plugin: 'xcode'
    apply plugin: 'visual-studio'
}
```

The application build configuration is defined at the app directory starting with calling the [`cpp-application`][7] plugin which is generating an executable file which can be found and executed at `app/build/install/main/{buildType}/{machine}`. Project internal dependencies can be defined by the `dependencies` clause with the implementation of the dependency defined as a project and the given name of the dependency. By default, Gradle is assuming the current host as target machine. If we want to consider other target machines we have to declare them as we do in our example with the [`targetMachines`][8] statement.

```groovy
plugins {
    id 'cpp-application'
}

application {
    dependencies {
        implementation project(':greeter')
    }

    targetMachines = [
        machines.windows.x86_64,
        machines.macOS.x86_64,
        machines.linux.x86_64
    ]
    
    baseName = "app"
}
```

The library build configuration is defined at the greeter directory starting with calling the [`cpp-library`][9] plugin and the type of [linkage][10], which can be `STATIC` and `SHARED`. Gradle is assuming we want `SHARED` libraries as default. A bit special is the way of how we have to resolve the dependency to the header only library {fmt}. Unfortunately, Gradle is not supporting header only libraries out of the box, but we can accomplish a workaround by adding the include path to the `includePathConfiguration` of the resulting binary. All other dependencies can be defined as api, in case we want to share the external dependency `api` with all consumers of our own defined library, or `implementation` in case we only want to use the dependency api private with our own library. A good example can be found in Gradle's [example repository][11].

```groovy
plugins {
    id 'cpp-library'
}

library {
    linkage = [Linkage.SHARED]

    targetMachines = [
        machines.windows.x86_64,
        machines.macOS.x86_64,
        machines.linux.x86_64
    ]

    baseName = "greeter"
}

def fmtHeaders = file("$rootDir/../fmt/include")

components.main.binaries.whenElementFinalized { binary ->
    project.dependencies {
        if (binary.optimized) {
            add(binary.includePathConfiguration.name, files(fmtHeaders))
        } else {
            add(binary.includePathConfiguration.name, files(fmtHeaders))
        }
    }
}
```

With Gradle, we can not only build applications and libraries, but we can also execute tests to check the resulting artifacts. A test can be defined by the `cpp-unit-test` plugin which is generating a test executable. In principle, we could use any of the existing big test libraries, such as [googletest][12], but in my opinion, the out of the box solution is pretty neat and lightweight and can be extended quite easily with external libraries.

```groovy
plugins {
    id 'cpp-unit-test'
}

unitTest {
    dependencies {
        implementation project(':greeter')
    }

    baseName = "greeterTest" 
}
```

With this project setup, we can build all artifacts by the command `./gradlew assemble` and run tests by `./gradlew check`. If we want to build and run all tests together we can invoke `./gradlew build`. In case we need a list of all available tasks provided by Gradle and its plugins we can simply list them including their description by the command `./gradlew tasks`. At [GitHub][2] you can find the resulting repository.

```bash
[bmahr@localhost gradleNative]$ ./gradlew tasks

> Task :tasks

------------------------------------------------------------
Tasks runnable from root project
------------------------------------------------------------

Build tasks
-----------
assemble - Assembles the outputs of this project.
build - Assembles and tests this project.
clean - Deletes the build directory.

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'gradleNativ'.
components - Displays the components produced by root project 'gradleNativ'. [incubating]
dependencies - Displays all dependencies declared in root project 'gradleNativ'.
dependencyInsight - Displays the insight into a specific dependency in root project 'gradleNativ'.
dependentComponents - Displays the dependent components of components in root project 'gradleNativ'. [incubating]
help - Displays a help message.
model - Displays the configuration model of root project 'gradleNativ'. [incubating]
projects - Displays the sub-projects of root project 'gradleNativ'.
properties - Displays the properties of root project 'gradleNativ'.
tasks - Displays the tasks runnable from root project 'gradleNativ' (some of the displayed tasks may belong to subprojects).

IDE tasks
---------
cleanVisualStudio
cleanXcode - Cleans XCode project files (xcodeproj)
openVisualStudio - Opens the Visual Studio solution
openXcode - Opens the Xcode workspace
visualStudio
xcode - Generates XCode project files (pbxproj, xcworkspace, xcscheme)

Verification tasks
------------------
check - Runs all checks.
runTest - Executes C++ unit tests.

Rules
-----
Xcode bridge tasks begin with _xcode. Do not call these directly.
Pattern: clean<TaskName>: Cleans the output files of a task.

To see all tasks and more detail, run gradlew tasks --all

To see more detail about a task, run gradlew help --task <task>

BUILD SUCCESSFUL in 0s
1 actionable task: 1 executed
```

![Build executed in shell](/img/gradlebuild.gif)

UPDATE: Thanks to [D. Lacasse][13], I updated the source set configuration section to customize the folder structure of a project.

[1]: https://en.wikipedia.org/wiki/Domain-specific_language
[2]: https://github.com/Ben1980/gradleNativ
[3]: https://thoughts-on-coding.com/2019/03/27/introduction-into-build-automation-setup-with-jenkins-and-cmake/
[4]: http://fmtlib.net/latest/index.html
[5]: https://github.com/gradle/native-samples/blob/master/cpp/swift-package-manager/build.gradle#L9-L17
[6]: https://www.jetbrains.com/clion/
[7]: https://docs.gradle.org/current/javadoc/org/gradle/language/cpp/CppApplication.html
[8]: https://docs.gradle.org/current/javadoc/org/gradle/language/ComponentWithTargetMachines.html#getTargetMachines--
[9]: https://docs.gradle.org/current/javadoc/org/gradle/language/cpp/CppLibrary.html
[10]: https://docs.gradle.org/current/javadoc/org/gradle/nativeplatform/Linkage.html
[11]: https://github.com/gradle/native-samples
[12]: https://github.com/google/googletest
[13]: https://twitter.com/lacasseio