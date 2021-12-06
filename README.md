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