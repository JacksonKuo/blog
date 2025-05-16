---
layout: post
title: "Frida Hooking the Firebase SDK: Part I - Writing the Apps"
date: 2024-12-01
tags: ["instrumentation"]
---

# Problem Statement

How do you use Frida to hook into a client app that uses the Firebase SDK?[^1] This problem can be further broken down into:

* Step 1: **Write a Swift/Obj-C app**
* Step 2: Learn to use Frida
* Step 3: Build a Swift app with Firebase SDK

# Step 1: Write a Swift/Obj-C app

#### Obj-C application: [app-obj-c](https://github.com/JacksonKuo/app-obj-c)

Learning notes on how to use Xcode:
* In order to push don't forget to add a commit message
* To create a Github repo: Source Control navigator > Repositories > Remotes > New `<project>` Remote

```objective-c
// main.m
#import <Foundation/Foundation.h>
#import "HelloWorld.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        HelloWorld *obj = [[HelloWorld alloc] init];

        NSLog(@"Press any key to display the message. Press 'q' to quit.");
        int c = getchar(); // Wait for user input
        if (c == 'q' || c == 'Q') {
            NSLog(@"Exiting...");
            return 0; // Exit the loop if 'q' or 'Q' is pressed
        } else {
            NSString *charAsString = [NSString stringWithFormat:@"%c", c];
            NSString *returnedMessage = [obj sayHello:charAsString];
            NSLog(@"sayHello return: %@", returnedMessage);
        }
        return 0;
    }
}
``` 

#### Swift application: [app-swift](https://github.com/JacksonKuo/app-swift)

Learning notes on adding Obj-C to Swift
* Obj-C is compatible with Swift but requires a `app-swift-Bridging-Header.h`

```swift
// main.swift
import Foundation

print("Input a string: ")
let input = readLine()
let hello = HelloWorld()
let message = hello.sayHello(input) ?? "Default Message"
print("sayHello return:  \(message)")
```

# References
[^1]: [https://kibty.town/blog/arc/](https://kibty.town/blog/arc/)