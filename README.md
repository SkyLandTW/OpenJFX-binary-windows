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

Source from [OpenJFX 8u172-b11](http://hg.openjdk.java.net/openjfx/8u-dev/rt/rev/ec63c85958b1)

#### Requirements

 - OpenJDK: I use [Zulu](https://www.azul.com/downloads/zulu/zulu-windows/) 8u172 client (.MSI)
 - Cygwin (x64)
       * openssh
       * bison
       * flex
       * g++
       * gperf
       * make
       * makedepend
       * mercurial
       * perl
       * zip
       * unzip

 - Microsoft Visual C++ 2010 Express SP1

   or Microsoft Visual Studio 2010, and then SP1

 - Microsoft Windows SDK for Windows 7.1

   if setup.exe fails, download the ISO, manually install every MSI packages inside, and:

       1. Create symbolic link *C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1* to *C:\Program Files\Microsoft SDKs\Windows\v7.1*, if the latter is created.
       2. Edit *C:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\Tools\VCVarsQueryRegistry.bat*, replace _\v7.0A_ with _\v7.1_

 - Microsoft DirectX SDK (June 2010): DXSDK_Jun10.exe
 - Gradle 2.8
 - Ant 1.8.4
 - CMake windows (NOT cygwin version), version 3.11.1 x64 works

#### Source changes

###### _modules\media\src\main\native\gstreamer\3rd_party\glib\glib-2.28.8\glib\gmain.c_

Change line 3692, from

```C
  GSource *source = g_source_new (&g_timeout_funcs, sizeof (GTimeoutSource));
#ifdef GSTREAMER_LITE
  if (source == NULL)
      return NULL;
#endif // GSTREAMER_LITE
  GTimeoutSource *timeout_source = (GTimeoutSource *)source;
```

To

```C
  GSource *source = g_source_new (&g_timeout_funcs, sizeof (GTimeoutSource));
  GTimeoutSource *timeout_source = (GTimeoutSource *)source; // MOVE THIS LINE
#ifdef GSTREAMER_LITE
  if (source == NULL)
      return NULL;
#endif // GSTREAMER_LITE
```

(move variable declaration to the beginning; workaround for C89 limit in VC++ 2010)

###### _modules\media\src\main\native\gstreamer\gstreamer-lite\gstreamer\plugins\elements\gstqueue.c_

Go the last function gst_queue_change_state() at the bottom of the file, add "{" and "}":

```C
    { // ADD THIS BRACE
    GstStateChangeReturn ret = GST_ELEMENT_CLASS (parent_class)->change_state (element, transition);

    if (ret == GST_STATE_CHANGE_FAILURE)
        return ret;

    switch (transition)
    {
        case GST_STATE_CHANGE_READY_TO_NULL:
            GST_QUEUE_MUTEX_LOCK (queue);
            queue->srcresult = GST_FLOW_WRONG_STATE; // make sure we stop _loop task
            g_cond_signal (queue->item_add);
            GST_QUEUE_MUTEX_UNLOCK (queue);
            break;
        default:
            break;
    }
    return ret;
    } // ADD THIS BRACE
```

(marks the area as a new block from which the "ret" variable may be declared; also workaround for the same C89 limit)

#### Build file changes

###### _buildSrc\win.gradle_

go to line 91, insert before the "if" block, in case the SDK directories are changed by scripts:

```groovy
WINDOWS_SDK_DIR = "C:/Program Files (x86)/Microsoft SDKs/Windows/v7.1"
WINDOWS_DXSDK_DIR = "C:/Program Files (x86)/Microsoft DirectX SDK (June 2010)"

if (WINDOWS_SDK_DIR == null || WINDOWS_SDK_DIR == "") {
    throw new GradleException("FAIL: WINSDK_DIR not defined");
}
```

###### _gradle.properties_

copy _gradle.properties.template_ to _gradle.properties_, go to line 45, uncomment:

```properties
COMPILE_WEBKIT = true
COMPILE_MEDIA = true
```

and line 62, uncomment:

```properties
BUILD_SRC_ZIP = true
```

#### Build!

1. Start *Start Visual Studio x64 Win64 Command Prompt (2010)*
2. Inside the above console, executes:

    *assume cygwin is at C:\cygwin64, replace with yours*

    ```batch
        set DXSDK_DIR=C:/Program Files (x86)/Microsoft DirectX SDK (June 2010)
        set WINSDK_DIR=C:/Program Files (x86)/Microsoft SDKs/Windows/v7.1
        C:\cygwin64\bin\mintty.exe -i /Cygwin-Terminal.ico -
    ```


3. Inside the cygwin console, executes:

    assume:

        * gradle is at C:\gradle-1.8
        * ant is at C:\apache-ant-1.8.4
        * cmake is at C:\cmake-3.11.1-win64-x64
    
    note cmake path needs to be in front of all others in order to override cygwin's cmake, if it exists


    ```bash
    export PATH=$PATH:/cygdrive/c/apache-ant-1.8.4/bin
    export PATH=$PATH:/cygdrive/c/gradle-1.8/bin
    export PATH=/cygdrive/c/cmake-3.11.1-win64-x64/bin:$PATH
    ```

4. cd to the rt directory, run gradle. Or step by step in case there are failures:

    ```bash
    gradle tasks
    gradle base:build -x test
    gradle graphics:build -x test
    gradle controls:build -x test
    gradle extensions:build -x test
    gradle swing:build -x test
    gradle swt:build -x test
    gradle fxml:build -x test
    gradle media:build -x test       # native compiling, prone to fail
    gradle web:build -x test         # native compiling, prone to fail, take forever
    gradle builders:build -x test
    gradle jmx:build -x test
    gradle fxpackager:build -x test  # native compiling

    # finalize everything
    gradle all -x test -x apps
    gradle src -x test
    ```

5. The result is at _build\bundles\javafx-sdk-overlay.zip_
