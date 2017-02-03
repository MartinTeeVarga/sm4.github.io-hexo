---
title: Gradle Release Plugin in Multi project Gradle build
category: build
date: 2017-02-03 13:30:40
tags: [gradle, release]
---

The typical software release at the end of iteration includes incrementing the software version, tagging the release in version control and publishing production artifacts. The [Gradle Release Plugin](https://github.com/researchgate/gradle-release) does all that and more. It makes releasing a Gradle project very easy. Until you need to release a multi-project gradle build. The official workaround did not work for me, so I have created my own.

## The problem
When simply adding the release plugin to the root project, tasks that should be run only once (such as VCS tagging) are run multiple times, thus failing the build. When adding the release plugin to the subprojects, there are issues with the separate versions and the fact that the VCS repository root is in the parent directory. 

## Official workaround
The workaround described on the plugin homepage recommends to add the release plugin to the root and then run the release tasks separately for sub-projects. In my opinion, this creates a confusion between versions (i.e. each sub-project is released on its own, increasing the version every time).

## My workaround
Based on the official workaround, it automates the release process and avoids creating multiple versions. The solution is to disable the release task in sub-projects to skip any release tasks executing on a wrong level while still publishing all sub-projects artifacts. Note that I am using the gradle wrapper, so I have added it to the script example.

```
plugins {
    id 'net.researchgate.release' version '2.3.5'
}
 
allprojects {
    apply plugin: 'maven-publish'
}
 
subprojects {
    task release(overwrite: true) {
        //overwrite release task in the subprojects
    }
}
 
afterReleaseBuild.dependsOn(':moduleA:publish', ':moduleB:publish')
 
task wrapper(type: Wrapper) {
    gradleVersion = '2.11'
}
```

To disable the release task in all sub-modules, it's possible to overwrite it in the main script subprojects part with an empty task. To enable publishing of the sub-modules artifacts, the dependency on the sub-project publish task must be defined as per the example for each sub-project that should have its artifacts published.

To release the project, simply run: 
```
gradlew release
```
