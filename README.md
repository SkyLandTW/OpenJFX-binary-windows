OpenJFX-binary-windows
======================

x64 Binary for JavaFX (OpenJFX) part of OpenJDK


Installation
------------

- Install OpenJDK binary such as [Zulu](http://www.azul.com/downloads/zulu/)
- Copy files in *sdk\bin* to *Zulu\zulu-8\bin*
- Copy files in *sdk\lib* to *Zulu\zulu-8\lib*
- Copy files in *sdk\rt\bin* to *Zulu\zulu-8\jre\bin*
- Copy files in *sdk\rt\lib* to *Zulu\zulu-8\jre\lib*
- Copy files in *sdk\rt\lib\ext* to *Zulu\zulu-8\jre\lib\ext*


Build Steps
-----------

Source from [OpenJFX 8u60](http://hg.openjdk.java.net/openjfx/8u60/rt/)

With:
 - Cygwin (x64)
 - Microsoft Visual C++ 2010 Express SP1
 - Microsoft Windows SDK for Windows 7.1
 - Microsoft DirectX SDK (June 2010)
 - Gradle 2.8

See [Building OpenJFX](https://wiki.openjdk.java.net/display/OpenJFX/Building+OpenJFX#BuildingOpenJFX-Windows)
