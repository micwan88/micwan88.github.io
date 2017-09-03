---
title: How to reference other project via local Maven repository in gradle
categories: java gradle
---

## How to reference other project via local Maven repository in gradle

To use other project as library via local Maven repository, you need to enable your library project for local Maven first. To do so, add `maven` plugin, `version` and `group` in `build.gradle`.


``` gradle
apply plugin: 'java'
apply plugin: 'maven'

//Define your library version and group id, take gatecoinapi4j as example.
//group id is 'mic.trade'
version = '0.1.0-SNAPSHOT'
group = 'mic.trade'
```


Please noted that the artifact info is the `rootProject.name` under `settings.gradle`. For example, 'gatecoinapi4j'.

OK. Now it is ready to build you jar and then push it into local Maven repository via below command:


```
./gradlew clean install
```


After that, `[artifact]-[version].jar` should be installed under your user directory of `~/.m2` and now you can reference the jar file in `build.gradle` from your another project.


``` gradle
repositories {
    mavenLocal()
    //You can add other repository here (like jcenter / mavenCentral) and
    //mavenLocal is use for reference jar build from your local library project
}

//Please modify the below version to match with the source of your library
dependencies {
	//Reference your local library project
	compile group: 'mic.trade', name: 'gatecoinapi4j', version: '0.1.0-SNAPSHOT'
}
```

