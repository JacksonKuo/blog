---
layout: post
title: "Frida Hooking the Firebase SDK: Part III - Firebase & Instrumenting"
date: 2024-12-02
tags: ["instrumentation"]
---

**Contents**
* TOC
{:toc}

# Problem Statement

How do you use Frida to hook into a client app that uses the Firebase SDK?[^1] This problem can be further broken down into:

* Step 1: Write a Swift/Obj-C app
* Step 2: Learn to use Frida
* Step 3: **Build a Swift app with Firebase SDK**

# Step 3: Build a Swift app with Firebase SDK

Let's start with definitions. The following are various data storage options for Firebase:

* Cloud Firestore: NoSQL database with data structured around documents and collections[^2]
* Realtime Database: NoSQL database with data stored as JSON[^3] [^4]
* CloudStorage for Firebase: GCP Cloud Storage buckets

#### Build

I created a Firebase project called `swift-project-74dfe` and a Firestore database with the following data and rules:

* Data
    * Collection: `col1`
    * Document: `doc1`
    * Field: `fld1:123`
* Rules
```
rules_version = '2';
service cloud.firestore {
    match /databases/{database}/documents {
        match /{document=**} {
            allow read: if true
        }
    }
}
```
* URL - [https://firestore.googleapis.com/v1/projects/swift-project-74dfe/databases/(default)/documents/col1](https://firestore.googleapis.com/v1/projects/swift-project-74dfe/databases/(default)/documents/col1)
```
{
    "documents": [
        {
            "name": "projects/swift-project-74dfe/databases/(default)/documents/col1/doc1",
            "fields": {
            "fld1": {
                "stringValue": "123"
            }
            },
            "createTime": "2024-12-02T04:27:32.358772Z",
            "updateTime": "2024-12-02T04:27:32.358772Z"
        }
    ]
}
```

#### Firebase App Setup[^5]

* New > Project > App
    * Organization Identifier: `com.jkuo`
    * Interface: SwiftUI
    * Language: Swift
    * Testing System: None
* Add Firebase SDK:
    * File > Add Package: `https://github.com/firebase/firebase-ios-sdk`
* Image Modules:
    Project workspace > Frameworks, Libraries, and Embedded Content > Add `FirebaseCore`, `FirebaseFirestore`
* Add Firebase to your Apple app
    * Project Overview > Project Settings > Add App
    * Apple bundle id: `com.jkuo.app-firebase`
* Add `GoogleService-Info.plist` to workspace
    * > Unlike how API keys are typically used, API keys for Firebase services are not used to control access to backend resources; that can only be done with Firebase Security Rules[^6]
* Add Outgoing Network Connections
    * Project workspace > Signing & Capabilities
    * App Sandbox > Network
    * Outgoing Connections (Client)
* Initialization code[^7]
    * [app_firebase.swift](https://github.com/JacksonKuo/app-firebase/blob/main/app-firebase/app_firebase.swift)
    * Create an AppDelegate and App struct
    * Don't forget to add the Adaptor: `@NSApplicationDelegateAdaptor(AppDelegate.self) var appDelegate`
    * Configure a FirebaseApp
* Create button to trigger DB read
    * [ContentView.swift](https://github.com/JacksonKuo/app-firebase/blob/main/app-firebase/ContentView.swift)

#### Test Instrumentation

Run `app-firebase` and attach JS interceptors:
* `cd /Users/jacksonkuo/Library/Developer/Xcode/DerivedData/app-firebase-*/Build/Products/Debug/app-firebase.app/Contents/MacOS`
* `frida app-firebase -l /Users/jacksonkuo/workspace/scripts-frida/intercept-firestore.js`[^8]
* Looks like under the covers the Firestore module uses Obj-C to make the document fetch calls
* Note that `frida-trace` really doesn't not like SwiftUI and will fail to run the JS interceptors. However attaching via `frida` and using the REPL still works fine 


```javascript
// intercept-firestore.js
var documentWithPath = ObjC.classes.FIRCollectionReference["- documentWithPath:"];
var collectionWithPath = ObjC.classes.FIRFirestore["- collectionWithPath:"];

Interceptor.attach(documentWithPath.implementation, {
  onEnter: function (args) {
    var message = ObjC.Object(args[2]);
    console.log(
      '\n[FIRCollectionReference documentWithPath:@"' + message.toString() +'"]'
    );
  },
});
Interceptor.attach(collectionWithPath.implementation, {
  onEnter: function (args) {
    var message = ObjC.Object(args[2]);
    console.log(
      '\n[FIRFireStore collectionWithPath:@"' + message.toString() + '"]'
    );
  },
});
```

**\*\*Output\*\***

```
[Local::app-firebase ]->
[FIRFireStore collectionWithPath:@"col1"]

[FIRCollectionReference documentWithPath:@"doc1"]
```

# References
[^1]: [https://kibty.town/blog/arc/](https://kibty.town/blog/arc/)
[^2]: [https://firebase.google.com/docs/firestore](https://firebase.google.com/docs/firestore)
[^3]: [https://firebase.google.com/docs/database](https://firebase.google.com/docs/database)
[^4]: [https://firebase.blog/posts/2017/10/cloud-firestore-for-rtdb-developers](https://firebase.blog/posts/2017/10/cloud-firestore-for-rtdb-developers)
[^5]: [https://firebase.google.com/docs/ios/setup](https://firebase.google.com/docs/ios/setup)
[^6]: [https://firebase.google.com/docs/projects/api-keys](https://firebase.google.com/docs/projects/api-keys)
[^7]: [https://github.com/JacksonKuo/app-firebase](https://github.com/JacksonKuo/app-firebase)
[^8]: [https://medium.com/@wabz/reverse-engineering-the-australian-governments-coronavirus-ios-application-c08790e895e9](https://medium.com/@wabz/reverse-engineering-the-australian-governments-coronavirus-ios-application-c08790e895e9)