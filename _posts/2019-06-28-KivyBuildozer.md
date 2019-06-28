---
title: "From Python to APK: How to use Kivy and Buildozer on Windows"
date: 2019-06-28
tags: [Guide, Python, Android, Kivy, Buildozer, APK]
header:
  image: "/assets/images/KivyBuildozer/kivy_logo.png"
excerpt: "A guide on how to create Android applications from Python code"
---
Kivy and Buildozer together allow developers to create an Android application package (APK) from Python code. While working on a project with these libraries it has come to my attention that this process can however be troublesome and time-consuming. To spare others the hours of looking at rushed documentation (especially for Buildozer) I decided to merge some guides together of all the extra work that to be done to be able to make Buildozer compile your Python code on a Windows operating system.

## Virtual Machine containing Kivy and Buildozer setup
1.	Download [Kivy Buildozer VM](https://txzone.net/files/torrents/kivy-buildozer-vm-2.0.zip) and unzip the file.
2.	Download the version of VirtualBox for your machine from [Oracle VirtualBox download area](https://www.virtualbox.org/wiki/Downloads) and install it.
3.	Download [Oracle VM VirtualBox Extension Pack for all platforms](https://download.virtualbox.org/virtualbox/5.2.10/Oracle_VM_VirtualBox_Extension_Pack-5.2.10.vbox-extpack) and install it.
4.	Start _VirtualBox_, click on _File_ _»_ _Import_ _Appliance..._
5.	Select the extracted directory, file named, _Buildozer_ _VM.ovf_
6.	Click on _Settings_ _»_ _General_ _»_ _Advanced_ and select _Bidirectional_ in the drop-down list for _Shared_ _Clipboard_:, and _Drag'n'Drop_:
7.	Click on _Settings_ _»_ _Shared_ _Folders_ and click _Add_ _new_ _shared_ _folder_. e.g. Kivy Apps folder. Make sure _Auto-mount_ has been selected.
8.	Click on _Settings_ _»_ _USB_, check _Enable_ _USB_ _Controller_, checkbox _USB_ _2.0_ _(EHCI)_ _Controller_, and click _Add_ _new_ _USB_ _filters_ e.g. Samsung phone.

## PyCharm IDE Setup
1.	Download [PyCharm Community](https://www.jetbrains.com/pycharm/download/#section=windows) and install it.
2.	Install [KV Lang Auto-completion and Highlighting](https://github.com/kivy/kivy/wiki/Setting-Up-Kivy-with-various-popular-IDE's)
3.	Please refer to [PyCharm Project Interpreter Setup](https://stackoverflow.com/questions/49466785/kivy-error-python-2-7-sdl2-import-error/49477111#49477111)

## Setting up adb/USB drivers on your Virtual Machine
1.	Start _VirtualBox_, select _Kivy/Buildozer_ _VM_ and click _Start_
2.	Password is _kivy_
3.	Start your file manager on your Virtual Machine and Click on _View_ _»_ _Show_ _Hidden_ _Files_
4.	Navigate to _/home/kivy/_ and open _.bashrc_
5.	Add the following lines to the end of the file:
	_export_ _PATH=${PATH}:~/.buildozer/android/platform/android-sdk/tools_
	_export_ _PATH=${PATH}:~/.buildozer/android/platform/android-sdk/platform-tools_
6.	Navigate to _/etc/udev/rules.d/_ and open a terminal here by right clicking and selecting _Open_ _Terminal_ _here_
7.	Enter sudo mousepad 80-android.rules into the terminal
8.	Add the following line to the file
	_SUBSYSTEM==”usb”,_ _ATTR{idVendor}==”USB-VENDOR-ID”,_ _MODE=”0666″_
	In which you replace _USB-VENDOR-ID_ with the ID of your device in Table 1
9.	Give permission to all users to read and write by typing _chmod_ _777_ _80-android.rules_ into the terminal
10.	Enter _adb_ _devices_ into the terminal to start detecting your phone or tablet

## Create a package for Android
1.	Start _VirtualBox_, select _Kivy/Buildozer_ _VM_ and click _Start_
2.	Password is _kivy_
3.	Copy your Kivy App from local machine to a _folder_ in Home folder.
4.	Double click _Home_ folder
5.	Double click your Kivy App folder
6.	Right mouse click and select _Open_ _Terminal_ _here_
7.	At Terminal command prompt, type _buildozer_ _init_ to create _buildozer.spec_ file.
8.	Double click _buildozer.spec_ file to make changes (e.g. _title,_ _package.name,_ _log_level_ _=_ _2_) and save changes.
9.	At Terminal command prompt, type _buildozer_ _android_ _debug_ _deploy_ _run_
10.	Wait a few minutes and you should be done!

<a href="/assets/images/KivyBuildozer/table_1.png"><img src="/assets/images/KivyBuildozer/table_1.png"></a>