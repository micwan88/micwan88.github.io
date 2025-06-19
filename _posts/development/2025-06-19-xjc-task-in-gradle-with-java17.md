---
title: Add Xjc task in gradle with Java 17
categories: java gradle xjc
---

XJC is a tool can be used to generate XML schema object based on XSD schema or WSDL file. However, somehow after Java 11, this tool no more in JDK. 
Also, we need to use 'jakarta' namespace for XML binding in Java 11 or higher as well.

So here is the sample gradle build showing how to include latest XJC ([part of Eclipse Implementation of JAXB](https://eclipse-ee4j.github.io/jaxb-ri/)).
It makes the build to generate the schema object and include those compiled objects automatically.


``` gradle
plugins {
    id 'java'
    //Only put fix version on spring-boot and let SpringBoot manage the dependency version
    id 'org.springframework.boot' version '3.5.0'
}

apply plugin: 'io.spring.dependency-management'

group = 'micwan88.github.com'
version = '1.0.0-SNAPSHOT'

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
}

configurations {
    all {
		exclude group: 'org.springframework.boot', module: 'spring-boot-starter-logging'
	}

    jaxb
}

repositories {
    // Use Maven Central for resolving dependencies.
    mavenCentral()
}

//A task to generate schema object and then copy to particular folder
task genJaxb {
	ext.sourcesDir = "${buildDir}/generated-sources/jaxb"
	ext.classesDir = "${buildDir}/classes/jaxb"
	ext.schema = "schema/myschema.xsd"

	outputs.dir classesDir

	doLast() {
		ant {
			taskdef name: "xjc", classname: "com.sun.tools.xjc.XJCTask", classpath: configurations.jaxb.asPath
			    mkdir(dir: sourcesDir)
			    mkdir(dir: classesDir)

                //Generate schema source via XJC
                //Refer to https://eclipse-ee4j.github.io/jaxb-ri/
				xjc(destdir: sourcesDir, schema: schema, package: "micwan88.github.com.schema") {
					//arg(value: "-wsdl")
					produces(dir: sourcesDir)
				}

                //Compile schema source
				javac(destdir: classesDir, source: java.sourceCompatibility, target: java.targetCompatibility, debug: true,
                    debugLevel: "lines,vars,source",
                    classpath: configurations.jaxb.asPath,
                    includeantruntime: false) {
					src(path: sourcesDir)
					include(name: "**/*.java")
					include(name: "*.java")
				}

                //Copy compiled schema object to class folder
				copy(todir: classesDir) {
                    fileset(dir: sourcesDir, erroronmissingdir: false) {
                        //only copy .class
                        exclude(name: "**/*.java")
                    }
			    }
		}
	}
}

dependencies {
    //Spring boot app
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'org.springframework.boot:spring-boot-starter-log4j2'
    implementation ('org.springframework.boot:spring-boot-starter-web-services') {
		exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
	}
	implementation 'org.springframework.ws:spring-ws-core'
	
    //For calling soap - jaxb xml binding (Java 11 or higher)
	implementation 'jakarta.xml.bind:jakarta.xml.bind-api'

    //For generate schema object via XJC (https://eclipse-ee4j.github.io/jaxb-ri/)
    jaxb 'com.sun.xml.bind:jaxb-impl'
    jaxb 'com.sun.xml.bind:jaxb-jxc'
    //Declare project depends on generated schema object
    implementation(files(genJaxb.classesDir).builtBy(genJaxb))
    
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}
```

References:
- [Eclipse Implementation of JAXB](https://eclipse-ee4j.github.io/jaxb-ri/4.0.5/docs/ch04.html)
- [Create Java Source From WSDL with Java 11 using JAXB](https://medium.com/@jigyasapgandhi/create-java-source-from-wsdl-with-java-11-using-jaxb-8390cf10d554)
