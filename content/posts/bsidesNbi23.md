---
title: "BsidesNBI Cyber Challenge '23"
date: 2023-11-08
draft: false
---

## Author of Challenges: i, [themadbit](https://twitter.com/themadbit)

This write-up is from the point of view of the creator. There are many ways to kill the proverbial rat and these challenges are no exception.

I created these challenges partly to learn and to share my knowledge with players at the BsidesNBI 2023 Cyber Challenge. Thanks to my team [fr334ks](https://twitter.com/fr334aks), my captain [koimet](https://twitter.com/k0imet_) & friend [Oste](https://twitter.com/oste_ke) for getting me as part of the creator team!

A special shout-out to my university team [cyberkimathi](https://twitter.com/cyberkimathi) for participating and ranking 4th.

![image](https://user-images.githubusercontent.com/84702057/280992022-a8ed0073-6a48-4d8f-9aa1-ac9d97a6e63a.png)

To the winners and all the players, well done!

## General comments (As a CTF player & now, as a creator):

I have found ctfs to be very meaningful and particularly a good way to get new skills, tools and methodologies that can be used in pentesting (and vice-versa).

I have previously interned as a junior security analyst and I credit CTFS for helping me level up, especially in mobile application security.

Most of the tools I use in pentesting are the same tools I use in CTFS with very few exceptions. The aim of writing this debrief is to encourage CTFS players to take CTFS seriously, and use them to judiciously learn.

### Tools:

1. jadx/jadx-gui - Decompilation
2. Genymotion - Android emulator
3. Linux swiss knife - strings | grep | Base64 e.t.c
4. android sdk - adb | logcat | e.t.c
4. Frida

### Chall 1: whatslif3

#### Test: Basic Android app reversing, cryptography (encoding/decoding)

Opening the application launches an activity (mainActivity) that has some text and a question in an `EditTextview`. The next step is to give a random answer and observe the output.


![image](https://user-images.githubusercontent.com/84702057/280998066-d251c0ec-db26-44ae-ace4-6c35c2aaefdf.png)

If the input is incorrect, the app toasts such a message.

![image](https://user-images.githubusercontent.com/84702057/280998904-eb9e0754-abd9-4ae0-9e6c-edcf61a9573a.png)

Time to decompile the app using [jadx-gui](https://github.com/skylot/jadx)

![image](https://user-images.githubusercontent.com/84702057/280999736-29a8aabe-16e7-447b-988e-2bb055217c7c.png)

The mainActivity has a `verifyAnswer` method that gets a the input string and compares it to the Base64 decode of a hardcorded string (`R.string.check_guess`)

Easier now, get the string(/res/values/strings.xml) and decode it.

![image](https://user-images.githubusercontent.com/84702057/281001900-9f2ca126-5a49-4afd-a2ea-bb42aae21ec7.png)

Use decode string to answer question. New actvity is loaded and seems to have the flag.

Time to get a good recipe. [Cyberchef](https://gchq.github.io/CyberChef/) has lots of those. 

![image](https://user-images.githubusercontent.com/84702057/281002852-22870285-0fb7-4a7e-a311-021e7158dd91.png)

Done:? BSidesNRB{c4r3full_hubr1s_w1ll_t4k3_us_0u7}

### Chall 2: Daily Blatha

#### Test: Firebase Misconfig, SQlite DB, SharedPrefs

This was fairly a unique challenge as there were no interactive activities, which meant that most of the work would be in the logic.

![image](https://user-images.githubusercontent.com/84702057/281005218-a0161fe2-8c71-4827-bf0f-d4b7fb812755.png)


Use Jadx-gui to decompile.

**Firebase DB endpoint.**

It is a fundamental pentesting methodology to check if an app uses a Firebase db and if it does, whether it is correctly configured.

In almost all cases, you'll find the db enpoint in the `strings.xml` file. However, in this case, I hardcoded the endpoint in one of the activities (firebaseActivity). Easy to find using jadx-gui *universal search tool.

Result: "https://dailyblatha-default-rtdb.europe-west1.firebasedatabase.app/"

To test for misconfigurations (read access) in the db, append `.json` to the enpoint URL and use curl or a browser to view results.


![image](https://user-images.githubusercontent.com/84702057/281009732-e0e1e06f-277b-44f5-8692-3fe6fc0896e0.png)

**SQlite db**

The SQlite db is natively supported in android and prefered as the local db for storage. For external databases, they'll be added in the assets directory and later initialized if need be.

Solution: Check for db in the assets folder and read its contents.

Done;?

![image](https://user-images.githubusercontent.com/84702057/281013002-2bbd5c97-c134-45a0-8436-08631c1a3d87.png)


**SharedPrefs**

Shared Preferences is a way to store and retrieve small amounts of primitive data as key/value pairs to a file on the device in Android.

In the applications activities, there's a `sharedPrefsActivity` that looks like this:

![image](https://user-images.githubusercontent.com/84702057/281014593-63babe37-9178-428d-a09a-28e12f1e8440.png)

Contains flag_3  that is base64 encoded.

Put the three flags together: BSidesNBI{4ndro1d_db_m1sc0nf1gs_4r3_d4ng3r0usss}


### Challl 3: watchdog

#### Test: Instrumentation using Frida

As I was writing the write-up, I noticed an intended obstacle to solving the challenge. The variable that was to be instrumentated was declared as a `final` variable (as in Java)and that means that it is immutable after initialization.

That makes the instrumentation process a little complex but nonetheless, I'll appreciate it if any of the players share the Frida script(s) they used to solve this challenge.

On running the application, there's an activity with a `MOVE` button which when clicked checks if the light is red and if it is it toasts a message.


<p float="left">
      <img src="https://user-images.githubusercontent.com/84702057/281015945-cb97ffe3-0a91-4856-a008-8f621457f23a.png" width="250" />
  <img src="https://user-images.githubusercontent.com/84702057/281016087-ab41badf-c9f2-4bec-9de2-0827d111d65b.png" width="250" /> 
</p>

Decompile the app to inspect the activity. From the variable names and classes we can tell the app is obfuscated. Go ahead and follow of the declaration of the method `b`

![image](https://user-images.githubusercontent.com/84702057/281181075-a984a788-d5c0-49ec-b2bf-8226a56a10af.png)

In the declaration of the method(`b`), there's a switchcase that checks if `mainActivity.f1258t` is `true`. So this is where the logic is at. If it's true, it toasts the message and if not, the flag is decrypted and made visible in he `mainActivity`.

![image](https://user-images.githubusercontent.com/84702057/281181469-15739822-9ad0-409e-a646-2a28774feada.png)

At this point, the challenge is simple. Change the value of `mainActivity.f1258t = false;` 

Frida is what I use. *However, you can patch the application and change the boolean value to false and when you run the app, you get the flag ([get more context here](https://google.com)).

I used the following Frida script. It is commented, so it's easy to know what it does. (Note: Frida scripts are written in JS)

```js
Java.perform(function () {
    var MainActivity = Java.use('com.bsidesnrb.watchdog.MainActivity');

    // Hook the onCreate method of MainActivity
    MainActivity.onCreate.overload('android.os.Bundle').implementation = function (bundle) {
        // Call the original onCreate method
        this.onCreate(bundle);

        // Attempt to modify the field directly, handle the case where the field might be obfuscated
        try {
            var f = MainActivity.class.getDeclaredField('f1258t');
            f.setAccessible(true);
            f.setBoolean(this, false);
            console.log('Field f1258t set to false successfully.');
        } catch (e) {
            console.log('Field f1258t not found. Attempting to find the obfuscated field name.');

            // If the field name is obfuscated, you might need to iterate over all fields and find the correct one by type
            var fields = MainActivity.class.getDeclaredFields();
            for (var i = 0; i < fields.length; i++) {
                var field = fields[i];
                if (field.getType().getName() === 'boolean') {
                    field.setAccessible(true);
                    field.setBoolean(this, false);
                    console.log('Field name: ' + field.getName() + ', Type: ' + field.getType().getName());
                    console.log('Obfuscated boolean field set to false successfully. Field name: ' + field.getName());
                    break;
                }
            }
        }
    };
});
```

![image](https://user-images.githubusercontent.com/84702057/281190553-4d630033-5fdd-4518-b6c5-e2dd2f6a6d7b.png)


## Bye
