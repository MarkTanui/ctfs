---
title: "SheHacks Intervarsity '23: Android Challenges"
date: 2023-09-29
draft: false
---


## shehacks intervasity '23 with *cyb3rk1m4th1*

![image](https://user-images.githubusercontent.com/84702057/271697776-71faf348-2e40-46cb-a5a5-c46e9fe98829.png)


Played with my Uni team [@cyb3rk1m4th1](https://x.com/@cyberkimathi) and ranked 3rd on the scoreboard. Great challenges and dope team mates too!

I solve android/mobile challenges for teams I play with and slowly getting to rev challs as well.

There were 6 challenges and we solved 4. Here's how I approached the challenges with tools I use in ctfs, bugbounty and general android pentesting.

### Tools:

1. jadx/jadx-gui - Decompilation
2. Genymotion - Android emulator
3. linux swiss knife - strings | grep | e.t.c
4. android sdk - adb | logcat | e.t.c
4. frida - (didn't use in any of the challenges. but important to have)

**Disclaimer** - I will not show (installation/tutorial) on how to use any of the tools. You can however install the tools and you know... learn how to use them.

### General Methodology:

I do not strickly adhere to this methodology (like I don't many things in life) but, I always try to. 

- Download the apk
- Install the apk to the android device (drag n drop to install in geny)
- Run the application (to know what it does...)
- Decompile with jadx-gui
- Depending with what we get after decompiling, things take their course. At this point, we can now start thinking about frida, objection, burpsuite, ghidra, e.t.c

## Challenges

Note: flag format was `flag{}` - Important when greping strings!

### Chall 1: 

![image-1](https://user-images.githubusercontent.com/84702057/271698241-0df462e4-92f5-493e-bf19-05b3ffcae88a.png)


Downloaded the app and installed it to my geny device. Launching the app, I'm greeted. There's no other [activity](https://developer.android.com/guide/components/activities/intro-activities), so decompiling is the next step.

![image-2](https://user-images.githubusercontent.com/84702057/271698529-1ad6c5ba-47e0-4cfd-a326-d2d88ae09814.png)


#### jadx-gui

Similar to the `main` function in  programming languages, android applications have a `mainActivity` that is launched first. The mainActivity is then important when reversing as it will always point you to other activities.

In the case of the multilingual app, the `mainActivity` sets a layout of the app and it's done. So what next? `strings`

![image-3](https://user-images.githubusercontent.com/84702057/271698637-1fc6dbc2-2b13-45a5-8578-516865b499b3.png)


### jadx

there's jadx-gui and jadx. gui for graphical user interface. jadx is a cli utility and I use it when I want to decompile the apk without the need to see the code.

usage: `jadx -d multilingual multilingual.apk`

![image-4](https://user-images.githubusercontent.com/84702057/271698784-118a8f5f-ad47-42d0-9b25-d937e2d64135.png)


The application is decompiled to the multilingual directory. The decompilation creates two other directories, `sources` and `resources`. Resources has the assests (strings, drawables, layouts, e.t.c) while sources has the code sources.

I used grep recursively on the resources dir in search of `flag`

![image-5](https://user-images.githubusercontent.com/84702057/271698814-4ec89dd2-ac88-4226-93ff-790a2e340784.png)


good. Got a three part flag which was base64 encoded.

![image-6](https://user-images.githubusercontent.com/84702057/271698836-b19a5098-4e4a-48a5-9bb9-394c5777c24e.png)


### Chall 2: 

![image-7](https://user-images.githubusercontent.com/84702057/271698852-8842190d-1c49-4f24-bffe-6b2b61f85cce.png)


#### Run application

There's no interaction we can have with the mainActivity, so we move to jadx-gui.

![image-8](https://user-images.githubusercontent.com/84702057/271698873-f7ff0727-dd84-40e9-84ab-b57af653a67b.png)

You now know about mainActivity, right? Nothing much there. Set's the layout and done. However some important info here: 'insider preview'.?

#### jadx-gui

![image-9](https://user-images.githubusercontent.com/84702057/271698892-784aa955-d5f4-424e-bacf-946ea0a50ee5.png)

Decompiling the app with jadx-gui, we see an `InsiderActivity`. The activity also sets a layout and it's done. But unlike the mainActivity, we haven't seen what is displayed by the InsiderActivity. There's no button or function to get the app to launch the `insiderActivity`. How do we do that?

The `androidManifest` which is an xml file, lists all the activities, permissions, intents, receivers and all other android components. This is important because it informs us if activities are exported or not. More on [exported activities](https://developer.android.com/topic/security/risks/android-exported)

The InsiderActivity is exported and that means we can access it using the `adb`'s activity manager (`am`), or create another application to do that (I don't think you want to do this. Please do though).

Learn about adb and how to use it [here](https://developer.android.com/tools/adb#howadbworks)

usage: `adb shell am start com.shehacks.intervasity.earlyaccess/.InsiderActivity`

![image-10](https://user-images.githubusercontent.com/84702057/271698944-1906fc0a-79bd-4b2b-a14a-5a7f23542fcb.png)

The command launches the activity and there's the flag!

![image-11](https://user-images.githubusercontent.com/84702057/271698963-9f773403-30aa-415d-9a1d-714ed6d925b3.png)


### Chall 3: 

![image-12](https://user-images.githubusercontent.com/84702057/271698993-d075c2e7-917f-45ed-8a93-2599ec2cda63.png)

#### run apk

On running the application, we get a prompt for some password. The password will perhaps retrieve some secured notes and maybe reveal a flag. We however don't know the password. Can we find it though?


![image-13](https://user-images.githubusercontent.com/84702057/271699016-94ff5ec1-1e86-475b-b867-381c84f6a7d7.png)

#### decompile with jadx-gui

Decompiling this apk, reveals that's it [obfuscated](https://levelup.gitconnected.com/android-obfuscation-e608f79f0d09). Nothing to worry about because with jadx-gui *everything is possible.

![image-14](https://user-images.githubusercontent.com/84702057/271699044-6f6e40ed-101f-45a5-ac33-fe6f3df0f36d.png)

Time to read through the code and get the logic. Some knowledge with java or ability to read and understand code is important. (Anyone doing this kind of thing is certainly smart enough to read code.)

##### The mainActivity

- has two methods `n()` and `o()`

`n()` - decodes a base64 encoded string and returns the decode string. okay...

```
    public static String n(String str) {
        byte[] bArr = new byte[0];
        try {
            bArr = Base64.decode(str.getBytes("UTF-8"), 0);
        } catch (UnsupportedEncodingException unused) {
            Log.e("MainActivity", "decodeBase64: Error occurred decoding base64 string: " + str);
        }
        return new String(bArr);
    }
```

`o()` - takes care of what you see on the screen. text et.al

At this point, we don't see any usage of `n()` in the mainActivity. Hunting it's usage with jadx-gui is easy. Rightclick the method and choose `Find Usage`.

Found it's usage in another class and there laid it's logic.

![image-15](https://user-images.githubusercontent.com/84702057/271699068-e2627a2a-078d-46a4-a793-a0a4b14f94f7.png)

- get's a string (R.string.secret), passes the string to `n()` as the parameter to decode, reverses that and converts it to a string.
- what to do? get the string, decode it and then reverse it.
- voila!! 


To get any referenced strings in the application, go to `/res/values/strings.xml`

![image-16](https://user-images.githubusercontent.com/84702057/271699096-4fcdbdba-789e-408d-8139-fe210bbc1965.png)

##### password
![image-17](https://user-images.githubusercontent.com/84702057/271699117-309a007f-d1d6-4fd1-9295-cd81798884ae.png)

![image-18](https://user-images.githubusercontent.com/84702057/271699142-b0fd43bb-ca1a-4a90-9613-fecad29ec143.png)
![image-19](https://user-images.githubusercontent.com/84702057/271699171-74e96e8f-a2c1-4624-a1bb-9b1eec689e92.png)


### Chall 4

![image-20](https://user-images.githubusercontent.com/84702057/271699204-662194b1-d9ef-492a-9609-43785bd2f466.png)

you know... run the app (check functionalities), decompile, yada yada yada

**note:** when you have an application that has register, login and activate functionalities, call burpsuite.

Also, note that not all apps will need reversing. Interacting with the functionalities offered may lead to the discovery of subtle logic bugs.

##### Methodology
- Register account
- Try to login (redirected to activate)
- Activate
- ! All these while intercepting with burp suite  

![image-23](https://user-images.githubusercontent.com/84702057/271699272-ba121c08-00d2-4e4d-a69a-22b3573052cc.png)



The activate functionality is interesting. Asks for a code that is said to be sent to your email. Waited for that email but nothing came!

Intercepting the `request_otp` endpoint with the correct email and password returns a code and a message. Ignore the message. No email is coming!

![image-24](https://user-images.githubusercontent.com/84702057/271699294-3c58ea8a-c71c-4d7f-80a6-e605723c4eac.png)


There, got the code and went a head to activate the account with it. Time to login.

![image-25](https://user-images.githubusercontent.com/84702057/271699313-af90d4af-8f0d-476a-9a15-9165d393a123.png)

hmmm... no flag, just a screen with my 0.00 balance (cries in real life)

Checked the response of the `/login` endpoint and there:

![image-26](https://user-images.githubusercontent.com/84702057/271699332-46257c6d-b3a3-4c59-850a-746193ccf954.png)

### Chall 5

![image-27](https://user-images.githubusercontent.com/84702057/271699350-43b407a3-1d30-435e-bea5-a83d537a5711.png)

A continuation of chall 4. Didn't solve this though!

... thought of IDOR, but the account id was encoded and couldn't get a way to decode it.


![image-28](https://user-images.githubusercontent.com/84702057/271699379-9ba52da1-94d5-45bd-9386-55a7f2ba24cb.png)

### Chall 6


![image-29](https://user-images.githubusercontent.com/84702057/271699404-6943f982-e45f-4f82-816d-215694ac6a7c.png)

A more secure app than chall 3. Reversed it and got the logic. Uses a password to unlock the notes, similar to chall 3. However the validation is done using functions in the libraries.

This required some knowledge of JNI functions and patience. Lots of patience reversing that. I didn't have the patience and run to other challenges. 

Check this great writeup by the [levanto](https://twitter.com/levanto_0) [**here**](https://evalevanto.github.io/posts/snotes/).  

jni_stuff_makes_me_cry, right? ofc