---
layout: post
date: 2023-12-15 19:55
title: EVE-NG PuTTY & Wireshark Windows Integration
---

# Introduction

I have been using EVE-NG to do some labbing for JNCIS and CCNP studies. I've found that their integration pack installs older versions of `PuTTY` and `Wireshark`. I wanted to use my already installed versions. To do so, I analyzed the provided registry and batch files in the integration pack to see what options need to be changed.

*Note: Please use the following instructions and files at your own discretion. ALWAYS BACKUP YOUR REGISTRY BEFORE MODIFYING IT.*

***

# Prerequisites

- **Windows OS**
- Existing **PuTTY** install
- Existing **Wireshark** install
- Existing **EVE-NG** install

***

# PuTTY Registry File

The only real changes are the file paths used within the files and the location of a wrapper batch file.

Instead of referencing the EVE-NG installed versions, I pointed them at the already installed/official ones.

IE: `C:\Program Files\PuTTY\putty.exe`

Save the below code block in a `.reg` file and install/open it **as admin** to merge it into your registry.

*EVE-NG Putty Registry File*
```text
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\SOFTWARE\Classes\Putty.telnet]
@="telnet"

[HKEY_CURRENT_USER\SOFTWARE\Classes\Putty.telnet\DefaultIcon]
@="C:\\Program Files\\PuTTY\\putty.exe, 0"

[HKEY_CURRENT_USER\SOFTWARE\Classes\Putty.telnet\shell]

[HKEY_CURRENT_USER\SOFTWARE\Classes\Putty.telnet\shell\open]

[HKEY_CURRENT_USER\SOFTWARE\Classes\Putty.telnet\shell\open\command]
@="\"C:\\Program Files\\PuTTY\\putty.exe\" %1"

[HKEY_CURRENT_USER\SOFTWARE\Putty]

[HKEY_CURRENT_USER\SOFTWARE\Putty\Capabilities]

[HKEY_CURRENT_USER\SOFTWARE\Putty\Capabilities\URLAssociations]
"telnet"="Putty.telnet"

[HKEY_CURRENT_USER\SOFTWARE\RegisteredApplications]
"Putty"="Software\\Putty\\Capabilities"

[HKEY_CURRENT_USER\SOFTWARE\Classes\telnet\shell]

[HKEY_CURRENT_USER\SOFTWARE\Classes\telnet\shell\open]

[HKEY_CURRENT_USER\SOFTWARE\Classes\telnet\shell\open\command]
@="\"C:\\Program Files\\PuTTY\\putty.exe\" %1"
```

***

# Wireshark Registry File

Below, the `HKEY_CLASSES_ROOT\capture\shell\open\command` file location can be anywhere you want to reference. I personally just put the file in the PuTTY install location.

Also, save the following code block in a `.reg` file and install/open it **as admin** to merge it into your registry.

*EVE-NG Wireshark Registry File*
```text
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\capture]
@="URL:UNetLab interface capture"
"URL Protocol"=""

[HKEY_CLASSES_ROOT\capture\shell]

[HKEY_CLASSES_ROOT\capture\shell\open]

[HKEY_CLASSES_ROOT\capture\shell\open\command]
@="\"C:\\Program Files\\PuTTY\\wireshark_wrapper.bat\" %1"
```

***

# Wireshark Wrapper Batch File

The next code block is what was previously referenced and is actually saved in the PuTTY install location.

Special Mention:
- The `USERNAME` and `PASSWORD` `SET` commands are your **EVE-NG login credentials**. So whatever you use to SSH into your EVE-NG install and manage the system is what you would use here. I personally use the VM version and provide the VM credentials.

Save the following code block as a `.bat` file with the name and location specified in the Wireshark Registry file above.

*EVE-NG Wireshark Wrapper Batch File*
```cmd
@ECHO OFF
SET USERNAME="root"
SET PASSWORD="eve"

SET S=%1
SET S=%S:capture://=%
FOR /f "tokens=1,2 delims=/ " %%a IN ("%S%") DO SET HOST=%%a&SET INT=%%b
IF "%INT%" == "pnet0" SET FILTER=" not port 22"

ECHO "Connecting to %USERNAME%@%HOST%..."

"C:\Program Files\PuTTY\plink.exe" -ssh -batch -pw %PASSWORD% %USERNAME%@%HOST% "tcpdump -U -i %INT% -s 0 -w -%FILTER%" | "C:\Program Files\Wireshark\Wireshark.exe" -k -i -
```

***

# One Last Thing...

If you have never connected to EVE-NG over SSH, you will need to do so and add the EVE-NG host hey to your **known hosts**. I'm not 100% sure, but I personally just connect with both `putty.exe` and `plink.exe` (installed alongside PuTTY) and add the key. If you do not, you will get an error.

***

# Conclusion

After saving and running the `.reg` files and saving the **Wireshark** `.bat` file in the appropriate location, you will be able to use the Native Console in EVE-NG with your existing Wireshark and PuTTY installations.

***

# Changelog

- **Initial Release**
  - 2023-12-15
- **Add section on connecting to add EVE-NG SSH key to known hosts.**
  - 2023-12-16
