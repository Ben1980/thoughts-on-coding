---
title: Introduction into Logging with Loguru
description: Post about thread save logging with C++ and loguru.
date: 2019-03-12
tags:
  - c++
  - library
  - logging
  - thread-safe
layout: layouts/post.njk
image: /img/log-2.png
---

This time I would like to give a short introduction into a nice little library I just encountered. It's called [loguru][1] and it's a lightweight, Thread-Safe, logging library with an impressive good written [documentation][2] and human-readable output.

![Hero Image: Logging excerpt](/img/log-2.png)

To use it all you need to do is to add a single header and source file to compile with. Unfortunately, that's, in my opinion, also a drawback. Yes, it's easier to install and usable in training, but it can't be exchanged by whoever is operating the system after deployment. In such cases, where the operator wants to define how and what to use for logging, a generic logging interface (facade pattern), such as [slf4cxx][3] which is similar to its java pendant [slf4j][4], would be preferable.

Loguru is supporting a various number of features, such as callbacks for logging and fatal errors, verbosity levels, assertions and aborts, and stack traces in case of aborts. It even supports [{fmt}][5]. Everyone who ever was used to java/spring log outputs will recognize its similarities.

```cpp
#include <thread>
#include "loguru.hpp"
#include "loguru.cpp"

void sleep(int ms)
{
    //We can also inject parameters into the logging strings
    VLOG_F(0, "Sleeping for %d ms", ms);
    std::this_thread::sleep_for(std::chrono::milliseconds(ms));
}

void complex()
{
    //LOG_SCOPE_F is indenting the following logs
    LOG_SCOPE_F(INFO, "Preparing complex calculation");

    //VLOG_F can take a dynamic defined verbosity level
    VLOG_F(0, "Heating up CPU");
    sleep(500);
    std::thread([](){
        loguru::set_thread_name("complex lambda");
       
        const bool value = true;
        LOG_IF_F(INFO, value, "This log is printed inside another thread");
    }).join();
}

void crashingFunction(int index)
{
    //ERROR_CONTEXT is logging certain arguments in case of a crash
    ERROR_CONTEXT("Computing with index", index);

    //Asserts are also possible
    std::vector<std::string> list;
    CHECK_F(index > 1, "Oh no, wrong index, index is %d!!", index);
}

int main(int argc, char* argv[])
{
    //Time stamping begin of logging and we can forward cli parameters such as -v (verbosity level) to loguru
    loguru::init(argc, argv);

    //Additional, if we need we can also put the logs into a defined file
    loguru::add_file("important.log", loguru::Truncate, loguru::Verbosity_INFO);

    LOG_F(INFO, "We are starting our complex threaded computation!");
    complex();
    LOG_F(INFO, "Complex computation done!");

    crashingFunction(-1);

    return 0;
}
```

```bash
date       time         ( uptime  ) [ thread name/id ]                   file:line     v| 
2019-03-11 21:46:14.591 (   0.000s) [main thread     ]             loguru.cpp:587   INFO| arguments: /mnt/c/Develop/LoggingWithLoguru/cmake-build-debug-wsl/LoggingWithLoguru
2019-03-11 21:46:14.591 (   0.000s) [main thread     ]             loguru.cpp:590   INFO| Current dir: /mnt/c/Develop/LoggingWithLoguru/cmake-build-debug-wsl
2019-03-11 21:46:14.591 (   0.000s) [main thread     ]             loguru.cpp:592   INFO| stderr verbosity: 0
2019-03-11 21:46:14.592 (   0.000s) [main thread     ]             loguru.cpp:593   INFO| -----------------------------------
2019-03-11 21:46:14.592 (   0.001s) [main thread     ]             loguru.cpp:751   INFO| Logging to 'important.log', mode: 'w', verbosity: 0
2019-03-11 21:46:14.592 (   0.001s) [main thread     ]               main.cpp:47    INFO| We are starting our complex threaded computation!
2019-03-11 21:46:14.592 (   0.001s) [main thread     ]               main.cpp:15    INFO| { Preparing complex calculation
2019-03-11 21:46:14.592 (   0.001s) [main thread     ]               main.cpp:18    INFO| .   Heating up CPU
2019-03-11 21:46:14.592 (   0.001s) [main thread     ]               main.cpp:8     INFO| .   Sleeping for 500 ms
2019-03-11 21:46:15.094 (   0.502s) [complex lambda  ]               main.cpp:25    INFO| .   This log is printed inside another thread
2019-03-11 21:46:15.094 (   0.503s) [main thread     ]               main.cpp:15    INFO| } 0.502 s: Preparing complex calculation
2019-03-11 21:46:15.094 (   0.503s) [main thread     ]               main.cpp:49    INFO| Complex computation done!
Stack trace:
3       0x7ff08b806a7a /mnt/c/Develop/LoggingWithLoguru/cmake-build-debug-wsl/LoggingWithLoguru(+0x6a7a) [0x7ff08b806a7a]
2       0x7ff08a641b97 __libc_start_main + 231
1       0x7ff08b80cfb5 /mnt/c/Develop/LoggingWithLoguru/cmake-build-debug-wsl/LoggingWithLoguru(+0xcfb5) [0x7ff08b80cfb5]
0       0x7ff08b80ceb5 /mnt/c/Develop/LoggingWithLoguru/cmake-build-debug-wsl/LoggingWithLoguru(+0xceb5) [0x7ff08b80ceb5]
------------------------------------------------
[ErrorContext]                main.cpp:32    Computing with index: -1
------------------------------------------------
2019-03-11 21:46:15.094 (   0.503s) [main thread     ]               main.cpp:36    FATL| CHECK FAILED:  index > 1  Oh no, wrong index, index is -1!!
bash: line 1:  2324 Aborted                 (core dumped) env "JETBRAINS_REMOTE_RUN"="1" '/mnt/c/Develop/LoggingWithLoguru/cmake-build-debug-wsl/LoggingWithLoguru'

Process finished with exit code 134
```

As you can see it's pretty straight forward to use. We are not only logging several messages on INFO verbosity level, but also a message in a named thread called "complex lambda". If we wouldn't have defined the thread name with `loguru::set_thread_name("complex lambda")`, loguru would state the name of a thread with a hex id. The main thread gets its name by calling `loguru::init(...)`. Because our small tool is crashing, loguru is printing us a stack trace which in my opinion is not as helpful as expected, but with `ERROR_CONTEXT` we get a little better output.

That's it for now with this post. We have now a short introduction into a, until now, rather unknown, but promising, logging library. Loguru is not only capable of producing Thread-Safe and human readable logging messages but also provides a very simple and handy interface to use.

[1]: hhttps://github.com/emilk/loguru/blob/master/README.md
[2]: https://emilk.github.io/loguru/index.html
[3]: https://archive.codeplex.com/?p=slf4cxx
[4]: https://www.slf4j.org
[5]: https://github.com/fmtlib/fmt