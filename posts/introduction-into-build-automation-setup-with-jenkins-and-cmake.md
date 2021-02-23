---
title: Introduction into an Automated C++ Build Setup with Jenkins and CMake
description: 
date: 2019-03-27
tags:
  - c++
  - build-automation
  - catch2
  - cicd
  - cmake
  - jenkins
  - pipeline
  - vcpkg
layout: layouts/post.njk
image: /img/pexels-panumas-nikhomkhai-1148820.jpg
---

Welcome back to a new post on thoughts-on-cpp.com. This time I would like to give an introduction into an automated build setup which, based upon [Jenkins][1] and [CMake][2], fulfills the following needs:

- Building a ready to deploy release on every commit
- Execution of all tests
- Running static code analysis to track code quality
- and easy to extend with automated deployment (CD)

For all the necessary sources, I prepared a [GitHub repository][3]. We are focusing here only on a technical part of an automated build process which is a prerequisite of a CI/CD (Continues Integration/Continues Deployment) process. It is quite a bit more necessary than just tools for a company to fully embrace the ideas behind a CI/CD process but with an automated build and test setup your on a good way.

![Hero Image: Server, Photo by panumas nikhomkhai from Pexels](/img/pexels-panumas-nikhomkhai-1148820.jpg)

The build, of a Qt and C++ based example desktop application, will be orchestrated by [Jenkins declarative pipeline][4]. The example application is based on a simple CMake project generated with [CLion][5]. As static code analysis tool, we are using [cppcheck][6]. Testing is done with my favorite testing framework [Catch2][7].

In this post, I presume your already familiar with a basic Jenkins setup. If you're not familiar with Jenkins, jenkins.io is a good source of information.

Jenkins consists of a declarative pipeline defined in a file called Jenkinsfile. This file has to be located in the projects root folder. The declarative pipeline of Jenkins is based upon [Groovy][8] as a [DSL][9] and provides a very expressive way to define a build process. Even though the DSL is very mighty because it is based on Groovy you can actually write little scripts, its documentation is, unfortunately, a bit mixed up with its predecessor, the scripting-based pipeline. For our example setup, it's looking the following way.

```groovy
pipeline {
	agent any

	options {
		buildDiscarder(logRotator(numToKeepStr: '10'))
	}

	parameters {
		booleanParam name: 'RUN_TESTS', defaultValue: true, description: 'Run Tests?'
		booleanParam name: 'RUN_ANALYSIS', defaultValue: true, description: 'Run Static Code Analysis?'
		booleanParam name: 'DEPLOY', defaultValue: true, description: 'Deploy Artifacts?'
	}

	stages {
        stage('Build') {
            steps {
                cmake arguments: '-DCMAKE_TOOLCHAIN_FILE=~/Projects/vcpkg/scripts/buildsystems/vcpkg.cmake', installation: 'InSearchPath'
                cmakeBuild buildType: 'Release', cleanBuild: true, installation: 'InSearchPath', steps: [[withCmake: true]]
            }
        }

        stage('Test') {
            when {
                environment name: 'RUN_TESTS', value: 'true'
            }
            steps {
                ctest 'InSearchPath'
            }
        }

        stage('Analyse') {
            when {
                environment name: 'RUN_ANALYSIS', value: 'true'
            }
            steps {
                sh label: '', returnStatus: true, script: 'cppcheck . --xml --language=c++ --suppressions-list=suppressions.txt 2> cppcheck-result.xml'
                publishCppcheck allowNoReport: true, ignoreBlankFiles: true, pattern: '**/cppcheck-result.xml'
            }
        }

        stage('Deploy') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                sh label: '', returnStatus: true, script: '''cp jenkinsexample ~
                cp test/testPro ~'''
            }
        }
	}
}
```

The syntax is pretty straight forward. A Jenkinsfile starts always with the declaration of a pipeline block, followed by the declaration of an [agent][10]. An agent describes on environment our build should be executed. In our case, we want it on any environment setup, but it could be also a labeled or a [docker][11] environment.

With the [options][12] directive, we define that we want to keep the last 10 build artifacts and code analysis results. With options we could also define a general build timeout, the number of retries we allow in case of a build failure, or the timestamp for console output while running the build.

The [parameters][13] directive provides us the possibility to define arbitrary build parameters of the types `string`, `text`, `booleanParam`, `choice`, `file`, and `password`. In our case, we use `booleanParam` to provide the user with an option to define which additional stages the user wants to execute in case of a manual execution of the project.

![Customized boolean parameters in Jenkins to perform presets of builds](/img/jenkinsparameters-min.png)
*Customized boolean parameters in Jenkins to perform presets of builds*

But even if those configuration possibilities of Jenkins are vast and very interesting, the really important part of the build declaration file is defined by the [stages][14] section with its [stage][15] directives. With an arbitrary number of possible stages, we have all the freedom to define our build process as we need to. Even [parallel][16] stages, for example, concurrent execution of testing and static code analysis, are possible.

With the first stage, "Build", we are instructing Jenkins to invoke its CMake-Plugin to generate our build setup and resolving all necessary dependencies via a [Vcpkg][17] CMake file. Afterward, the build is executed with `cmake --build .` through the `withCmake: true` setting. User-defined toolchains are also no problem so we could have also defined several build setups with GCC, Clang and Visual Studio Compiler.

The other stages, Test, Analyse and Deploy, are again pretty straight forward. All of them have one notable thing in common which is the [when][18] directive. With this directive, we can control if a stage gets executed if the condition inside the curly braces returns true. In our case, we use the when directive to evaluate the build parameters we introduced at the beginning of this post. The syntax might be a bit irritating at first glance but after all, it does its job.

To get Jenkins executing our nice pipeline we just need to tell it from which repository to pull. This can be done via the project configuration. Here you just need to choose the option 'Pipeline script from SCM'. If everything is set up correctly you end up with a smooth running automated build process. Even better, if you have configured Jenkins correctly and it has a static connection to the internet, you can connect Jenkins to GitHub over a [Webhook][19]. This means that GitHub will invoke a build every time someone pushed a commit to the repository.

![Example configuration of a GitHub based git repository](/img/jenkinssourcecontrolconnection-min.png)
*Example configuration of a GitHub based git repository*

To conclude this post I would like to point out that this isn't **THE BEST WAY** to configure automated builds. It is one way of millions possible. And it's the way which worked out pretty well for me in my daily work. I introduced an adapted version of this several years ago in our company and Jenkins is serving us great since then. One important fact of Jenkins declarative pipeline is the point that it's versioned over the SCM as the rest of the code is, and that's a very important feature.

[1]: https://jenkins.io/
[2]: https://cmake.org/
[3]: https://github.com/Ben1980/jenkinsexample
[4]: https://jenkins.io/doc/book/pipeline/syntax/
[5]: https://www.jetbrains.com/clion/
[6]: http://cppcheck.sourceforge.net
[7]: https://github.com/catchorg/Catch2
[8]: http://groovy-lang.org/
[9]: https://en.wikipedia.org/wiki/Domain-specific_language
[10]: https://jenkins.io/doc/book/pipeline/syntax/#agent
[11]: https://www.docker.com/
[12]: https://jenkins.io/doc/book/pipeline/syntax/#options
[13]: https://jenkins.io/doc/book/pipeline/syntax/#parameters
[14]: https://jenkins.io/doc/book/pipeline/syntax/#stages
[15]: https://jenkins.io/doc/book/pipeline/syntax/#stage
[16]: https://jenkins.io/doc/book/pipeline/syntax/#parallel
[17]: https://github.com/Microsoft/vcpkg
[18]: https://jenkins.io/doc/book/pipeline/syntax/#when
[19]: https://developer.github.com/webhooks/