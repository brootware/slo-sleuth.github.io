# Installing Autopsy on MacOS Catalina

## Table of Contents
- [What is Autopsy?](#What-is-Autopsy?)
- [Installation Overview](#Installation-Overview)
- [First Things First](#First-Things-First)
- [Installing the Java Development Kit](#Installing-the-Java-Development-Kit)
- [Building the Sleuthkit](#Building-the-Sleuthkit)
- [Installing External Tool Dependencies](#Installing-External-Tool-Dependencies)
- ["Installing" Autopsy](#Installing-Autopsy)
- [Running Autopsy](#Running-Autopsy)
- [Troubleshooting](#Troubleshooting)

## What is Autopsy?

[Autopsy](https://www.autopsy.com/) is a digital forensics tool with a graphical interface that can run on Window, Linux, and macOS.  It is developed and provided to the forensics community at no cost by [Basis Technology Corp](https://www.basistech.com/). It's underlying engine is the set of command line tools found in [The Sleutkit](http://sleuthkit.org/).

Autopsy support on macOS by Basis Technology is minimal and not all features of the interface currently work, e.g., the TimeLine tool.  Windows is the main target platform for Autopsy.  That said, it still provides many useful features to the digital forensics examiner working in macOS.

> **NOTE**: This guide is meant to ease the process of installing Autopsy on macOS.  It is a prolonged explanation so that you can understand the process and make more meaningful requests for assistance, if needed.

## Installation Overview

The process of installing Autopsy on macOS generally involves the following steps:

- Installing dependencies:
  - Java Development Kit
  - The Sleuthkit *(from source)*
  - TestDisk
  - Gstreamer
- Configuring your environment
- Configuring Autopsy

This guide will assume that the user is employing the [Homebrew](https://brew.sh/) package manager to facilitate installation.

## First Things First

You may have reached this guide after failed installation attempts.  This may have left your system in state incompatible with Autopsy and the Sleuthkit.  The installation will go smoother if you first clean up your system:

```sh
% brew uninstall sleuthkit
% brew cleanup
```

## Installing the Java Development Kit

As of this writing, the sleuthkit and autopsy should be run with [Bellsoft Liberica JDK](https://bell-sw.com/).  The homebrew `openjdk` will install with the `ant` package, but we will force it to build **The Sleuthkit** java archive in a later step.


> **IMPORTANT**: Uninstall any JDKs currently installed on the system.  If you installed with brew, simply `brew uninstall <package-name>`.  If you installed with a downloaded installer, follow the developer's instructions for removal.  *Autopsy can report errors with competing JDKs installed.*

Use brew to add the Bellsoft third-party repository and install the full version of Liberica JDK 8.

```sh
% brew tap bell-sw/liberica
% brew install --cask  liberica-jdk8-full
```

Set the `JAVA_HOME` environment variable so that Liberica Java can be found.

```sh
% export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)
```

> **IMPORTANT**: The export statement above is only effective in shell process in which it is run, i.e., it doesn't persist when you reopen the shell or open another shell.

Test that you have the correct java JDK-8 installed.  It should be Java version 1.8.0, but the build number, represented below as `\###`, can vary.

```sh
% java -version
    openjdk version "1.8.0.###"
    OpenJDK Runtime Environment (build 1.8.0_###-BellSoft-b##)
    OpenJDK 64-Bit Server VM (build ##.###-b##, mixed mode)
```

### JAVA Environment Variable Persistence

To make the JAVA_HOME variable persistent and thus your life much easier, write the `export` in the section above command into your .`~.bashrc` (macOS 10.14 and below) and/or `~/.zshrc` (macOS 15). 

```sh
% echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 1.8)' | tee >> ~/.bashrc >> ~./zshrc
```

Every new shell instance will now set the required JAVA_HOME variable automatically.  To test, reopen the terminal (or open a new tab) and execute:

```sh
% echo $JAVA_HOME
/Library/Java/JavaVirtualMachines/liberica-jdk-8-full.jdk/Contents/Home
```

## Building the Sleuthkit

Each version of Autopsy requires a specific version of the sleuthkit.  For example, Autopsy 4.16.0 requires Sleuthkit 4.10.0.  While sleuthkit is included in the Windows installation package, this is not the case for Linux and macOS.  Instead, you must build and install it yourself.

> **IMPORTANT**: The Homebrew package manager has a prebuilt sleuthkit v4.10.0 package, but it was built with the wrong version of Java to support Autopsy.  *You must build sleuthkit from source* with the Liberica-jdk8-full package from the previous section.*

### Install Sleuthkit Dependencies

The Sleuthkit requires several packages to build it with full support.

```sh
% brew install ant afflib libewf libpq
```

Now create a link from the liberica-jdk8 installation to where `ant` expects to find openjdk.  This is necessary to build sleuthkit with liberica java 1.8.0 support.

```sh
% rm /usr/local/opt/openjdk
% ln -s $JAVA_HOME /usr/local/opt/openjdk
```

Test your link file creation to ensure it is pointing at the correct java developement kit:

```sh
% ls -l /usr/local/opt/openjdk 
lrwxr-xr-x  1 user  admin  71 Apr 23 17:19 /usr/local/opt/openjdk -> /Library/Java/JavaVirtualMachines/liberica-jdk-8-full.jdk/Contents/Home
```

> **NOTE**: Your JAVA_HOME variable must be set for the link to be created.

The basic Sleuthkit dependencies should now be met. 

### Build and Install the Sleuthkit

Download the appropriate [Sleuthkit TAR file](https://github.com/sleuthkit/sleuthkit/releases).  For Autopsy 4.16.0, download `sleuthkit-4.10.0.tar.gz`. 

Open a terminal and change to the download directory, likely `~/Downloads/`.  Then:

```sh
% tar xzvf sleuthkit-4.10.0.tar.gz
% cd sleuthkit-4.10.0
```

You have expanded the sleuthkit source code and changed into the root of the source code directory.  Before you configure the installation, you must set the CPPFLAGS variable to achieve postgresql support.  Then, from the sleuthkit directory, execute the configuration command.

```sh
% export CPPFLAGS="-I/usr/local/opt/libpq/include"
% ./configure
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... config/install-sh -c -d
...
configure:
Building:
   afflib support:                        yes
   libewf support:                        yes
   zlib support:                          yes

   libvhdi support:                       no
   libvmdk support:                       no
   postgresql support:                    yes
Features:
   Java/JNI support:                      yes
   Multithreading:                        yes
```

If you did not see affirmative `Java/JNI support` in the configure command output, **_stop.  Do not go on_**.  Autopsy 4.16 requires a `sleuthkit-4.10.0.jar` file built with liberica java 1.8.0 to function.  Repeat the  [dependency installation process](#Installing-Sleuthkit-dependencies) with particular focus on the link file creation.

If your configuration file looks like the one above, i.e., support for afflib, libewf, zlib, postgresql, Java/JNI, and Multithreading at the least, then you are ready to proceed with the `make` command to build sleuthkit.

```sh
% make 
...
copyMacLibs:
     [copy] Copying 1 file to /Users/user/Downloads/sleuthkit-4.8.0/bindings/java/build/NATIVELIBS/x86_64/mac
     [copy] Copying 1 file to /Users/user/Downloads/sleuthkit-4.8.0/bindings/java/build/NATIVELIBS/amd64/mac

copyLibs-SQLite:

dist-SQLite:
      [jar] Building jar: /Users/user/Downloads/sleuthkit-4.8.0/bindings/java/dist/sleuthkit-4.8.0.jar

BUILD SUCCESSFUL
Total time: 4 seconds
```

You will seen many commands, messages, and warnings over the several minutes it will take to build sleuthkit.  At the end of the build process, you should see `BUILD SUCCESSFUL` if all went well.

Now install sleuthkit to put the tools and libraries and make them accessible through your `PATH`.

```sh
% sudo make install
```

> **TIP**: This is the only point of the installation process where you are required to execute a command with sudo (as root).  Do not complicate the installation process by executing the other commands as root.

Verify that you have java support by locating the `sleuthkit-4.10.0.jar` file:

```sh
% ls /usr/local/share/java     
sleuthkit-4.10.0.jar
```

The Sleuthkit is now properly installed and ready to support Autopsy, but Autopsy needs a few more software packages to acheive full functionality.

## Installing External Tool Dependencies

### TestDisk

The testdisk package includes the photorec tool, a dependency of Autopsy.  Photorec is used by Autopsy for file carving.

```sh
% brew install testdisk
```

### Gstreamer

The gstreamer package is required for video playback.  It has plugins that provide the functionality needed by gstreamer applications.

```
% brew install gstreamer gst-plugins-base gst-plugins-good
```

You have now installed the external tool dependencies for Autopsy.

## "Installing" Autopsy

You don't really "install" Autopsy in the true sense of the word.  You simply expand the Autopsy release ZIP file, run a configuration script, and then start Autopsy from the executable file in the Autopsy `bin` directory. 

First, download the [Autopsy ZIP file](https://github.com/sleuthkit/autopsy/releases) and expand it in a location where you, as a user, have access (again, you do not need to be, nor should you be, the root user to run Autopsy).  The `~/Downloads` directory is both an acceptable and convenient location.

In the terminal, change to the autopsy-4.16.0 directory and execute the `unix_setup.sh` script to configure Autopsy.  The script tells Autopsy where to find the photorec tool, checks that the JAVA_HOME variable is set, and copies the sleuthkit-4.10.0.jar file into the Autopsy tree.

```sh
% cd ~/Downloads/autopsy-4.16.0
% sh unix_setup.sh
---------------------------------------------
Checking prerequisites and preparing Autopsy:
---------------------------------------------
-n Checking for PhotoRec...
found in /usr/local/bin
-n Checking for Java...
found in /Library/Java/JavaVirtualMachines/liberica-jdk-8-full.jdk/Contents/Home
-n Checking for Sleuth Kit Java bindings...
found in /usr/local/share/java
-n Copying sleuthkit-4.10.0.jar into the Autopsy directory...
done

Autopsy is now configured. You can execute bin/autopsy to start it
```

You have successfully installed Autopsy and are ready to run it.  If you received errors, _do not try to start Autopsy_.  Doing so can create settings application support settings that will complicate starting Autopsy once you've corrected the errors.  

See [Troubleshooting](#Troubleshooting) if you are having problems starting autopsy after successful configuration.

## Running Autopsy

You can execute Autopsy in the manner stated at the end of the configuration output.  From the root of the autopsy folder, execute:

```sh
% bin/autopsy
```

Each time you choose to start Autopsy, you'll need to change to the Autopsy installation directory or type a long path, e.g., `~/Downloads/autopsy-4.16.0/bin/autopsy`.  However, you can simplify the process in many ways, but I'll demonstrate two here:

- Create a link to Autopsy in the home folder.  Then open a terminal and launch from the command line.

	```sh
  % ln -s ./Downloads/autopsy-4.16.0/bin/autopsy autopsy
  % ./autopsy
  ```

- Create an application launcher with Automator.
  - Launch Automator from Launchpad.  It starts silently, so expand the window from the Dock icon.
  - Choose *Application* for your type of document.
    - Under Actions in the left pane, select *Utilities* then *Run Shell Script* in the next column.  
    - In the box below the Shell selection, enter:
      `./Downloads/autopsy-4.14.0/bin/autopsy`

  - Save your automator application file as "Autopsy" and you will now have an *Autopsy.app* in your applications folder.

> **Tip**: open the icon.ico in the Autopsy folder and copy it into the Autopsy.app "info" screen to have use the Autopsy icon on your automator application.  Google how to change a macOS icon if you need more information on the steps required.

### Known Problems with Autopsy on macOS

- The Autopsy Timeline module does not work under macOS.  There is no plan to fix it, to my knowledge.  

- Video and audio playback doesn't work in application tab of the Data Content window.  It is likely a path issue that I hope to track down.  You can still play media files by right-clicking on them and choose "Open in External Viewer".

- The Autopsy Github site [states VHD and VMDK image files are not supported](https://github.com/sleuthkit/autopsy/blob/develop/Running_Linux_OSX.txt).  However, Sleuthkit can, in fact, be compiled with VHD and VMDK support.  I have not yet tested if VM image support can be enabled by compiling sleuthkit in this way and doing so is beyond the scope of this guide.

- The external hex editor path is not configurable.

## Troubleshooting

Make sure your `JAVA_HOME` environment variable is set *in the terminal in which you configuring Autopsy*.

```sh
% echo $JAVA_HOME
/Library/Java/JavaVirtualMachines/liberica-jdk-8-full.jdk/Contents/Home
```

> **NOTE**: If nothing returns, `JAVA_HOME` is not set.  Refer to [JAVA Environment Variable Persistence](#JAVA-Environment-Variable-Persistence) for help.

When building Sleuthkit, make sure `ant` will find the liberica-jdk8 installation:

```sh
% ln -s $JAVA_HOME /usr/local/opt/openjdk
```

Check that you, in fact, built Sleuthkit with Java support.  If so, you should have a `sleuthkit-4.10.0.jar` file in `/usr/local/share/java`.

```sh
% ls /usr/local/share/java/
sleuthkit-4.10.0.jar
```

> **NOTE**: If nothing returns, you did not build sleuthkit with java support.  Refer to [Building the Sleuthkit](#Building-the-Sleuthkit) for help.

If Autopsy starts without and errors but you don't seee any normal Autopsy controls (New Case dialog and/or a menu bar with "Case | View | Tools | Windows | Help"), then you probably started Autopsy at least one time before the build was correct.  To overcome this issue, delete the Autopsy application support folder:

```
% rm -rf ~/Library/Application\ Support/autopsy
```

If Autopsy starts with a Java error reporting an incompatible or later version of Java, e.g., "InvocationTargetException" or references to JDK 13, then you built sleuthkit with the wrong Java development kit.  Remove any third party Java installations except the liberica-jdk-8.  If they were installed with brew, you can find and remove them with:

```
% brew cask list
% brew uninstall <package-name>
```

> **NOTE**: If you download and installed Java runtime or development kit from a website, seek directions for uninstalling at from the creator.  It is likely a manual process, but it is essential to your success in runing Autopsy.

### Still Having Trouble?

I've tried to make this guide as complete as possible without making it overwhelming (I don't think I succeeded).  There is place you can go for individualized assistance: the [Sleuthkit Discourse forum](https://sleuthkit.discourse.group/). 

Before you post a question, look to see if it has already been answered.  You'll get faster results.  Search the forum with words specific to your problem.  Questions that have already been answered elsewhere are less likely to receive a response.

If your installation question has not already been answered, post it in the [Autopsy on Linux / MacOS](https://sleuthkit.discourse.group/c/autopsylinux/14) category. Try to be as specific as possible:

Rather than "It didn't work" or "Autopsy doesn't start" statements, be as specific as possible:

- Summarize the problem as  _the topic of your post_. Examples:
  - Autopsy reports wrong Sleuthkit version, 
  - unix_setup.sh reports sleuthkit-4.8.0.jar not found
  - Autopsy reports Java-FX is missing
- What commands did you run?  
  - Paste the commands exactly, indent them by four spaces (block quote) to make them easy to read.
  - Think of it this way: how can someone help you if they don't know what you did?
- What did you expect to happen?
  - Describe what you expected to the the result.  In unix-like OSes, most commands succeed silently.
- What actually happened?
  - Paste or screen shot the result, but pasting is preferred as it is easier to detect white-space issues, font differences, etc.
- What steps have you taken to try to resolve the problem?
  - List any troubleshooting steps you have taken, and their results.  You may be halfway to a solution or you may have inadvertently created another problem.

Posting your commands, the results commands, screen shots, etc., will help people help you, and it will almost certainly guarantee a response to your question.  Questions that are too general are likely to go unanswered because there is no starting point for a resolution.

> **IMPORTANT**: Remember, Basis Tech provides the tool for free.  They support when they can, but the paying work has to come first.  The people trying to answer your questions are most often volunteering their time and not directly related to Basis Tech.  A little clarity and decorum goes a long way...
