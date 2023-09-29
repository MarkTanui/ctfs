---
title: "Android Challenges: SheHacks Intervarsity '23"
date: 2023-09-29
draft: false
---


# shehacks intervasity '23 with *cyb3rk1m4th1*

![scoreboard](../../assets/shehacks23/image.png)

Played with my Uni team [@cyb3rk1m4th1](https://x.com/@cyberkimathi) and ranked 3rd on the scoreboard. Great challenges and dope team mates too!

I solve android/mobile challenges for teams I play with and slowly getting to rev challs as well.

There were 6 challenges and we solved 4. Here's how I approached the challenges with tools I use in ctfs, bugbounty and general android pentesting.

### Tools:

1. jadx/jadx-gui - Decompilation
2. Genymotion - Android emulator
3. linux swiss knife - strings | grep | e.t.c
4. android sdk - adb | logcat | e.t.c
4. frida - (didn't use in any of the challenges. but important to have)

**Disclaimer** - I will not show how to use any of the tools. You can however install the tools and you know... learn how to use them.

### General Methodology:

I do not strickly adhere to this methodology (like I don't many things in life) but, I always try to. 

- Download the apk
- Install the apk to the android device (drag n drop to install in geny)
- Run the application (to know what it does...)
- Decompile with jadx-gui
- Depending with what we get after decompiling, things take their cause. At this point, we can now start thinking about frida, objection, burpsuite, ghidra, e.t.c

## Challenges

Note: flag format was `flag{}` - Important when greping strings!

### Chall 1: 

![multilingual](../../assets/shehacks23/image-1.png)

Downloaded the app and installed it to my geny device. Launching the app, I'm greeted. There's no other [activity](https://developer.android.com/guide/components/activities/intro-activities), so decompiling is the next step.

![Alt text](../../assets/shehacks23/image-2.png)

#### jadx-gui

Similar to the `main` function in  programming languages, android applications have a MainActivity that is launched first. The mainActivity is then important when reversing as it will always point you to other activities.

In the case of the multilingual app, the mainActivity sets a layout of the app and it's done. So what next? strings

![Alt text](../../assets/shehacks23/image-3.png)

### jadx

there's jadx-gui and jadx. gui for graphical user interface. jadx is a cli utility and I use it when I want to decompile the apk without the need to see the code.

usage: `jadx -d multilingual multilingual.apk`

![Alt text](../../assets/shehacks23/image-4.png)

The application is decompiled to the multilingual directory. The directory has two other directories, sources and resources. Resources has the assests (strings, drawables, layouts, e.t.c) while sources has the code sources.

I used grep recursively on the resources dir in search of `flag`

![Alt text](../../assets/shehacks23/image-5.png)

good. Got a three part flag which was base64 encoded.

![Alt text](../../assets/shehacks23/image-6.png)

### Chall 2: 

![Alt text](../../assets/shehacks23/image-7.png)

#### Run application

No functionality we can interact with, so we move to jadx-gui.

![Alt text](../../assets/shehacks23/image-8.png)

You know about mainActivity, right? Nothing much there. Set's the layout and done. However some important info here: 'insider preview'??

#### jadx-gui

![Alt text](../../assets/shehacks23/image-9.png)

Decompiling the app with jadx-gui, we see an `insiderActivity`. The activity also sets a layout and it's done. But we haven't seen what the layout is!! There's no button or function to get the app to launch the `insiderActivity`. How do we do that?

The `androidManifest` which is an xml file, list all the activities, permissions, intents, receivers and all other android components. This is important because it informs us if activities are exported or not. More on [exported activities](https://developer.android.com/topic/security/risks/android-exported)

The insiderActivity is exported and that means we can access it using the `adb` activity manager (`am`), or create another application to do that (I don't think you want to do this. Please do though).

Learn about adb and how to use it [here](https://developer.android.com/tools/adb#howadbworks)

usage: `adb shell am start com.shehacks.intervasity.earlyaccess/.InsiderActivity`

![Alt text](../../assets/shehacks23/image-10.png)

The command launches the activity and there's the flag!

![Alt text](../../assets/shehacks23/image-11.png)


### Chall 3: 

![Alt text](../../assets/shehacks23/image-12.png)

#### run apk

On running the application, we get a prompt for some password. The password will perhaps retrieve some secured notes and maybe reveal a flag. We however don't the password. Can we find it though?


![Alt text](../../assets/shehacks23/image-13.png)

#### decompile with jadx-gui

Decompiling this apk, reveals that's it [obfuscated](https://levelup.gitconnected.com/android-obfuscation-e608f79f0d09). Nothing to worry about because with jadx-gui *everything is possible.

![Alt text](../../assets/shehacks23/image-14.png)

Time to read through the code and get the logic. Some knowledge with java or ability to read and understand code is important. Anyone doing this kind of thing is certainly smart enough to read code.

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

![Alt text](../assets/shehacks23/image-15.png)

- get's a string (R.string.secret), passes the string to `n()` as the parameter to decode, reverses that and converts it to a string.
- what to do? get the string, decode it and then reverse it.
- voila!! 


To get any referenced strings in the application, go to `/res/values/strings.xml`

![Alt text](../../assets/shehacks23/image-16.png)

##### password
![Alt text](../../assets/shehacks23/image-17.png)

![Alt text](../../assets/shehacks23/image-18.png) 
![Alt text](../../assets/shehacks23/image-19.png)


### Chall 4

![Alt text](../assets/shehacks23/image-20.png)

you know... run the app (check functionalities), decompile, yada yada yada

**note:** when you have an application that has register, login and activate functionalities, call burpsuite.

Also, note that not all apps will need reversing. Interact with the functionalities offered to discover some subtle logic bugs.

##### Methodoology
- Register account
- Try to login (redirected to activate)
- Activate
- ! All these while intercepting with burp suite


![Alt text](../../assets/shehacks23/image-23.png)

The activate functionality is interesting. Asks for a code that is said to be sent to your email. Waited for that email but nothing came!

Intercepting the `request_otp` endpoint with the correct email and password returns a code and a message. Ignore the message. No email is coming!

![Alt text](../../assets/shehacks23/image-24.png)

There, got the code and went a head to activate the account. Time to login.

![Alt text](../../assets/shehacks23/image-25.png)

hmmm... no flag, just a screen with my 0.00 balance (cries in reallife)

Checked the response of the `/login` endpoint and there:

![Alt text](../../assets/shehacks23/image-26.png)

### Chall 5

![Alt text](../../assets/shehacks23/image-27.png)

Didn't solve this!

... thought of IDOR, but the account id was encoded and couldn't get a way to decode it.


![Alt text](../../assets/shehacks23/image-28.png)

### Chall 6


![Alt text](../../assets/shehacks23/image-29.png)

A more secure app than chall 3. Reversed it and got the logic. Used a password to unlock the notes, similar to chall 3. However the validation is done using functions in the libraries.

This required some knowledge of JNI functions and patience. Lots of patience reversing that. I didn't have the patience and run to other challenges. 

Check this writeup by the creator [here](https://evalevanto.github.io/posts/snotes/).  
jni_stuff_makes_me_cry, right? ofc