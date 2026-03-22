---
layout: post
title: "Python: Threading"
date: 2026-3-22
tags: ["python"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Breakdown threading in Python primarily following this guide: [https://realpython.com/intro-to-python-threading/](https://realpython.com/intro-to-python-threading/). With examples being here: [https://github.com/JacksonKuo/scripts-threading](https://github.com/JacksonKuo/scripts-threading)

# Python Threads
Apparently there's a few ways to do threads in Python:

* thread
* threading
* multiprocessing
* asyncio

Good candidates for threads include:

> Tasks that spend much of their time waiting for external events are generally good candidates for threading. Problems that require heavy CPU computation and spend little time waiting for external events might not run faster at all.[^1]

Perfect, I have a great case subject that would fit.

#### Daemon Threads
This is a little confusing to me. By default `threading.Thread` uses non-daemon threads.

>  Daemon threads are abruptly stopped at shutdown [^2]

> A non-daemon thread must finish its work before the program can exit[^3]

So a daemon thread, primarily matters when the main program exits. 

#### Threading and Joins
`thread.join` is basically telling the caller, in our case `main`, hey you can't continue until this specific thread finishes. 

#### Threading module
```python
    threads = list()
    for index in range(15):
        x = threading.Thread(target=thread_function, args=(index,))
        threads.append(x)
        x.start()

    for index, thread in enumerate(threads):
        thread.join()
```

Here the first `t[0].join()` will halt until the `t[0]` thread is done. However all the other `t[x]` are still running and may have completed already. Subsequent `t[x+1].join()` if finished, then will move to to `t[x+2].join()`. This loop ensures all threads are done. 

So a join at the end of main kinda acts like a non-daemon thread. 

Also `args` accepts either a tuple or a list. 

Also `import concurrent.futures` has `ThreadPoolExecutor` and is recommended so you don't accident forget to call `join`. However the `ThreadPoolExecutor` uses the `executor.map()` which is kinda confusing to look at. 

#### Content Manager
Python has something called context managers: [https://www.geeksforgeeks.org/python/context-manager-in-python/](https://www.geeksforgeeks.org/python/context-manager-in-python/)

> Python’s context managers provide a neat way to automatically set up and clean up resources, ensuring they’re properly managed even if errors occur.

Implemented using `with:` and replaces the `try` and `finally` and will auto clean up when exiting the scope.

`ThreadPoolExecutor` is a content manager for `.join()`.

#### Locks
So I think technically python `dict` is safe for atomic writes. But it's generally good practice to include a lock on the shared dict. 

```python
def wrap_thread_function(name):
    x = thread_function(name)
    lock.acquire()
    try:
        shared_dict[name] = x
    finally:
        lock.release()
```

#### Case Subject
So the final code for my use case ended up being:

```python
def _run_threaded(repos, workers):
    all_secrets = {}
    sem = threading.Semaphore(workers)

    def _worker(count, repo):
        with sem:
            _log(...)
            state[repo] = function(repo)
    
    threads = []
    for count, repo in enumerate(repos)
        t = threading.Thread(target=_worker, args=(count,))
        threads.append(t)
        t.start()
    for t in threads:
        t.join()
```

# References
[^1]: [https://realpython.com/intro-to-python-threading/](https://realpython.com/intro-to-python-threading/)

[^2]: [https://docs.python.org/3/library/threading.html#thread-objects](https://docs.python.org/3/library/threading.html#thread-objects)

[^3]: [https://medium.com/@abhishekjainindore24/advanced-python-7-daemon-vs-non-daemon-threads-graceful-thread-shutdown-060ebc21b4f7](https://medium.com/@abhishekjainindore24/advanced-python-7-daemon-vs-non-daemon-threads-graceful-thread-shutdown-060ebc21b4f7)