---
layout: blogpost
title:  "Run Linux commands from Codesys"
date:   2022-3-1 16:00:00 +0100
categories: Codesys
tags: Codesys Linux HMI
excerpt_separator: <!--more-->
---
This tutorial explains how to run Linux commands from Codesys. There is a [library in Codesys](https://help.codesys.com/webapp/idx-SysProcess%20Implementation-lib;product=SysProcess%20Implementation;version=3.5.16.0) available that allows to manage processes on the target system. The target system needs to support this.

To test the functionality a [Turck TX707 HMI](https://www.turck.de/en/product/00000044000118d80003003a) with Codesys runtime v3.5.16.30 is used. 

[![CodesysLinux](/assets/img/CodesysLinux.png)](/assets/img/CodesysLinux.png)

<!--more-->

## Configure PLC
First enable SSH access in the system settings of the TX707. This is explained in the [manual](https://www.turck.de/attachment/100002669.pdf) in chapter 8. In the system settings, navigate to `Service settings`, then enable `SSH Server`. Now a program like [Putty](https://www.putty.org/) can be used to access the HMI via SSH.

Login via SSH and navigate to the codesys folder with the following command: (the folder might be different on another PLC)

``` bash
cd /mnt/data/hmi/cds3/deploy
```

_The location may differ on different PLC's. Sometimtes the SysProcess information should be added in a seperate CODESYSControl_User.cfg file. See also the [Codesys FAQ](https://faq.codesys.com/display/CDSFAQ/Location+of+the+configuration+file)_

Now open the Codesys config file with the following command:

``` bash
sudo nano CODESYSControl.cfg
```

Add the following line (in the middle of the file) to give Codesys permission to execute commands:
``` bash
[SysProcess]
BasePriority=Realtime
Command=AllowAll
```

The result should look like this:

[![CodesysSysProcessNano](/assets/img/CodesysSysProcessNano.png)](/assets/img/CodesysSysProcessNano.png)

Press `Ctrl`+`X` to exit the file editor and choose `Y` to save the file. Then press `Enter` to confirm. Now the correct file is created to allow Codesys to execute processes and commands. If only a specific program or command is needed, then this is also possible by writing the following in the file (for example):

``` bash
[SysProcess]
Command.0=echo
Command.1=tar
```

Now only the `echo` and `tar` command can be executed by Codesys.

After changing the config file, reboot the PLC.

## Execute command from Codesys
In Codesys add the `SysProcess Implementation` library in the library manager.

[![CodesysSysProcessLibrary](/assets/img/CodesysSysProcessLibrary.png)](/assets/img/CodesysSysProcessLibrary.png)

Now write a small program that executes a Linux command based on a test variable. For example:

Declaration:
```
PROGRAM PLC_PRG
VAR
	bTest : BOOL;
	testTrigger : Standard.R_TRIG;
	
	sCommand : STRING;
	refCommand : REFERENCE TO STRING;
	sOutput : STRING;
	refOutput : REFERENCE TO STRING;
	
	result : RTS_IEC_RESULT;
END_VAR
```

Implementation:
```
testTrigger(CLK:= bTest, Q=> );

sCommand:= 'echo "Hello world!"';

refCommand REF= sCommand;
refOutput REF= sOutput;

IF testTrigger.Q THEN
	SysProcessExecuteCommand2(pszCommand:= refCommand, pszStdOut:= refOutput, udiStdOutLen:= SIZEOF(sOutput), pResult:= ADR(result));
	
	bTest:= FALSE;
END_IF
```

When going online with Codesys the result should look like this:

[![CodesysEchoHelloWorld](/assets/img/CodesysEchoHelloWorld.png)](/assets/img/CodesysEchoHelloWorld.png)

The command succesfully executed, the result is visible in the sOutput string.

With the same library it's also possible to create or terminate processes. It's also possible to execute Python scripts, bash scripts or gzipping files. Everything that's possible with a command via SSH is now possible to do dynamically with Codesys.