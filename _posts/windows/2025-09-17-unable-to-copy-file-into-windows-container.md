---
title: Unable to copy file into Windows container through docker cp command
categories: windows docker
---

In Windows or saying Windows container, sometimes docker may not work as expected. For example, when you trying to copy some file to a `running` Windows container via `docker cp somefile.txt ltsc2025:C:\`. It gives you an error as below:


```
docker cp somefile.txt ltsc2025:C:\

time="2025-09-17T10:33:15+01:00" level=error msg="Can't add file \\\\?\\C:\\Temp\\somefile.txt to tar: io: read/write on closed pipe"
time="2025-09-17T10:33:15+01:00" level=error msg="Can't close tar writer: io: read/write on closed pipe"
no such directory

```

It is really weird as that command work perfectly for Linux type of container. After I googling on web, I found that someone saying that issue is related to Hyper-V and somehow the copy command only work for Windows container in `stopped` (or it is called `exited`) state. I have tried and it's work !

So what I need to do is just to stop the container, copy the file and then start the container again.

```
docker stop ltsc2025
docker cp somefile.txt ltsc2025:C:\
docker start -i ltsc2025
```

Note: if you think you need copy file into the container after started, you cannot use option of `--rm` in docker run as it will remove the container automatically when you stop it.

References:
- [Unable to copy to Windows container ('docker cp') on Windows 10](https://stackoverflow.com/questions/45654570/unable-to-copy-to-windows-container-docker-cp-on-windows-10)