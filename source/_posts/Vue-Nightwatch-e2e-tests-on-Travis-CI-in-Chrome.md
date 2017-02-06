---
title: Vue Nightwatch e2e tests on Travis CI in Chrome
date: 2017-02-07 09:12:43
tags: [javascript, testing, nightwatch, travis, e2e]
---

[Travis CI](https://travis-ci.org/) is a great continuous integration tool available for free for open source projects with seamless GitHub integration. It can test various projects thanks to different virtual machine images available. However if you need to mix languages, things get a bit more complex.

If you have a JavaScript project with e2e tests using [Nightwatch](http://nightwatchjs.org/), you won't be able to run them on NodeJS Travis image. Nightwatch depends on Java to run the Selenium server. Fortunately the Java image contains `node`, `npm` and `nvm`. So to make it work, I used the following Travis configuration in `.travis.yaml`:

```
sudo: required
dist: trusty
language: java
addons:
  apt:
    sources:
      - google-chrome
    packages:
      - google-chrome-stable
jdk:
  - oraclejdk8
node_js:
  - 6
before_install:
  - export CHROME_BIN=chromium-browser
  - export DISPLAY=:99.0
  - sh -e /etc/init.d/xvfb start
  - nvm install 6.9.5
  - npm install -g yarn
install:
  - yarn install
script:
  - yarn test
```

The `sudo` is required to install `chrome`. The latest Nightwatch requires Java 8. I am not sure if the `node_js` has any effect and might as well try to remove it. I think the image is running only an older version of `node`. The `before_install` part sets environmental properties to point to `chrome` and a virtual display. Then the `xvfb` creates the virtual display that is used by `chrome`. Last the version of `node` that I am using for development is installed. Finally `yarn` is installed and tests are run through `yarn`.

