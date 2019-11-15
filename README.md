# HelloAgora
This doc contains instructions for how to start a basic call using the Agora SDK for Android. This application provides basic video call functionalities (switch camera, mute/unmute local audio and join/leave channel), voice changer and remote screen shacking. 

# Prerequisites
* Android Studio 3.0 or later
* Android SDK API Level 16 or higher
* A mobile device running Android 4.1 or later
* A valid Agora account ([Sign up](https://console.agora.io/) for free)

# Set up the development environment
In this section, we will create an Android project, integrate the SDK into the project, and add the Android device permissions to prepare the development environment.

## Create an Android project
If you already have a project, you can skip to the next section. Otherwise, follow the directions below to build an Android project from scratch.

1. Open **Android Studio** and click **Start a new Android Studio project**.

2. On the **Choose your project** panel, choose **Phone and Tablet** > **Empty Activity**, and click **Next**.

3. On the **Configure your project** panel, fill in the following contents:

 * **Name**: The name of your project, for example, HelloAgora
 * **Package name**: The name of the project package, for example, io.agora.helloagora
 * **Project location**: The path to save the project
 * **Language**: The programming language of the project, for example, Java
 * **Minimum API level**: The minimum API level of the project

Click **Finish**. Follow the on-screen instructions, if any, to install the plug-ins.

## Integrate the SDK
Choose either of the following methods to integrate the Agora SDK into your project.

**Method 1: Automatically integrate the SDK with JCenter**

Add the following line in the **/app/build.gradle** file of your project:

```javascript
...
dependencies {
    ...
    // 2.9.2 is the latest version of the Agora SDK. You can set it to other versions.
    implementation 'io.agora.rtc:full-sdk:2.9.2'
}
```

**Method 2: Manually copy the SDK files**

 1. Go to [SDK Downloads](https://docs.agora.io/en/Agora%20Platform/downloads), download the latest version of the Agora SDK, and unzip the downloaded SDK package.

 2. Copy the following files or subfolders from the libs folder of the downloaded SDK package to the path of your project.


| File or subfolder      | Path of your project  |
| ---------------------- | --------------------- |
| agora-rtc-sdk.jar file | /app/libs/            |
| arm-v8a folder         | /app/src/main/jniLibs/|
| armeabi-v7a folder     | /app/src/main/jniLibs/|
| x86 folder             | /app/src/main/jniLibs/|
| x86_64 folder          | /app/src/main/jniLibs/|

## Add project permissions
Add the following permissions in the **/app/src/main/AndroidManifest.xml** file for device access according to your needs:

```javascript
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
   package="io.agora.tutorials1v1acall">

   <uses-permission android:name="android.permission.READ_PHONE_STATE" />   
   <uses-permission android:name="android.permission.INTERNET" />
   <uses-permission android:name="android.permission.RECORD_AUDIO" />
   <uses-permission android:name="android.permission.CAMERA" />
   <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
   <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
   <!-- The Agora SDK requires Bluetooth permissions in case users are using Bluetooth devices.-->
   <uses-permission android:name="android.permission.BLUETOOTH" />
   <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
   <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
...
</manifest>
```

## Prevent code obfuscation
Add the following line in the app/proguard-rules.pro file to prevent code obfuscation:
```javascript
-keep class io.agora.**{*;}

```

# Implement basic video call function
This section will introduce how to use Agora SDK to make a basic one-to-one video call. 

### 1. Create the UI

Create the user interface (UI) for the one-to-one call in the layout file of your project. 

First we add a local video view, a remote video view and a end-call button into the UI.

```javascript
<?xml version="1.0" encoding="UTF-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_video_chat_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="io.agora.tutorials1v1vcall.VideoChatViewActivity">

    <RelativeLayout
        android:id="@+id/remote_video_view_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/remoteBackground">
        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_above="@id/icon_padding">
            <ImageView
                android:layout_width="@dimen/remote_back_icon_size"
                android:layout_height="@dimen/remote_back_icon_size"
                android:layout_centerInParent="true"
                android:src="@drawable/icon_agora_largest"/>
        </RelativeLayout>
        <RelativeLayout
            android:id="@+id/icon_padding"
            android:layout_width="match_parent"
            android:layout_height="@dimen/remote_back_icon_margin_bottom"
            android:layout_alignParentBottom="true"/>
    </RelativeLayout>

    <FrameLayout
        android:id="@+id/local_video_view_container"
        android:layout_width="@dimen/local_preview_width"
        android:layout_height="@dimen/local_preview_height"
        android:layout_alignParentEnd="true"
        android:layout_alignParentRight="true"
        android:layout_alignParentTop="true"
        android:layout_marginEnd="@dimen/local_preview_margin_right"
        android:layout_marginRight="@dimen/local_preview_margin_right"
        android:layout_marginTop="@dimen/local_preview_margin_top"
        android:background="@color/localBackground">

        <ImageView
            android:layout_width="@dimen/local_back_icon_size"
            android:layout_height="@dimen/local_back_icon_size"
            android:layout_gravity="center"
            android:scaleType="centerCrop"
            android:src="@drawable/icon_agora_large" />
    </FrameLayout>

    <RelativeLayout
        android:id="@+id/control_panel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_marginBottom="@dimen/control_bottom_margin">

        <ImageView
            android:id="@+id/btn_call"
            android:layout_width="@dimen/call_button_size"
            android:layout_height="@dimen/call_button_size"
            android:layout_centerInParent="true"
            android:onClick="onCallClicked"
            android:src="@drawable/btn_endcall"
            android:scaleType="centerCrop"/>

    </RelativeLayout>

</RelativeLayout>
```

### 2. Get the device permission
Call the _checkSelfPermission_ method to access the camera and the microphone of the Android device when launching the activity.

```javascript
private static final int PERMISSION_REQ_ID = 22;

// Ask for Android device permissions at runtime.
private static final String[] REQUESTED_PERMISSIONS = {
        Manifest.permission.RECORD_AUDIO,
        Manifest.permission.CAMERA,
        Manifest.permission.WRITE_EXTERNAL_STORAGE
};

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_video_chat_view);

    // If all the permissions are granted, initialize the RtcEngine object and join a channel.
    if (checkSelfPermission(REQUESTED_PERMISSIONS[0], PERMISSION_REQ_ID) &&
            checkSelfPermission(REQUESTED_PERMISSIONS[1], PERMISSION_REQ_ID) &&
            checkSelfPermission(REQUESTED_PERMISSIONS[2], PERMISSION_REQ_ID)) {
        initEngineAndJoinChannel();
    }
}

private boolean checkSelfPermission(String permission, int requestCode) {
    if (ContextCompat.checkSelfPermission(this, permission) !=
            PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions(this, REQUESTED_PERMISSIONS, requestCode);
        return false;
    }

    return true;
}
```

### 3. Initialize RtcEngine

Create and initialize the RtcEngine object before calling any other Agora APIs.

In this step, you need to use the App ID of your project. Follow these steps to [create an Agora project](https://docs.agora.io/en/Agora%20Platform/manage_projects?platform=All%20Platforms) in Console and get an [App ID](https://docs.agora.io/en/Agora%20Platform/terms?platform=All%20Platforms#a-nameappidaapp-id).

 1. Go to [Console](https://sso.agora.io/login/?response_type=code&client_id=59T2cPE9by201sbXpExrCEVNgIi0su1h&redirect_uri=https%3A%2F%2Fconsole.agora.io%2Fapi%2Fv2%2Foauth) and click the [Project Management](https://sso.agora.io/login/?response_type=code&client_id=59T2cPE9by201sbXpExrCEVNgIi0su1h&redirect_uri=https%3A%2F%2Fconsole.agora.io%2Fapi%2Fv2%2Foauth) on the left navigation panel.
 2. Click **Create** and follow the on-screen instructions to set the project name, choose an authentication mechanism, and Click **Submit**.
 3. On the **Project Management** page, find the **App ID** of your project.

Call the create method and pass in the App ID to initialize the RtcEngine object.

You can also listen for callback events, such as when the local user joins the channel, and when the first video frame of a remote user is decoded. Do not implement UI operations in these callbacks.

```javascript
private final IRtcEngineEventHandler mRtcEventHandler = new IRtcEngineEventHandler() {
    @Override
    // Listen for the onJoinChannelSuccess callback.
    // This callback occurs when the local user successfully joins the channel.
    public void onJoinChannelSuccess(String channel, final int uid, int elapsed) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","Join channel success, uid: " + (uid & 0xFFFFFFFFL));
            }
        });
    }

    @Override
    // Listen for the onFirstRemoteVideoDecoded callback.
    // This callback occurs when the first video frame of a remote user is received and decoded after the remote user successfully joins the channel.
    // You can call the setupRemoteVideo method in this callback to set up the remote video view.
    public void onFirstRemoteVideoDecoded(final int uid, int width, int height, int elapsed) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","First remote video decoded, uid: " + (uid & 0xFFFFFFFFL));
                setupRemoteVideo(uid);
            }
        });
    }

    @Override
    // Listen for the onUserOffline callback.
    // This callback occurs when the remote user leaves the channel or drops offline.
    public void onUserOffline(final int uid, int reason) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","User offline, uid: " + (uid & 0xFFFFFFFFL));
                onRemoteUserLeft();
            }
        });
    }
};

...

// Initialize the RtcEngine object.
private void initializeEngine() {
    try {
        mRtcEngine = RtcEngine.create(getBaseContext(), getString(R.string.agora_app_id), mRtcEventHandler);
    } catch (Exception e) {
        Log.e(TAG, Log.getStackTraceString(e));
        throw new RuntimeException("NEED TO check rtc sdk init fatal error\n" + Log.getStackTraceString(e));
    }
}
```

### 4. Set up the local video view
After initializing the RtcEngine object, set the local video view before joining the channel so that you can see yourself in the call. Follow these steps to configure the local video view:

 * Call the _enableVideo_ method to enable the video module.
 * Call the _createRendererView_ method to create a SurfaceView object.
 * Call the _setupLocalVideo_ method to configure the local video display settings.

```javascript

private void setupLocalVideo() {

    // Enable the video module.
    mRtcEngine.enableVideo();

    // Create a SurfaceView object.
    private FrameLayout mLocalContainer;
    private SurfaceView mLocalView;

    mLocalView = RtcEngine.CreateRendererView(getBaseContext());
    mLocalView.setZOrderMediaOverlay(true);
    mLocalContainer.addView(mLocalView);
    // Set the local video view.
    VideoCanvas localVideoCanvas = new VideoCanvas(mLocalView, VideoCanvas.RENDER_MODE_HIDDEN, 0);
    mRtcEngine.setupLocalVideo(localVideoCanvas);
}
```

### 5. Join a channel

After initializing the RtcEngine object and setting the local video view (for a video call), you can call the joinChannel method to join a channel. In this method, set the following parameters:

 * token: Pass a token that identifies the role and privilege of the user. You can set it as one of the following values:

  * NULL.
  * A temporary token generated in Console. A temporary token is valid for 24 hours. For details, see [Get a Temporary Token](https://docs.agora.io/en/Agora%20Platform/token?platform=All%20Platforms#get-a-temporary-token).
  * A token generated at the server. This applies to scenarios with high-security requirements. For details, see [Generate a token from Your Server](https://docs.agora.io/en/Video/token_server).
    
    *If your project has enabled the app certificate, ensure that you provide a token.

  * channelName: Specify the channel name that you want to join. Users that input the same channel name join the same channel.

  * uid: ID of the local user that is an integer and should be unique. If you set uid as 0, the SDK assigns a user ID for the local user and returns it in the _onJoinChannelSuccess_ callback.

For more details on the parameter settings, see [joinChanne](https://docs.agora.io/en/Video/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#a8b308c9102c08cb8dafb4672af1a3b4c).

```javascript
private void joinChannel() {
        // Join a channel with a token.
        mRtcEngine.joinChannel(getString(R.string.temp_token), "Andrew", "Extra Optional Data", 0);
    }
```

### 6. Set up the remote video view

In a video call, you should be able to see other users too. This is achieved by calling the setupRemoteVideo method after joining the channel.

Shortly after a remote user joins the channel, the SDK gets the remote user's ID in the _onFirstRemoteVideoDecoded_ callback. Call the _setupRemoteVideo_ method in the callback, and pass in the uid to set the video view of the remote user.

```javascript
    @Override
    // Listen for the onFirstRemoteVideoDecoded callback.
    // This callback occurs when the first video frame of a remote user is received and decoded after the remote user successfully joins the channel.
    // You can call the setupRemoteVideo method in this callback to set up the remote video view.
    public void onFirstRemoteVideoDecoded(final int uid, int width, int height, int elapsed) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","First remote video decoded, uid: " + (uid & 0xFFFFFFFFL));
                setupRemoteVideo(uid);
            }
        });
    }

private void setupRemoteVideo(int uid) {

    // Create a SurfaceView object.
    private RelativeLayout mRemoteContainer;
    private SurfaceView mRemoteView;


    mRemoteView = RtcEngine.CreateRendererView(getBaseContext());
    mRemoteContainer.addView(mRemoteView);
    // Set the remote video view.
    mRtcEngine.setupRemoteVideo(new VideoCanvas(mRemoteView, VideoCanvas.RENDER_MODE_HIDDEN, uid));

}
```

### 7. Leave the channel
Call the _leaveChannel_ method to leave the current call according to your scenario, for example, when the call ends, when you need to close the app, or when your app runs in the background.

```javascript
@Override
protected void onDestroy() {
    super.onDestroy();
    if (!mCallEnd) {
        leaveChannel();
    }
    RtcEngine.destroy();
}

private void leaveChannel() {
    // Leave the current channel.
    mRtcEngine.leaveChannel();
}
```

# Add more functions 
In this section, we will add more functions in the app: mute/unmute local audio, switch the camera direction, voice change and remote screen shacking.

## 1. Add buttons into the UI
Add a mute button, a switch camera button, a screen shacking button and a vioce changer button below the call button we added in the previous step in the _activity_video_chat_view.xml_ file .
```javascript
<RelativeLayout
        android:id="@+id/control_panel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_marginBottom="@dimen/control_bottom_margin">

        <ImageView
            android:id="@+id/btn_call"
            android:layout_width="@dimen/call_button_size"
            android:layout_height="@dimen/call_button_size"
            android:onClick="onCallClicked"
            android:layout_centerInParent="true"
            android:src="@drawable/btn_endcall"
            android:scaleType="centerCrop"/>

        <ImageView
            android:id="@+id/btn_switch_camera"
            android:layout_width="@dimen/other_button_size"
            android:layout_height="@dimen/other_button_size"
            android:layout_toRightOf="@id/btn_call"
            android:layout_toEndOf="@id/btn_call"
            android:layout_marginLeft="@dimen/control_bottom_horizontal_margin"
            android:layout_centerVertical="true"
            android:onClick="onSwitchCameraClicked"
            android:src="@drawable/btn_switch_camera"
            android:scaleType="centerCrop"/>

        <ImageView
            android:id="@+id/btn_mute"
            android:layout_width="@dimen/other_button_size"
            android:layout_height="@dimen/other_button_size"
            android:layout_toLeftOf="@id/btn_call"
            android:layout_toStartOf="@id/btn_call"
            android:layout_marginRight="@dimen/control_bottom_horizontal_margin"
            android:layout_centerVertical="true"
            android:onClick="onLocalAudioMuteClicked"
            android:src="@drawable/btn_unmute"
            android:scaleType="centerCrop"/>


        <ImageView
            android:id="@+id/btn_switch_voice"
            android:layout_width="@dimen/other_button_size"
            android:layout_height="@dimen/other_button_size"
            android:layout_toLeftOf="@id/btn_mute"
            android:layout_toStartOf="@id/btn_mute"
            android:layout_marginRight="@dimen/control_bottom_horizontal_margin"
            android:layout_centerVertical="true"
            android:onClick="onSwitchVoiceClicked"
            android:src="@drawable/btn_change_voice"
            android:scaleType="centerCrop"/>

        <ImageView
            android:id="@+id/btn_shake"
            android:layout_width="@dimen/other_button_size"
            android:layout_height="@dimen/other_button_size"
            android:layout_toRightOf="@id/btn_switch_camera"
            android:layout_toEndOf="@id/btn_switch_camera"
            android:layout_marginLeft="@dimen/control_bottom_horizontal_margin"
            android:layout_centerVertical="true"
            android:onClick="onRemoteShackClicked"
            android:src="@drawable/btn_shack"
            android:scaleType="centerCrop"/>
    </RelativeLayout>
```

## 2. Mute the local audio

Call the _muteLocalAudioStream_ method to stop or resume sending the local audio stream to mute or unmute the local user.

```javascript
public void onLocalAudioMuteClicked(View view) {
    mMuted = !mMuted;
    mRtcEngine.muteLocalAudioStream(mMuted);
}
```

## 3. Switch the camera direction

Call the _switchCamera_ method to switch the direction of the camera.

```javascript
public void onSwitchCameraClicked(View view) {
    mRtcEngine.switchCamera();
}  
```

## 4. Voice changer

Call the _setLocalVoiceChanger_ method to enable voice changer. There are several voice changer options can be chosen, more details at [Voice Changer Api](https://docs.agora.io/en/Video/voice_changer_android?platform=Android).

```javascript
public void onSwitchVoiceClicked(View view) {
        if (!isVoiceChanged) {
            //start voice change to little girl, can be changed to different voices
            mRtcEngine.setLocalVoiceChanger(3);
        }else {
            //disable voice change
            mRtcEngine.setLocalVoiceReverbPreset(0);
        }
        isVoiceChanged = !isVoiceChanged;
    }
```

## 5. Remote screen shacking
This function is to create a shacking animation on the other people's remote view.  

 1. First creat a shacking animation in the _res_ file.
```javascript
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="100"
    android:fromDegrees="-5"
    android:pivotX="50%"
    android:pivotY="50%"
    android:repeatCount="5"
    android:repeatMode="reverse"
    android:toDegrees="5" />
```

 2. Send a message whenever the remote shacking button is been pressed.
```javascript
public void onRemoteShackClicked(View view) {
        //send message to the other user with data = {1}
        mRtcEngine.sendStreamMessage(mRtcEngine.createDataStream(true, true), new byte[]{1});
}
```

 3. Overwrite _onStreamMessage_ and _onStreamMessageError_ methods to receive data.
```javascript
        @Override
        public void onStreamMessage(int uid, int streamId, byte[] data) {
            super.onStreamMessage(uid, streamId, data);
            //do shacking when receive 1
            if (data.length == 1 && data[0] == 1) {
                performAnimation();
            }
        }

        @Override
        public void onStreamMessageError(int uid, int streamId, int error, int missed, int cached) {
            super.onStreamMessageError(uid, streamId, error, missed, cached);
        }
```

 4. Perform shacking animation.
```javascript 
    public void performAnimation() {
        mRemoteContainer.startAnimation(AnimationUtils.loadAnimation(this, R.anim.shake));
    }
```

# More Improvements
In this section, we will add some improvements to make the app more robust. 

## Screen rotation
Add _configChanges_ in the _Manifest_ file to prevent activity been destroyed from screen rotation.

```javascript
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        ...
        <activity android:name=".VideoChatViewActivity"
            android:configChanges="orientation|screenSize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        ...
    </application>
```

## Audio enhancement
Enabling in-ear monitoring function in the RtcEngine will provides a mix of the audio sources to the host with low latency.

```javascript
// Enables in-ear monitoring. The default value is false.
rtcEngine.enableInEarMonitoring(true);

// Sets the volume of the in-ear monitor. The value ranges between 0 and 100. The default value is 100, which represents the original volume captured by the microphone.
rtcEngine.setInEarMonitoringVolume(80);
```

# Run the project
Run the project on your Android device. You can see both the local and remote video views when you successfully start a one-to-one video call in the app.
