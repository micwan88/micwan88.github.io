---
title: Default legacy ANSI codepage (ACP) is always UTF-8(65001) under LTSC2019
categories: windows docker hyperv
---

## Default legacy ANSI codepage problem

Something weird about Windows container if the image is based on `ltsc2019`...
While you are running it with `--isolation=hyperv` option, you will found that the `default` legacy ANSI codepage (ACP) is always UTF-8 (65001) no matter what you have going to change by using `Set-WinSystemLocale`, `Set-Culture`, `chcp` or even updating registry.

Here you can get the proof by running the LTSC2019 with HyperV isolation:
```
docker run -it --rm --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 PowerShell
```

After started the powershell, checking the GetACP() return, it should be 65001:
```
Add-Type -TypeDefinition @"
using System;
using System.Runtime.InteropServices;
public class AcpHelper {
    [DllImport("kernel32.dll")]
    public static extern int GetACP();
}
"@

[AcpHelper]::GetACP()

#It should return 65001
```
Same result can be checked by below command as well (Both return default legacy ANSI codepage (ACP) and that is for legacy system):
```
[System.Text.Encoding]::Default.CodePage

#Should return 65001 as well
```

After spent some time to google and found that is Windows container issue and it has been fixed since LTSC2022. 
So below command should return 1251 instead of 65001:

```
docker run -it --rm --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2022 PowerShell

#After powershell has been launch, try below command, it should return 1251
[System.Text.Encoding]::Default.CodePage
```

### Additional Note
- The problem happen with `isolation=hyperV`, if your host is LTSC2019 same as the image and the host's lanague setting is 1251. The same setting will bring into the container(as isolation=process means shared the kernel from host).
- The problem also happen when you running the image (LTSC2019) under Windows 10 / 11 using Docker Desktop, because it is `isolation=hyperV`by default in Windows 10 / 11.


References:
- [ANSI Code Page in Hyper-V Windows containers should not be 65001](https://github.com/microsoft/Windows-Containers/issues/579)
- [Windows 10 Pro - Docker Container System Locale](https://stackoverflow.com/questions/61769729/windows-10-pro-docker-container-system-locale)