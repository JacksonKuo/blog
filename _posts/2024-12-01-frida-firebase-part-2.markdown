---
layout: post
title: "Frida Hooking the Firebase SDK: Part II - Learning Frida"
date: 2024-12-01
tags: ["instrumentation"]
---

**Contents**
* TOC
{:toc}

# Problem Statement

How do you use Frida to hook into a client app that uses the Firebase SDK?[^1] This problem can be further broken down into:

* Step 1: Write a Swift/Obj-C app
* Step 2: **Learn to use Frida**
* Step 3: Build a Swift app with Firebase SDK

# Step 2: Learn to use Frida

There are a couple of important areas in understanding and using Frida:
* frida-trace
* Frida CLI
* JavaScript API
* Python

#### frida-trace

The frida-trace tool can be used to quickly find if a program calls certain functions: [^2] [^3]
* `frida-trace` will start a new process:
* `frida-trace -m "-[HelloWorld **]" -f app-obj-c`
* Looks like `frida-trace` will automatically provide user input and then always terminate
```
jacksonkuo@Mac Debug % frida-trace -m "-[HelloWorld **]" -f app-obj-c
Instrumenting...
-[HelloWorld sayHello:]: Loaded handler at "/Users/jacksonkuo/Library/Developer/Xcode/DerivedData/app-obj-c-fdizadrqsywpebegzhhnhkcblhfj/Build/Products/Debug/__handlers__/HelloWorld/sayHello_.js"
Started tracing 1 function. Web UI available at http://localhost:63344/
2024-12-01 19:06:22.915 app-obj-c[63453:9711305] Press any key to display the message. Press 'q' to quit.
2024-12-01 19:06:22.916 app-obj-c[63453:9711305] sayHello arg: ÿ
2024-12-01 19:06:22.916 app-obj-c[63453:9711305] sayHello return: sayHello arg: ÿ
           /* TID 0x103 */
     8 ms  -[HelloWorld sayHello:0x6000006c89a0]
Process terminated
```
* After `frida-trace` is run, a `__handlers__` folder will be created with auto-generated JS stub files that can be used for hooking
```
defineHandler({
    onEnter(log, args, state) {
        log(`-[HelloWorld sayHello]`);
    },
    onLeave(log, retval, state) {
    }
});
```

#### Frida CLI

`frida` provides a REPL interface that attaches to an already running process[^4]. `frida` is how we want to hook functions, whereas `frida-trace` is how we want to quickly find if certain functions are called.
* Xcode builds are saved in the following folder: `/Users/jacksonkuo/Library/Developer/Xcode/DerivedData/<project-name>/Build/Products/Debug`
* Run the app `./app-obj-c`
* `frida app-obj-c -l /Users/jacksonkuo/workspace/scripts-frida/intercept-helloworld.js`
* Frida will search for a running application called `app-obj-c`, `-l` will load the JS file and attach the interceptors

Useful REPL search queries:
* `ObjC.available`: check if a Objective-C runtime loaded
* `ObjC.classes`: list currently available classes

#### JavaScript API

* Writing our own JS file to attach to a running process:[^5]
* Interceptors will hook on entry and exit of the `sayHello()` function and print out the arguments and return value
* `retval` is a NativePointer 

```javascript
// intercept-helloworld.js
var hello = ObjC.classes.HelloWorld["- sayHello:"];

Interceptor.attach(hello.implementation, {
  onEnter: function (args) {
    var message = ObjC.Object(args[2]);
    console.log(
      '\n[ HelloWorld hook parameter value: "' + message.toString() + '"]'
    );
  },
  onLeave: function (retval) {
    console.log(
      '\n[ HelloWorld hook return value: "' + new ObjC.Object(ptr(retval)).toString() + '"]'
    );
  },
});
```

#### Python

Python shim to attach the JS script to the `app-obj-c` process[^6]

```python
# attach-obj-c.py
import frida
import sys

def on_message(message, data):
    print("[{}] => {}".format(message, data))

def main(target_process):
    session = frida.attach(target_process)

    script = session.create_script("""
        var hello = ObjC.classes.HelloWorld['- sayHello:'];
        Interceptor.attach(hello.implementation, {
          onEnter(args) {
            var message = new ObjC.Object(args[2]);
            console.log('message: ' + message.toString());
          }
        });
    """)
    script.on("message", on_message)
    script.load()
    print("[!] Ctrl+D or Ctrl+Z to detach from instrumented program.\n\n")
    sys.stdin.read()
    session.detach()

if __name__ == "__main__":
   main("app-obj-c")
```

# References

[^1]: [https://kibty.town/blog/arc/](https://kibty.town/blog/arc/)
[^2]: [https://frida.re/docs/frida-cli/](https://frida.re/docs/frida-cli/)
[^3]: [https://medium.com/@wabz/reverse-engineering-the-australian-governments-coronavirus-ios-application-c08790e895e9](https://medium.com/@wabz/reverse-engineering-the-australian-governments-coronavirus-ios-application-c08790e895e9)
[^4]: [https://frida.re/docs/javascript-api/](https://frida.re/docs/javascript-api/)
[^5]: [https://frida.re/docs/examples/macos/](https://frida.re/docs/examples/macos/)
[^6]: [https://learnfrida.info/macos/](https://learnfrida.info/macos/)
