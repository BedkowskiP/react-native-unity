# react-native-unity

Unity version 2020.3.38f

Project tested on android studio virtual device: Pixel 2 Android API 33 Google Play | x84_64

### Recreation of my steps to make [@azesmway/react-native-unity](https://github.com/azesmway/react-native-unity#readme) working for me

Project creation:

1. Create new project:

```
npx react-native init <project_name> --template react-native-template-typescript
```

2. Add [@azesmway/react-native-unity](https://github.com/azesmway/react-native-unity#readme) to your project:

```
npm install @azesmway/react-native-unity

or

yarn add @azesmway/react-native-unity
```
### Android

1. Inside `android/` create `local.properties` file with the following line inside:

```
sdk.dir=C:\\Users\\<user_name>\\AppData\\Local\\Android\\Sdk
```
2. Add the following lines to `android/settings.gradle`:
```
include ':unityLibrary'
project(':unityLibrary').projectDir=new File('..\\unity\\builds\\android\\unityLibrary')
```
3. Add into `android/build.gradle`:
```
allprojects {
  repositories {
    // this
    flatDir {
        dirs "${project(':unityLibrary').projectDir}/libs"
    }
// ...
```
4. Add into `android/gradle.properties`:
```
unityStreamingAssets=.unity3d
```
5. Add strings to `android/app/src/main/res/values/strings.xml`:
```
<string name="game_view_content_description">Game view</string>
```
6. Inside project root folder create folders `unity/builds`.


### Unity - project settings
1. Add this code in Unity:
```
using System;
using System.Collections;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using UnityEngine.UI;
using UnityEngine;

public class NativeAPI {
#if UNITY_IOS && !UNITY_EDITOR
  [DllImport("__Internal")]
  public static extern void sendMessageToMobileApp(string message);
#endif
}

public class ButtonBehavior : MonoBehaviour
{
  public void ButtonPressed()
  {
    if (Application.platform == RuntimePlatform.Android)
    {
      using (AndroidJavaClass jc = new AndroidJavaClass("com.azesmwayreactnativeunity.ReactNativeUnityViewManager"))
      {
        jc.CallStatic("sendMessageToMobileApp", "The button has been tapped!");
      }
    }
    else if (Application.platform == RuntimePlatform.IPhonePlayer)
    {
#if UNITY_IOS && !UNITY_EDITOR
      NativeAPI.sendMessageToMobileApp("The button has been tapped!");
#endif
    }
  }
}
```
2. Inside Unity Player settings change your settings to:

![image](https://user-images.githubusercontent.com/10899007/189357244-710b01e8-3e13-4876-a66f-a61868db5cc1.png)
![image](https://user-images.githubusercontent.com/10899007/189357331-5bf9c0bf-c5a0-4ec5-8352-c6e9d4269a2b.png)[^1]

3. Change build settings to:

![image](https://user-images.githubusercontent.com/10899007/189357603-c6eaf09f-6209-45fa-8a8c-43ba8589e158.png)

4. Export project to `unity/builds/android`.
5. After export remove `<intent-filter>...</intent-filter>` from `<project_name>/unity/builds/android/unityLibrary/src/main/AndroidManifest.xml` at unityLibrary to leave only integrated version.

### Usage
## Sample Code
```
import React, { useRef, useEffect } from 'react';

import UnityView from '@azesmway/react-native-unity';
import { View } from 'react-native';

interface IMessage {
  gameObject: string;
  methodName: string;
  message: string;
}

const Unity = () => {
  const unityRef = useRef<UnityView>(null);

  useEffect(() => {
    if (unityRef?.current) {
      const message: IMessage = {
        gameObject: 'gameObject',
        methodName: 'methodName',
        message: 'message',
      };
      unityRef.current.postMessage(message.gameObject, message.methodName, message.message);
    }
  }, []);

  return (
    <View style={{ flex: 1 }}>
      <UnityView
        ref={unityRef}
        style={{ flex: 1 }}
        onUnityMessage={(result) => {
          console.log('onUnityMessage', result.nativeEvent.message)
        }}
      />
    </View>
  );
};

export default Unity;
```

Run project with:

```
npx react-native run-android && npx react-native start
```


[^1]: I'm not sure if x86(Chrome OS) and x86-64(Chrome OS) is required.
