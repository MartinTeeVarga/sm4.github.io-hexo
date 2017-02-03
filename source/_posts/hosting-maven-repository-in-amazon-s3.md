---
title: "Hosting a Maven Repository in Amazon S3"
category: build
tags: [s3, aws, cloud, maven, gradle, build]
date: 2015-07-16 00:00:00
---
We had a simple requirement to host our internal deployed artifacts in the Amazon Cloud. We have migrated most of our builds to Gradle. Since version 2.4, Gradle added support for repositories hosted in Amazon AWS S3.

## Ivy vs. Maven

Both Ivy and Maven repositories in S3 are supported by Gradle. But since Ivy is trying to mimic maven and we already had a maven repo, I picked maven. 

## Setup

I've created an S3 bucket and inside a simple structure:

```
maven
|- snapshots
|- internal
```
    
## Getting dependencies

The following code in `build.gradle` enables download of the dependencies: 

```
apply plugin: 'maven'

repositories {
    maven {
        name "s3snapshots"
        url "s3://bucket-name/maven/snapshots"
        credentials(AwsCredentials) {
            accessKey "aws access key"
            secretKey "aws secret key"
        }
    }
    maven {
        name "s3internal"
        url "s3://bucket-name/maven/internal"
        credentials(AwsCredentials) {
            accessKey "aws access key"
            secretKey "aws secret key"
        }
    }
}
```

## Publishing dependencies

I played with the uploadArchives gradle task, but it was creating the Ivy structure. I have decided to go with the newer maven-publish plugin. 

The following shows how to switch between two repositories and how to reference previously defined repositories. The switching works very well
with [Gradle Release Plugin](https://github.com/researchgate/gradle-release) that we are using now.

```
apply plugin: 'maven-publish'

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        add project.version.endsWith('-SNAPSHOT') ? project.repositories.s3RepoSnapshots : project.repositories.s3RepoInternal
    }
}
```

To include source jars, test jars or other archives, e.g. from distribution plugin, add this to the publication. 
Group Id, Artifact Id can be set here as well. 

```
apply plugin: 'maven-publish'

publishing {
    publications {
        maven(MavenPublication) {
            groupId `group`
            artifactId 'artifact'
            from components.java
            artifact sourceJar {
                classifier "sources"
            }
            artifact testJar {
                classifier "test"
            }
            artifact distZip {
                classifier "zip"
            }
        }
    }
}
```

## Migration from other maven repositories

I have successfully migrated Apache Archiva to S3 simply by copying the whole directory structure from Archiva's data folder to S3. Any maven repository should be possible to migrate the same way.

## Missing features

The main disadvantage of this solution is the lack of administration and inability to prune snapshots. I am planning to create a utility that will take care of that.

Fortunately the S3 cost is very small so this will do for now.

## Example

[You can find an example implementation on GitHub.](https://github.com/sm4/s3repo-demo)