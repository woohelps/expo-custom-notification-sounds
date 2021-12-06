# I want to make a demo android app which can play custom notification sounds on receive FCM notification.

##Here are the details of this demo:
* expo `init expo-custom-notification-sounds` (choose blank template)
* run `expo install expo-notifications` under root of project
* According [the document](https://docs.expo.dev/versions/latest/sdk/notifications/#setting-custom-notification-sounds), add the expo-notifications plugin to your app.json file as below:
```{
  "expo": {
    "plugins": [
      [
        "expo-notifications",
        {
          "sounds": ["./assets/mySoundFile.wav"]
        }
      ]
    ]
  }
}
```
* According [this](https://docs.expo.dev/push-notifications/faq/#i-want-to-play-a-custom-sound), I need to set up [Using FCM for Push Notifications] (https://docs.expo.dev/push-notifications/using-fcm/)
  In my app.json, add an android.googleServicesFile field with the relative path to the google-services.json which I created in the firebase console.
```
{
  ...
  "android": {
    "googleServicesFile": "./google-services.json",
    ...
  }
}
```
* Confirm that my API key in google-services.json has the correct "API restrictions" in the Google Cloud Platform API Credentials console.
  ![my API key in google-services](https://overseas-toronto-1252412068.cos.na-toronto.myqcloud.com/apk/currrent_key.png)
  ![API Key restrictions setting](https://overseas-toronto-1252412068.cos.na-toronto.myqcloud.com/apk/api_key_restrictions_setting.png)
* According [this](https://docs.expo.dev/guides/config-plugins/) run `expo prebuild`. create android native folders
* Add [Demo](https://docs.expo.dev/push-notifications/overview/) code in the App.js
```
import Constants from 'expo-constants';
import * as Notifications from 'expo-notifications';
import React, { useState, useEffect, useRef } from 'react';
import { Text, View, Button, Platform } from 'react-native';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true, // change false to true
    shouldSetBadge: false,
  }),
});

export default function App() {
  const [expoPushToken, setExpoPushToken] = useState('');
  const [notification, setNotification] = useState(false);
  const notificationListener = useRef();
  const responseListener = useRef();

  useEffect(() => {
    registerForPushNotificationsAsync().then(token => setExpoPushToken(token));

    // This listener is fired whenever a notification is received while the app is foregrounded
    notificationListener.current = Notifications.addNotificationReceivedListener(notification => {
      setNotification(notification);
    });

    // This listener is fired whenever a user taps on or interacts with a notification (works when app is foregrounded, backgrounded, or killed)
    responseListener.current = Notifications.addNotificationResponseReceivedListener(response => {
      console.log(response);
    });

    return () => {
      Notifications.removeNotificationSubscription(notificationListener.current);
      Notifications.removeNotificationSubscription(responseListener.current);
    };
  }, []);

  return (
    <View
      style={{
        flex: 1,
        alignItems: 'center',
        justifyContent: 'space-around',
      }}>
      <Text>Your expo push token: {expoPushToken}</Text>
      <View style={{ alignItems: 'center', justifyContent: 'center' }}>
        <Text>Title: {notification && notification.request.content.title} </Text>
        <Text>Body: {notification && notification.request.content.body}</Text>
        <Text>Data: {notification && JSON.stringify(notification.request.content.data)}</Text>
      </View>
      <Button
        title="Press to Send Notification"
        onPress={async () => {
          await sendPushNotification(expoPushToken);
        }}
      />
    </View>
  );
}

// Can use this function below, OR use Expo's Push Notification Tool-> https://expo.dev/notifications
async function sendPushNotification(expoPushToken) {
// Instead of using expo push notification server, using FCM Server here:
 /* 
  const message = {
    to: expoPushToken,
    sound: 'default',
    title: 'Original Title',
    body: 'And here is the body!',
    data: { someData: 'goes here' },
  };

  await fetch('https://exp.host/--/api/v2/push/send', {
    method: 'POST',
    headers: {
      Accept: 'application/json',
      'Accept-encoding': 'gzip, deflate',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(message),
  });
  */
  
  await fetch('https://fcm.googleapis.com/fcm/send', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `key=AAAAVqXLbNU:APA91bHTGb9Ow5EyjHQI1YNxKiXbM_DFMBImV_uscXZrN-gV3bZDmO6u3vRcYsnd1GGQRIin5WSb3xw3oMNk_7FIuxXcna8xfLHScMRxEur1SXHjQS-4NYFDn_5NvUYBfAYxBexV3YsY`,
  },
  body: JSON.stringify({
    to: expoPushToken,
    priority: 'normal',
    data: {
      experienceId: '@yourExpoUsername/yourProjectSlug',
      title: "\uD83D\uDCE7 You've got mail",
      message: 'Hello world! \uD83C\uDF10',
    },
  }),
});
  
}

async function registerForPushNotificationsAsync() {
  let token;
  if (Constants.isDevice) {
    const { status: existingStatus } = await Notifications.getPermissionsAsync();
    let finalStatus = existingStatus;
    if (existingStatus !== 'granted') {
      const { status } = await Notifications.requestPermissionsAsync();
      finalStatus = status;
    }
    if (finalStatus !== 'granted') {
      alert('Failed to get push token for push notification!');
      return;
    }
    // token = (await Notifications.getExpoPushTokenAsync()).data;
    // Since only FCM cupport custom sound, we have to change this to get device token here.
    token = (await Notifications.getDevicePushTokenAsync()).data:
    console.log(token);
  } else {
    alert('Must use physical device for Push Notifications');
  }

  if (Platform.OS === 'android') {
    Notifications.setNotificationChannelAsync('default', {
      name: 'default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
      lightColor: '#FF231F7C',
    });
  }

  return token;
}
```

![FCM-SERVER-KEY](https://overseas-toronto-1252412068.cos.na-toronto.myqcloud.com/apk/fcm-server-key.png)
