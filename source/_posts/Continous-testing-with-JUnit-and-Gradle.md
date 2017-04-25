---
title: Continous testing with JUnit and Gradle
category: build
date: 2017-04-25 20:55:15
tags: [gradle, testing, continous, junit, java]
---

One of the features I like when working on a small library is continuous testing. I used to setup my IDE to re-run my tests whenever a file changes. This article describes how to setup this kind of testing with Gradle and JUnit.

## Continuous tests

Gradle can turn any build into a [continuous build](https://docs.gradle.org/current/userguide/continuous_build.html). Just run any target you want with the `--continuous` flag:

```
gradle test --continuous
```

Whenever a test file changes, Gradle will re-compile the whole project and run all tests. With the current Gradle changes to incremental build and the enhancements to compilation avoidance, the compile steps are usually very fast. This switch is usually enough for a small project.

## Re-running only changed tests

However the tests run-time can be long and running only changed tests would improve productivity. In that case, Gradle AFAIK doesn't offer anything out of the box. But it's quite easy to add this functionality using a custom [incremental task](https://docs.gradle.org/current/userguide/custom_tasks.html#sec:implementing_an_incremental_task).