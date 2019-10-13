---
title: Use system default proxy in Java app
categories: java
---

To enable Java app to use system default proxy (i.e. socks or http[s]), just add below JVM param during startup.

```
-Djava.net.useSystemProxies=true
```


## Specify proxy setting in Java app

To specify the proxy setting rather than use the system's default, just set the corresponding JVM params as below:

- For HTTP

```
#Set proxy host address
-Dhttp.proxyHost=proxyserver.host.com
#Set proxy port (default is 80)
-Dhttp.proxyPort=80
#To exclude some domains for proxy
-Dhttp.nonProxyHosts="some.internaldomain.com|www.internaldomain.com"
```


- For HTTPS

```
#Set proxy host address
-Dhttps.proxyHost=proxyserver.host.com
#Set proxy port (default is 443)
-Dhttps.proxyPort=443
#To exclude some domains for proxy (It is same as http)
-Dhttp.nonProxyHosts="some.internaldomain.com|www.internaldomain.com"
```


- For FTP

```
#Set proxy host address
-Dftp.proxyHost=proxyserver.host.com
#Set proxy port (default is 80)
-Dftp.proxyPort=80
#To exclude some domains for proxy
-Dftp.nonProxyHosts="some.internaldomain.com|www.internaldomain.com"
```


- For Socks

```
#Set proxy host address
-DsocksProxyHost=proxyserver.host.com
#Set proxy port (default is 1080)
-DsocksProxyPort=1080
```


References:
- [Java Networking and Proxies](https://docs.oracle.com/javase/8/docs/technotes/guides/net/proxies.html)
