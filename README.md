#### This document will soon be re-written for the upcoming release.

# Material Camera

Android's video recording APIs are very difficult to figure out, especially since a lot of manufacturers
like to mount their camera sensors upside down or sideways. This library is a result of lots of research
and experimentation to get video recording to work universally.

![Art](https://raw.githubusercontent.com/afollestad/material-camera/master/art/deviceart.png)

---

# Notice

Please report any issues you have, and include device information. Camera behavior can be unpredictable
across different Android manufacturers and versions, especially on pre-Lollipop devices. I've done quite
a bit of testing, but it's possible I missed something.

**Some of this documentation needs to be updated.**

---

# Gradle Dependency

[![Release](https://jitpack.io/v/afollestad/material-camera.svg)](https://jitpack.io/#afollestad/material-camera)
[![Build Status](https://travis-ci.org/afollestad/material-camera.svg)](https://travis-ci.org/afollestad/material-camera)
[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg?style=flat-square)](https://www.apache.org/licenses/LICENSE-2.0.html)

### Repository

Add this in your root `build.gradle` file (**not** your module `build.gradle` file):

```gradle
allprojects {
	repositories {
		...
		maven { url "https://jitpack.io" }
	}
}
```

### Dependency

Add this in your module's `build.gradle` file:

```gradle
dependencies {
    ...
    compile('com.github.afollestad:material-camera:bda3fd2ce1@aar') {
        transitive = true
    }
}
```

---


# Basics

### Android Manifest

First, you have to register two library Activities from your app's `AndroidManifest.xml` file:

```xml
<activity
    android:name="com.afollestad.materialcamera.CaptureActivity"
    android:theme="@style/MaterialCamera.CaptureActivity" />
<activity
    android:name="com.afollestad.materialcamera.CaptureActivity2"
    android:theme="@style/MaterialCamera.CaptureActivity" />
```
            
Feel free to use your own custom theme. The included themes give the activities a good default look. 
See the sample project for more details.

### Code

```java
private final static int CAMERA_RQ = 6969; 

File saveFolder = new File(Environment.getExternalStorageDirectory(), "MaterialCamera Sample");
if (!saveFolder.mkdirs())
    throw new RuntimeException("Unable to create save directory, make sure WRITE_EXTERNAL_STORAGE permission is granted.");

new MaterialCamera(this)                       // Constructor takes an Activity
    .allowRetry(true)                          // Whether or not 'Retry' is visible during playback
    .autoSubmit(false)                         // Whether or not user is allowed to playback videos after recording. This can affect other things, discussed in the next section.
    .saveDir(saveFolder)                       // The folder recorded videos are saved to
    .primaryColorAttr(R.attr.colorPrimary)     // The theme color used for the camera, defaults to colorPrimary of Activity in the constructor
    .showPortraitWarning(true)                 // Whether or not a warning is displayed if the user presses record in portrait orientation
    .defaultToFrontFacing(false)               // Whether or not the camera will initially show the front facing camera
    .retryExits(false)                         // If true, the 'Retry' button in the playback screen will exit the camera instead of going back to the recorder
    .start(CAMERA_RQ);                         // Starts the camera activity, the result will be sent back to the current Activity
```

**Note**: For `retryExists(true)`, `onActivityResult()` in the `Activity` that starts the camera will
receive `MaterialCamera.STATUS_RETRY` as the value of the `MaterialCamera.STATUS_EXTRA` intent extra.

---

# Length Limiting

You can specify a time limit for recording. `countdownMillis(long)`, `countdownSeconds(float)`, 
and `countdownMinutes(float)` are all methods for length limiting.

```java
new MaterialCamera(this)
    .countdownMinutes(2.5f)
    .start(CAMERA_RQ);
```

When the countdown reaches 0, recording stops. There are different behaviors that can occur after this based on
`autoSubmit` and `autoRetry`:

1. `autoSubmit(false)`, `allowRetry(true)`
    * The user will be able to playback the recording, and the 'Retry' button will be visible. This is default behavior.
2. `autoSubmit(false)`, `allowRetry(false)`
    * The user will be able to playback the recording, but the 'Retry' button will be hidden.
3. `autoSubmit(true)`, `allowRetry(false)`
    * The user won't be able to playback the recording, the result will immediately be returned to the starting Activity.
4. `autoSubmit(true)`, `allowRetry(true)`
    * If you don't specify a length limit, the behavior will be the same as number 3. If you do specify a length limit, the user is allowed to retry, but the countdown timer will continue until it reaches 0. When the countdown is complete, the result will be returned to the starting Activity automatically.

If you want the countdown to start immediately when the camera is open, as opposed to when the user presses
'Record', you can set `countdownImmediately(true)`:

```java
new MaterialCamera(this)
    .countdownMinutes(2.5f)
    .countdownImmediately(true)
    .start(CAMERA_RQ);
```

---

# Receiving Results

```java
public class MainActivity extends AppCompatActivity {

    private final static int CAMERA_RQ = 6969;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        new MaterialCamera(this)
            .start(CAMERA_RQ);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        // Received recording or error from MaterialCamera
        if (requestCode == CAMERA_RQ) {
        
            if (resultCode == RESULT_OK) {
                Toast.makeText(this, "Saved to: " + data.getDataString(), Toast.LENGTH_LONG).show();
            } else if(data != null) {
                Exception e = (Exception) data.getSerializableExtra(MaterialCamera.ERROR_EXTRA);
                e.printStackTrace();
                Toast.makeText(this, e.getMessage(), Toast.LENGTH_LONG).show();
            }
        }
    }
}
```

---

# [LICENSE](/LICENSE.md)

###### Copyright 2016 Aidan Follestad

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
