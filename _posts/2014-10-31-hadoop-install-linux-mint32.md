---
layout: post
title: "Installing Hadoop on Linux Mint"
author: "RP"
date: "2014/10/31"
output:
  html_document:
    keep_md: true
comments: True
tags: [hadoop, big data, linux]
---
In this post I list steps to install Hadoop on LinuxMint 17. After installing Hadoop I also list the steps to set up a single-node cluster. My system is a laptop running Windows 7 Home Premium. On this laptop Linux Mint is hosted on Oracle's VirtualBox VM 4.3.16r59572 which is itself hosted on Windows 7. I am a Hadoop beginner and a relatively light user (less experienced?) of Linux and my attempt here is to document the installation and set up process in detail to help the reader understand the purpose and motivations for some of the steps in the installation process.

The installation process documented here is a synthesis of steps and neat ideas from a few resources on the web that I discovered during my research. Those resources are included in the References section at the end of this post.

Hadoop requires Java 1.5 (aka Java 5) and therefore the installation process described here consists of two parts: Part I covers Java installation, and Part II covers Hadoop installation.
###Part I - Java Installation
Login to your Linux user account and open up a terminal window. Menu → Terminal or Menu → System Tools → Terminal. In this installation the Linux user account name is `user1`.
The first step before installing the Java Development Kit (JDK) is to determine the system architecture of the system: is it a 32-bit (x86) system or a 64-bit (x64) system? This is determined by using the uname command which prints the system information.

```
user1@user1-Vbox ~ $ uname -m
user1@user1-VBox ~ $ uname --kernel-name --processor
Linux i686
user1@user1-Vbox ~ $ 
```

The lscpu command yields more user-friendly information.

```
user1@user1-VBox ~ $ lscpu
Architecture:          i686
CPU op-mode(s):        32-bit
Byte Order:            Little Endian
CPU(s):                1
On-line CPU(s) list:   0
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 23
Stepping:              10
CPU MHz:               2107.042
BogoMIPS:              4214.08
user1@user1-VBox ~ $ 
```
i686 refers to the Intel P6 microarchitecture. The first two rows of the response to lscpu command mean that the kernel is 32-bit (Architecture: i686) and that the CPU is set to operate in 32-bit instruction width mode (CPU op-mode(s): 32-bit). So, from the perspective of Linux Mint17, my laptop is a 32-bit system. On a 64-bit system the command will return a string containing '64', for example, x86\_64 (for an Intel CPU) or amd64 (for a 64-bit AMD CPU).
<span style="text-decoration: underline;">Note</span>: _On my laptop the Virtual Box VM runs in 32-bit mode and so does Linux Mint17. But Windows7 on my laptop reports System Type as 64-bit Operating System. Why then are Virtual Box, and consequently Linux Mint17, running in 32-bit mode? The answer is that it is hardware capability, not Windows, that determines the mode a VM runs in.  For a VM to run in 64-bit mode the underlying hardware, specifically the CPU, must be 64-bit AND capable of virtualization support.  Additionally, the virtualization support must be enabled.  The processor in my laptop is an Intel Core2 Duo T6600 CPU which is a 64-bit CPU but does not support virtualization.  Indeed, running the [Microsoft Hardware-Assisted Virtualization Detection Tool](http://www.microsoft.com/en-us/download/details.aspx?id=592" target="_blank") on my laptop confirms this because it reports back the following: "This computer does not have hardware-assisted virtualization."_

The latest version of Java (JDK), as of mid-October 2014, is JDK 8. Download the appropriate JDK binary for your system from [Orcale](http://www.oracle.com/technetwork/java/javase/downloads/index.html). Since mine is a 32-bit system, I download file jdk-8u25-linux-i586.tar.gz and save it into the Downloads directory (`/home/user1/Downloads`). For a 64-bit system the corresponding file to download would be jdk-8u25-linux-x64.tar.gz. The remaining installation instructions apply the same way for a 64-bit system as for the 32-bit system.
<span style="text-decoration: underline;">Note</span>: _This is not an RPM installation, so be careful to download only the .tar.gz file._

First, remove current installation, if any, of OpenJDK, IcedTea, and any Java stuff unrelated to Oracle Java.

```
user1@user1-Vbox ~ $ sudo apt-get update
user1@user1-Vbox ~ $ sudo apt-get remove openjdk* icedtea*
user1@user1-Vbox ~ $ sudo apt-get remove default-jre default-jdk gcj-jre gcj-jdk
user1@user1-Vbox ~ $ sudo apt-get remove icedtea-{6..7}-plugin icedtea-plugin

# clean up
user1@user1-Vbox ~ $ sudo apt-get autoremove
user1@user1-Vbox ~ $ sudo apt-get autoclean
```

Install Java in `/opt/java`. Create the java directory under `/opt`, change directory to `/opt/java`, relocate the downloaded JDK tar file from Downloads directory to `/opt/java` directory, and extract the files.

```
user1@user1-Vbox ~ $ sudo mkdir -p /opt/java
user1@user1-Vbox ~ $ cd /opt/java
user1@user1-Vbox ~ $ sudo cp /home/user1/Downloads/jdk-8u25-linux-i586.tar.gz .
user1@user1-Vbox ~ $ sudo tar -zxvf jdk-8u25-linux-i586.tar.gz
```

Create a sym-link called `current-java` to the just-created jdk1.8.0_25 directory. When it is time to upgrade JDK this sym-link will help simplify the upgrade process -- fewer steps will be required during upgrade compared to first-time installation.

```
user1@user1-Vbox /opt/java $ sudo rm -f /opt/java/current-java
user1@user1-Vbox /opt/java $ sudo ln -s /opt/java/jdk1.8.0_25 /opt/java/current-java
```

Now current-java points to jdk1.8.0_25. When it is time to upgrade, simply delete current-java and re-create a sym-link to the newer version JDK folder.

Verify that the symbolic link has been created.

```
user1@user1-Vbox /opt/java $ ls
current-java  jdk1.8.0_25  jdk-8u25-linux-i586.tar.gz
```

Update the system Java settings. The following set of steps need be executed only one time - during initial installation. These steps are not required during upgrade. Since several commands need to be executed, it is best to create a file that contains all those commands and then run the file. Let's call this file updt_java_settings.cmds. Run these commands to create the file.

```
user1@user1-Vbox /opt/java $ su
user1-Vbox java # cd /opt/java
user1-Vbox java # touch update_java_settings.cmds       # create the file
user1-Vbox java # chmod +x update_java_settings.cmds    # set execute permissions
```

The text below is the set of commands to update Java system settings. Copy the full text below to the clipboard.

```
update-alternatives --install /usr/bin/jexec jexec /opt/java/current-java/lib/jexec 1065 &amp;&amp; sleep 1s
update-alternatives --set jexec /opt/java/current-java/lib/jexec &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/appletviewer appletviewer /opt/java/current-java/bin/appletviewer 1065 &amp;&amp; sleep 1s
update-alternatives --set appletviewer /opt/java/current-java/bin/appletviewer &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/apt apt /opt/java/current-java/bin/apt 1065 &amp;&amp; sleep 1s
update-alternatives --set apt /opt/java/current-java/bin/apt &amp;&amp; sleep 1s
# the file /opt/java/current-java/bin/ControlPanel was ignored
update-alternatives --install /usr/bin/extcheck extcheck /opt/java/current-java/bin/extcheck 1065 &amp;&amp; sleep 1s
update-alternatives --set extcheck /opt/java/current-java/bin/extcheck &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/idlj idlj /opt/java/current-java/bin/idlj 1065 &amp;&amp; sleep 1s
update-alternatives --set idlj /opt/java/current-java/bin/idlj &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jar jar /opt/java/current-java/bin/jar 1065 &amp;&amp; sleep 1s
update-alternatives --set jar /opt/java/current-java/bin/jar &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jarsigner jarsigner /opt/java/current-java/bin/jarsigner 1065 &amp;&amp; sleep 1s
update-alternatives --set jarsigner /opt/java/current-java/bin/jarsigner &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/java java /opt/java/current-java/bin/java 1065 &amp;&amp; sleep 1s
update-alternatives --set java /opt/java/current-java/bin/java &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/javac javac /opt/java/current-java/bin/javac 1065 &amp;&amp; sleep 1s
update-alternatives --set javac /opt/java/current-java/bin/javac &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/javadoc javadoc /opt/java/current-java/bin/javadoc 1065 &amp;&amp; sleep 1s
update-alternatives --set javadoc /opt/java/current-java/bin/javadoc &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/javafxpackager javafxpackager /opt/java/current-java/bin/javafxpackager 1065 &amp;&amp; sleep 1s
update-alternatives --set javafxpackager /opt/java/current-java/bin/javafxpackager &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/javah javah /opt/java/current-java/bin/javah 1065 &amp;&amp; sleep 1s
update-alternatives --set javah /opt/java/current-java/bin/javah &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/javap javap /opt/java/current-java/bin/javap 1065 &amp;&amp; sleep 1s
update-alternatives --set javap /opt/java/current-java/bin/javap &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/java-rmi.cgi java-rmi.cgi /opt/java/current-java/bin/java-rmi.cgi 1065 &amp;&amp; sleep 1s
update-alternatives --set java-rmi.cgi /opt/java/current-java/bin/java-rmi.cgi &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/javaws javaws /opt/java/current-java/bin/javaws 1065 &amp;&amp; sleep 1s
update-alternatives --set javaws /opt/java/current-java/bin/javaws &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jcmd jcmd /opt/java/current-java/bin/jcmd 1065 &amp;&amp; sleep 1s
update-alternatives --set jcmd /opt/java/current-java/bin/jcmd &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jconsole jconsole /opt/java/current-java/bin/jconsole 1065 &amp;&amp; sleep 1s
update-alternatives --set jconsole /opt/java/current-java/bin/jconsole &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jcontrol jcontrol /opt/java/current-java/bin/jcontrol 1065 &amp;&amp; sleep 1s
update-alternatives --set jcontrol /opt/java/current-java/bin/jcontrol &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jdb jdb /opt/java/current-java/bin/jdb 1065 &amp;&amp; sleep 1s
update-alternatives --set jdb /opt/java/current-java/bin/jdb &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jhat jhat /opt/java/current-java/bin/jhat 1065 &amp;&amp; sleep 1s
update-alternatives --set jhat /opt/java/current-java/bin/jhat &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jinfo jinfo /opt/java/current-java/bin/jinfo 1065 &amp;&amp; sleep 1s
update-alternatives --set jinfo /opt/java/current-java/bin/jinfo &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jmap jmap /opt/java/current-java/bin/jmap 1065 &amp;&amp; sleep 1s
update-alternatives --set jmap /opt/java/current-java/bin/jmap &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jmc jmc /opt/java/current-java/bin/jmc 1065 &amp;&amp; sleep 1s
update-alternatives --set jmc /opt/java/current-java/bin/jmc &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jmc.ini jmc.ini /opt/java/current-java/bin/jmc.ini 1065 &amp;&amp; sleep 1s
update-alternatives --set jmc.ini /opt/java/current-java/bin/jmc.ini &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jps jps /opt/java/current-java/bin/jps 1065 &amp;&amp; sleep 1s
update-alternatives --set jps /opt/java/current-java/bin/jps &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jrunscript jrunscript /opt/java/current-java/bin/jrunscript 1065 &amp;&amp; sleep 1s
update-alternatives --set jrunscript /opt/java/current-java/bin/jrunscript &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jsadebugd jsadebugd /opt/java/current-java/bin/jsadebugd 1065 &amp;&amp; sleep 1s
update-alternatives --set jsadebugd /opt/java/current-java/bin/jsadebugd &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jstack jstack /opt/java/current-java/bin/jstack 1065 &amp;&amp; sleep 1s
update-alternatives --set jstack /opt/java/current-java/bin/jstack &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jstat jstat /opt/java/current-java/bin/jstat 1065 &amp;&amp; sleep 1s
update-alternatives --set jstat /opt/java/current-java/bin/jstat &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jstatd jstatd /opt/java/current-java/bin/jstatd 1065 &amp;&amp; sleep 1s
update-alternatives --set jstatd /opt/java/current-java/bin/jstatd &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/jvisualvm jvisualvm /opt/java/current-java/bin/jvisualvm 1065 &amp;&amp; sleep 1s
update-alternatives --set jvisualvm /opt/java/current-java/bin/jvisualvm &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/keytool keytool /opt/java/current-java/bin/keytool 1065 &amp;&amp; sleep 1s
update-alternatives --set keytool /opt/java/current-java/bin/keytool &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/native2ascii native2ascii /opt/java/current-java/bin/native2ascii 1065 &amp;&amp; sleep 1s
update-alternatives --set native2ascii /opt/java/current-java/bin/native2ascii &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/orbd orbd /opt/java/current-java/bin/orbd 1065 &amp;&amp; sleep 1s
update-alternatives --set orbd /opt/java/current-java/bin/orbd &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/pack200 pack200 /opt/java/current-java/bin/pack200 1065 &amp;&amp; sleep 1s
update-alternatives --set pack200 /opt/java/current-java/bin/pack200 &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/policytool policytool /opt/java/current-java/bin/policytool 1065 &amp;&amp; sleep 1s
update-alternatives --set policytool /opt/java/current-java/bin/policytool &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/rmic rmic /opt/java/current-java/bin/rmic 1065 &amp;&amp; sleep 1s
update-alternatives --set rmic /opt/java/current-java/bin/rmic &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/rmid rmid /opt/java/current-java/bin/rmid 1065 &amp;&amp; sleep 1s
update-alternatives --set rmid /opt/java/current-java/bin/rmid &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/rmiregistry rmiregistry /opt/java/current-java/bin/rmiregistry 1065 &amp;&amp; sleep 1s
update-alternatives --set rmiregistry /opt/java/current-java/bin/rmiregistry &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/schemagen schemagen /opt/java/current-java/bin/schemagen 1065 &amp;&amp; sleep 1s
update-alternatives --set schemagen /opt/java/current-java/bin/schemagen &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/serialver serialver /opt/java/current-java/bin/serialver 1065 &amp;&amp; sleep 1s
update-alternatives --set serialver /opt/java/current-java/bin/serialver &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/servertool servertool /opt/java/current-java/bin/servertool 1065 &amp;&amp; sleep 1s
update-alternatives --set servertool /opt/java/current-java/bin/servertool &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/tnameserv tnameserv /opt/java/current-java/bin/tnameserv 1065 &amp;&amp; sleep 1s
update-alternatives --set tnameserv /opt/java/current-java/bin/tnameserv &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/unpack200 unpack200 /opt/java/current-java/bin/unpack200 1065 &amp;&amp; sleep 1s
update-alternatives --set unpack200 /opt/java/current-java/bin/unpack200 &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/wsgen wsgen /opt/java/current-java/bin/wsgen 1065 &amp;&amp; sleep 1s
update-alternatives --set wsgen /opt/java/current-java/bin/wsgen &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/wsimport wsimport /opt/java/current-java/bin/wsimport 1065 &amp;&amp; sleep 1s
update-alternatives --set wsimport /opt/java/current-java/bin/wsimport &amp;&amp; sleep 1s
update-alternatives --install /usr/bin/xjc xjc /opt/java/current-java/bin/xjc 1065 &amp;&amp; sleep 1s
update-alternatives --set xjc /opt/java/current-java/bin/xjc &amp;&amp; sleep 1s
# system config for /opt/java/current-java/jre/bin/
# the file /opt/java/current-java/bin/ControlPanel was ignored
update-alternatives --install /usr/bin/java_vm java_vm /opt/java/current-java/jre/bin/java_vm 1065 &amp;&amp; sleep 1s
update-alternatives --set java_vm /opt/java/current-java/jre/bin/java_vm &amp;&amp; sleep 1s
```
Open updt\_java\_settings.cmds in your favorite editor and paste the clipboard. Save the file, exit out of the editor and execute all the commands.

```
user1-Vbox java # /bin/bash updt_java_settings.cmds
```

The Java installation is now complete. Verify that Java has installed properly.

```
user1-Vbox java # java -version
java version "1.8.0_25"
Java(TM) SE Runtime Environment (build 1.8.0_25-b17)
Java HotSpot(TM) Client VM (build 25.25-b02, mixed mode)
```

Set up environment variables for use with the new Java installation. These variables could be set up for each user or for the whole system. For individual user the changes can be made to `~/.bashrc` and for system overall, to `/etc/bash.bashrc`. Copy text below into the appropriate file.

```
### $CLASS_PATH
CLASS_PATH="$CLASS_PATH"
if ! [[ "$CLASS_PATH" =~ ^:*(.*:+)*\.{1}(:+.*)*$ ]]; then
CLASS_PATH="$CLASS_PATH:."
fi
if ! [[ "$CLASS_PATH" =~ ^:*(.*:+)*\.{2}(:+.*)*$ ]]; then
CLASS_PATH="$CLASS_PATH:.."
fi
export CLASS_PATH

### $JAVA_HOME
: ${JAVA_HOME:=/opt/java/current-java}
export JAVA_HOME

### $PATH
export PATH="$PATH:$JAVA_HOME/bin"
```

The above environment variables will take effect upon rebooting the system or sourcing the newly modified `.bashrc` file. Create sym-links to the Java manpages. The following steps till the end of this section are optional.

```
user1@user1-Vbox /opt/java $ su
user1-Vbox java # cd /opt/java
user1-Vbox java # touch manpagelinker.cmds       # create the file
user1-Vbox java # chmod +x manpagelinker.cmds    # set execute permissions
```

Open the manpagelinker.cmds file in your text editor, copy-paste the text below into it, and save the file.

```
ln -s /opt/java/current-java/man/man1/appletviewer.1 /usr/share/man/man1/appletviewer.1
ln -s /opt/java/current-java/man/man1/apt.1 /usr/share/man/man1/apt.1
ln -s /opt/java/current-java/man/man1/extcheck.1 /usr/share/man/man1/extcheck.1
ln -s /opt/java/current-java/man/man1/idlj.1 /usr/share/man/man1/idlj.1
ln -s /opt/java/current-java/man/man1/jar.1 /usr/share/man/man1/jar.1
ln -s /opt/java/current-java/man/man1/jarsigner.1 /usr/share/man/man1/jarsigner.1
ln -s /opt/java/current-java/man/man1/java.1 /usr/share/man/man1/java.1
ln -s /opt/java/current-java/man/man1/javac.1 /usr/share/man/man1/javac.1
ln -s /opt/java/current-java/man/man1/javadoc.1 /usr/share/man/man1/javadoc.1
ln -s /opt/java/current-java/man/man1/javafxpackager.1 /usr/share/man/man1/javafxpackager.1
ln -s /opt/java/current-java/man/man1/javah.1 /usr/share/man/man1/javah.1
ln -s /opt/java/current-java/man/man1/javap.1 /usr/share/man/man1/javap.1
ln -s /opt/java/current-java/man/man1/javaws.1 /usr/share/man/man1/javaws.1
ln -s /opt/java/current-java/man/man1/jcmd.1 /usr/share/man/man1/jcmd.1
ln -s /opt/java/current-java/man/man1/jconsole.1 /usr/share/man/man1/jconsole.1
ln -s /opt/java/current-java/man/man1/jdb.1 /usr/share/man/man1/jdb.1
ln -s /opt/java/current-java/man/man1/jhat.1 /usr/share/man/man1/jhat.1
ln -s /opt/java/current-java/man/man1/jinfo.1 /usr/share/man/man1/jinfo.1
ln -s /opt/java/current-java/man/man1/jmap.1 /usr/share/man/man1/jmap.1
ln -s /opt/java/current-java/man/man1/jmc.1 /usr/share/man/man1/jmc.1
ln -s /opt/java/current-java/man/man1/jps.1 /usr/share/man/man1/jps.1
ln -s /opt/java/current-java/man/man1/jrunscript.1 /usr/share/man/man1/jrunscript.1
ln -s /opt/java/current-java/man/man1/jsadebugd.1 /usr/share/man/man1/jsadebugd.1
ln -s /opt/java/current-java/man/man1/jstack.1 /usr/share/man/man1/jstack.1
ln -s /opt/java/current-java/man/man1/jstat.1 /usr/share/man/man1/jstat.1
ln -s /opt/java/current-java/man/man1/jstatd.1 /usr/share/man/man1/jstatd.1
ln -s /opt/java/current-java/man/man1/jvisualvm.1 /usr/share/man/man1/jvisualvm.1
ln -s /opt/java/current-java/man/man1/keytool.1 /usr/share/man/man1/keytool.1
ln -s /opt/java/current-java/man/man1/native2ascii.1 /usr/share/man/man1/native2ascii.1
ln -s /opt/java/current-java/man/man1/orbd.1 /usr/share/man/man1/orbd.1
ln -s /opt/java/current-java/man/man1/pack200.1 /usr/share/man/man1/pack200.1
ln -s /opt/java/current-java/man/man1/policytool.1 /usr/share/man/man1/policytool.1
ln -s /opt/java/current-java/man/man1/rmic.1 /usr/share/man/man1/rmic.1
ln -s /opt/java/current-java/man/man1/rmid.1 /usr/share/man/man1/rmid.1
ln -s /opt/java/current-java/man/man1/rmiregistry.1 /usr/share/man/man1/rmiregistry.1
ln -s /opt/java/current-java/man/man1/schemagen.1 /usr/share/man/man1/schemagen.1
ln -s /opt/java/current-java/man/man1/serialver.1 /usr/share/man/man1/serialver.1
ln -s /opt/java/current-java/man/man1/servertool.1 /usr/share/man/man1/servertool.1
ln -s /opt/java/current-java/man/man1/tnameserv.1 /usr/share/man/man1/tnameserv.1
ln -s /opt/java/current-java/man/man1/unpack200.1 /usr/share/man/man1/unpack200.1
ln -s /opt/java/current-java/man/man1/wsgen.1 /usr/share/man/man1/wsgen.1
ln -s /opt/java/current-java/man/man1/wsimport.1 /usr/share/man/man1/wsimport.1
ln -s /opt/java/current-java/man/man1/xjc.1 /usr/share/man/man1/xjc.1
```

Now create sym-links to the manpages.

```
user1-Vbox java # /bin/bash manpagelinker.cmds
```

Java installation is now complete. Next we start with Part II, Hadoop Installation.

###Part II - Hadoop Installation
This part of the installation assumes that Java has been successfully installed and verified.

Create a new user group for Hadoop and create a new user account for Hadoop programming. This provides the benefit of keeping the Hadoop installation and user account(s) separate from other software installations running on the system. For example, security, permissions, data storage, etc. can be managed independently for Hadoop-related activities.

```
user1@user1-VBox ~ $ cd ~
user1@user1-VBox ~ $ sudo addgroup hadoop
user1@user1-VBox ~ $ sudo adduser --ingroup hadoop hduser1
```

####Install and Configure SSH
Hadoop requires communication between multiple processes running on one or more machines which, in turn, implies that the user needs to be able to connect to the various hosts without a password. This is achieved by using Secure Shell (SSH). We use OpenSSH (the free version of SSH connectivity tools). The setup requires three steps:
&nbsp;&nbsp;&nbsp;i. install OpenSSH,
&nbsp;&nbsp;&nbsp;ii. generate SSH key pair with empty passphrase, and
&nbsp;&nbsp;&nbsp;iii. enable access to your local machine with the newly created keys

In the main user account (<code>user1</code> in our setup) install OpenSSH.

```
user1@user1-VBox ~ $ sudo apt-get install openssh-server
```

Generate the SSH key pair.

```
user1@user1-VBox ~ $ su hduser1
hduser1@user1-VBox /home/user1 $ ssh-keygen -t rsa -P ""
```

Simply hit Enter to accept the default filename in which to save the key. This will create directory `/home/hduser1/.ssh` and save a public key in file `rsa.pub` in that directory. Next, add this public key to the list of authorized keys to enable access to the local machine.

```
hduser1@user1-VBox /home/user1 $ cat $HOME/.ssh/id_rsa.pub &gt;&gt; $HOME/.ssh/authorized_keys
```

Test the SSH setup (by issuing command below) by connecting to the local machine from `hduser1`. This also saves the host fingerprint of the local machine to list of hosts known to user <code>hduser1</code>. Specifically, the local machine host fingerprint is added to `hduser1`'s known_hosts file.

```
hduser1@user1-VBox /home/user1 $ ssh localhost
```

Enter yes to the question “Are you sure you want to continue connecting (yes/no)?” This step confirms to user `hduser1` that the host certificate presented by the local machine can be trusted. This completes SSH installation and configuration. Type exit (or `Ctrl-d`) to close the connection to the local host.

####Install Hadoop
Login to the main user account (`user1` in our case). Download the latest Hadoop binaries from [Apache](http://www.apache.org/dyn/closer.cgi/hadoop/common/). Click on the suggested mirror, then click on the 'stable' directory, and finally on the link to the hadoop-x.y.z.tar.gz file, where x.y.z represents the Hadoop version number. As of mid-October 2014 the stable Hadoop version is 2.5.1.  Save the file into the local 'Downloads' directory.

```
user1@user1-VBox ~ $ cd Downloads/
user1@user1-VBox ~/Downloads $ sudo tar zxvf hadoop-2.5.1.tar.gz
```

Relocate the Hadoop files to /usr/local, create a sym-link, and change ownership of the Hadoop directories and files to the hadoop group and `hduser1`.

```
user1@user1-VBox ~/Downloads $ sudo mv hadoop-2.5.1 /usr/local
user1@user1-VBox ~/Downloads $ cd /usr/local/
user1@user1-VBox /usr/local $ sudo ln -s hadoop-2.5.1 hadoop
user1@user1-VBox /usr/local $ sudo chown -R hduser1:hadoop hadoop
user1@user1-VBox /usr/local $ sudo chown -R hduser1:hadoop hadoop-2.5.1
```

####Configure the Environment
In a freshly created user account on Linux Mint there is no `.bashrc` file. It needs to be created and then edited to add commands to configure the environment.

```
user1@user1-VBox ~ $ su hduser1
hduser1@user1-VBox /home/user1 $ cd ~
hduser1@user1-VBox ~ $ touch .bashrc
```

Open `.bashrc` in your text editor and add the following lines.

```
# Run the .bashrc file in /etc first if it exists
if [ -f /etc/bash.bashrc ]; then
. /etc/bash.bashrc
fi

# Hadoop variables
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib"

export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin

export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
```

<span style="text-decoration: underline;">Notes</span>:

1. _Environment variable JAVA_HOME is not defined or exported here because those actions happen at the system level via `/etc/bash.bashrc`._
2. _Setting and exporting env variable $HADOOP_COMMON_LIB_NATIVE_DIR fixes the warning shown below when Hadoop is started (via start-dfs.sh) or stopped (via stop-dfs.sh)._

     _Java HotSpot(TM) Client VM warning: You have loaded library /usr/local/hadoop-2.5.1/lib/native/libhadoop.so.1.0.0 which might have disabled stack guard. The VM will try to fix the stack guard now.
It's highly recommended that you fix the library with 'execstack -c ', or link it with '-z noexecstack'._

Verify that Hadoop is installed properly.

```
hduser1@user1-VBox ~ $ source .bashrc
hduser1@user1-VBox ~ $ hadoop version
Hadoop 2.5.1
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r 2e18d179e4a8065b6a9f29cf2de9451891265cce
Compiled by jenkins on 2014-09-05T23:11Z
Compiled with protoc 2.5.0
From source with checksum 6424fcab95bfff8337780a181ad7c78
This command was run using /usr/local/hadoop-2.5.1/share/hadoop/common/hadoop-common-2.5.1.jar
hduser1@user1-VBox ~ $
```

####Configure Hadoop
Configure the directories Hadoop uses to store data, etc. By default, Hadoop uses hadoop.tmp.dir as temporary storage location for the local filesystem and for HDFS. Create the temporary storage area. Also create storage for hdfs to store namenode and datanode data. But first get back to the main user account (i.e., return to <code>user1</code>, since <code>hduser1</code> does not have the required permissions for the next steps).

```
hduser1@user1-VBox ~ $ exit
user1@user1-VBox ~ $ sudo mkdir -p /app/hadoop/tmp
user1@user1-VBox ~ $ sudo mkdir -p /app/hadoop/usr/hduser1/data/hdfs/namenode
user1@user1-VBox ~ $ sudo mkdir -p /app/hadoop/usr/hduser1/data/hdfs/datanode
```

Change ownership of the directories to the hadoop group and change permissions to tighten security.

```
user1@user1-VBox ~ $ sudo chown hduser1:hadoop /app/hadoop/tmp
user1@user1-VBox ~ $ sudo chown -R hduser1:hadoop /app/hadoop/usr/hduser1
user1@user1-VBox ~ $ sudo chmod 750 /app/hadoop/tmp
user1@user1-VBox ~ $ sudo chmod -R 750 /app/hadoop/usr/hduser1
```

Now configure these four .xml files: `core-site.xml`, `yarn-site.xml`, `mapred-site.xml`, and `hdfs-site.xml`. By default, file `mapred-site.xml` is not present. It can be created from `mapred-site.xml.template`.

```
user1@user1-VBox ~ $ cd /usr/local/hadoop/etc/hadoop/
user1@user1-VBox /usr/local/hadoop/etc/hadoop $ sudo cp mapred-site.xml.template mapred-site.xml
```

Open each .xml file using your favorite editor, copy the text below for the corresponding file, paste it between the and tags, and save and exit the file.

<strong>coresite.xml</strong>
<pre><strong>&lt;property&gt;</strong>
  <strong>&lt;name&gt;</strong>hadoop.tmp.dir<strong>&lt;/name&gt;</strong>
  <strong>&lt;value&gt;</strong>/app/hadoop/tmp<strong>&lt;/value&gt;</strong>
  <strong>&lt;description&gt;</strong>A base for other temporary directories.<strong>&lt;/description&gt;</strong>
<strong>&lt;/property&gt;</strong>
<strong>&lt;property&gt;</strong>
  <strong>&lt;name&gt;</strong>fs.default.name<strong>&lt;/name&gt;</strong>
  <strong>&lt;value&gt;</strong>hdfs://localhost:54310<strong>&lt;/value&gt;</strong>
  <strong>&lt;description&gt;</strong>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  URI's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The URI's authority is used to
  determine the host, port, etc. for a filesystem.<strong>&lt;/description&gt;</strong>
<strong>&lt;/property&gt;</strong>
</pre>
<strong>yarn.xml</strong>
<pre><strong>&lt;property&gt;</strong>
  <strong>&lt;name&gt;</strong>yarn.nodemanager.aux-services<strong>&lt;/name&gt;</strong>
  <strong>&lt;value&gt;</strong>mapreduce\_shuffle<strong>&lt;/value&gt;</strong>
<strong>&lt;/property&gt;</strong>
<strong>&lt;property&gt;</strong>
  <strong>&lt;name&gt;</strong>yarn.nodemanager.aux-services.mapreduce.shuffle.class<strong>&lt;/name&gt;</strong>
  <strong>&lt;value&gt;</strong>org.apache.hadoop.mapred.ShuffleHandler<strong>&lt;/value&gt;</strong>
<strong>&lt;/property&gt;</strong>
</pre>
<strong>mapred.xml</strong>
<pre><strong>&lt;property&gt;</strong>
  <strong>&lt;name&gt;</strong>mapreduce.framework.name<strong>&lt;/name&gt;</strong>
  <strong>&lt;value&gt;</strong>yarn<strong>&lt;/value&gt;</strong>
<strong>&lt;/property&gt;</strong>
</pre>
<strong>hdfs.xml</strong>
<pre><strong>&lt;property&gt;</strong>
  <strong>&lt;name&gt;</strong>dfs.replication<strong>&lt;/name&gt;</strong>
  <strong>&lt;value&gt;</strong>1<strong>&lt;/value&gt;</strong>
  <strong>&lt;description&gt;</strong>Default block replication.
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.<strong>&lt;/description&gt;</strong>
<strong>&lt;/property&gt;</strong>
<strong>&lt;property&gt;</strong>
  <strong>&lt;name&gt;</strong>dfs.namenode.name.dir<strong>&lt;/name&gt;</strong>
  <strong>&lt;value&gt;</strong>file:///home/hduser1/data/hdfs/namenode<strong>&lt;/value&gt;</strong>
<strong>&lt;/property&gt;</strong>
<strong>&lt;property&gt;</strong>
  <strong>&lt;name&gt;</strong>dfs.datanode.data.dir<strong>&lt;/name&gt;</strong>
  <strong>&lt;value&gt;</strong>file:///home/hduser1/data/hdfs/datanode<strong>&lt;/value&gt;</strong>
<strong>&lt;/property&gt;</strong>
</pre>
<span style="text-decoration: underline;">Note</span>: _In the text above, the value of the namenode and datanode storage directories must be proper URIs, and therefore three forward slash ('/') characters are required, not just two (i.e., the third forward slash is not a typo)._
####Format HDFS
This step is similar to formatting any new storage (e.g., a USB personal drive) prior to use. But first switch to the Hadoop user account (`hduser1`)

```
user1@user1-VBox ~ $ su hduser1
hduser1@user1-VBox /home/user1 $ cd ~
hduser1@user1-VBox ~ $ /usr/local/hadoop/bin/hdfs namenode -format
```

If the formatting goes through properly the output will look something like this

```
hduser1@user1-VBox $ /usr/local/hadoop/bin/hdfs namenode -format
14/10/17 12:23:50 INFO namenode.NameNode: STARTUP_MSG:
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = user1-VBox/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.5.1
STARTUP_MSG:   classpath = /usr/local/hadoop-

STARTUP_MSG:   build = https://git-wip-us.apache.org/repos/asf/hadoop.git -r 2e18d179e4a8065b6a9f29cf2de9451891265cce; compiled by 'jenkins' on 2014-09-05T23:11Z
STARTUP_MSG:   java = 1.8.0_25
************************************************************/
14/10/17 12:23:50 INFO namenode.NameNode: registered UNIX signal handlers for [TERM, HUP, INT]
14/10/17 12:23:50 INFO namenode.NameNode: createNameNode [-format]
14/10/17 12:23:52 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Formatting using clusterid: CID-96f0c84f-4438-4118-b039-ff311731f47e
14/10/17 12:23:52 INFO namenode.FSNamesystem: fsLock is fair:true

14/10/17 12:23:54 INFO util.GSet: Computing capacity for map cachedBlocks
14/10/17 12:23:54 INFO util.GSet: VM type       = 32-bit
14/10/17 12:23:54 INFO util.GSet: 0.25% max memory 966.7 MB = 2.4 MB
14/10/17 12:23:54 INFO util.GSet: capacity      = 2^19 = 524288 entries
14/10/17 12:23:54 INFO namenode.FSNamesystem: dfs.namenode.safemode.threshold-pct = 0.9990000128746033
14/10/17 12:23:54 INFO namenode.FSNamesystem: dfs.namenode.safemode.min.datanodes = 0
14/10/17 12:23:54 INFO namenode.FSNamesystem: dfs.namenode.safemode.extension     = 30000
14/10/17 12:23:54 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
14/10/17 12:23:54 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
14/10/17 12:23:54 INFO util.GSet: Computing capacity for map NameNodeRetryCache
14/10/17 12:23:54 INFO util.GSet: VM type       = 32-bit
14/10/17 12:23:54 INFO util.GSet: 0.029999999329447746% max memory 966.7 MB = 297.0 KB
14/10/17 12:23:54 INFO util.GSet: capacity      = 2^16 = 65536 entries
14/10/17 12:23:54 INFO namenode.NNConf: ACLs enabled? false
14/10/17 12:23:54 INFO namenode.NNConf: XAttrs enabled? true
14/10/17 12:23:54 INFO namenode.NNConf: Maximum size of an xattr: 16384
Re-format filesystem in Storage Directory /home/hduser1/data/hdfs/namenode ? (Y or N) Y
14/10/17 12:24:01 INFO namenode.FSImage: Allocated new BlockPoolId: BP-1140521248-127.0.1.1-1413573841475
14/10/17 12:24:01 INFO common.Storage: Storage directory /home/hduser1/data/hdfs/namenode has been successfully formatted.
14/10/17 12:24:02 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid &gt;= 0
14/10/17 12:24:02 INFO util.ExitUtil: Exiting with status 0
14/10/17 12:24:02 INFO namenode.NameNode: SHUTDOWN_MSG:
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at user1-VBox/127.0.1.1
************************************************************/
hduser1@user1-VBox $
```

Start HDFS and Yarn.

```
hduser1@user1-VBox ~ $ start-dfs.sh
hduser1@user1-VBox ~ $ start-yarn.sh
```

Verify that the expected Hadoop processes are running properly using the jps command.

```
hduser1@user1-VBox ~ $ jps
5076 NodeManager
5382 Jps
4662 DataNode
4569 NameNode
4987 ResourceManager
4813 SecondaryNameNode
hduser1@user1-VBox ~ $
```

Validate the installation by running an example MapReduce job to compute the value of pi.

```
hduser1@user1-VBox ~ $ hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.1.jar  pi 4 1000
```

####References
[1] [Running Hadoop on Ubuntu Linux (Single-Node Cluster)](http://www.michael-noll.com/tutorials/running-hadoop-on-ubuntu-linux-single-node-cluster/) by Michael G. Noll


[2] [Install Hadoop/YARN 2.4.0 on Ubuntu (Virtual Box)](http://parambirs.wordpress.com/2014/05/20/install-hadoopyarn-2-4-0-on-ubuntu-virtualbox/) by Param Gyaan


[3] [How to Completely Install Oracle JDK on Linux Mint 17](http://community.linuxmint.com/tutorial/view/1839)
