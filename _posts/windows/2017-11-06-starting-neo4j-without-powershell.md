---
title: Starting Neo4j without Powershell
categories: windows neo4j
---

## Starting Neo4j without Powershell

In fact, neo4j is Java based app and so we can start it by using java command as below:

- Starting the console

``` bash

set NEO4J_HOME=C:/neo4j-community-3.3.0

java -cp %NEO4J_HOME%/plugins;%NEO4J_HOME%/conf;%NEO4J_HOME%/lib/*;%NEO4J_HOME%/plugins/* -server -XX:+UseG1GC -XX:-OmitStackTraceInFastThrow -XX:+AlwaysPreTouch -XX:+UnlockExperimentalVMOptions -XX:+TrustFinalNonStaticFields -XX:+DisableExplicitGC -Djdk.tls.ephemeralDHKeySize=2048 -Dunsupported.dbms.udc.source=tarball -Dfile.encoding=UTF-8 org.neo4j.server.CommunityEntryPoint --home-dir=%NEO4J_HOME% --config-dir=%NEO4J_HOME%/conf

```

- Starting the cypher shell

``` bash

set NEO4J_HOME=C:/neo4j-community-3.3.0

java -jar %NEO4J_HOME%/bin/tools/cypher-shell-all.jar

```