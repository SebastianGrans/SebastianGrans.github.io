---
layout: post
title: Installing OpenCV on MacOS 10.11 with Homebrew
---

<center>
<img src="{{ site.baseurl }}/images/opencv.jpg" alt="OpenCV Logo" style="display: block;"/>
</center>
## Intro
I wanted to play around with OpenCV, so I found [this](https://opencv-java-tutorials.readthedocs.io/en/latest/01-installing-opencv-for-java.html#install-opencv-3-x-under-macos) guide. Everything seemed to work fine, but when brew started to install `tbb` I got the error below. 

Apparently, as of August 2018, MacOS 10.11 is no longer supported by Homebrew nor Apple. Apparently I should upgrade, but I'm to lazy to do that now. So if there is anyone as lazy as me and also want to install OpenCV on their computer: This story is for you. 

```
~/L/C/Homebrew ❯❯❯ brew install tbb
Updating Homebrew...
==> Downloading https://github.com/01org/tbb/archive/2019_U1.tar.gz
Already downloaded: /Users/grans/Library/Caches/Homebrew/downloads/299075a15748661fc223991466252f4966e8af064f28b77e889930b096b3001b--tbb-2019_U1.tar.gz
==> make tbb_build_prefix=BUILDPREFIX compiler=clang
==> make tbb_build_prefix=BUILDPREFIX compiler=clang extra_inc=big_iron.inc
==> python -c import setuptools... --no-user-cfg install --prefix=/usr/local/Cel
Last 15 lines from /Users/grans/Library/Logs/Homebrew/tbb/03.python:
swig -python -c++ -O -threads -I/usr/local/Cellar/tbb/2019_U1/include -o tbb/api_wrap.cpp tbb/api.i
creating build
creating build/temp.macosx-10.11-x86_64-2.7
creating build/temp.macosx-10.11-x86_64-2.7/tbb
clang -fno-strict-aliasing -fno-common -dynamic -g -O2 -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -I/usr/local/Cellar/tbb/2019_U1/include -I/usr/local/include -I/usr/local/opt/openssl/include -I/usr/local/opt/sqlite/include -I/usr/local/Cellar/python@2/2.7.15_1/Frameworks/Python.framework/Versions/2.7/include/python2.7 -c tbb/api_wrap.cpp -o build/temp.macosx-10.11-x86_64-2.7/tbb/api_wrap.o -std=c++11 -Wno-unused-variable
In file included from tbb/api_wrap.cpp:3137:
In file included from /usr/local/Cellar/tbb/2019_U1/include/tbb/tbb.h:83:
/usr/local/Cellar/tbb/2019_U1/include/tbb/task_group.h:132:53: error: 'uncaught_exceptions' is unavailable: introduced in macOS 10.12
            bool stack_unwinding_in_progress = std::uncaught_exceptions() > 0;
                                                    ^
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1/exception:119:63: note: 'uncaught_exceptions' has been explicitly marked unavailable here
_LIBCPP_FUNC_VIS _LIBCPP_AVAILABILITY_UNCAUGHT_EXCEPTIONS int uncaught_exceptions() _NOEXCEPT;
                                                              ^
1 error generated.
error: command 'clang' failed with exit status 1

Do not report this issue to Homebrew/brew or Homebrew/core!


Error: You are using macOS 10.11.
We (and Apple) do not provide support for this old version.
You will encounter build failures and other breakages.
Please create pull-requests instead of asking for help on Homebrew's
GitHub, Discourse, Twitter or IRC. As you are running this old version,
you are responsible for resolving any issues you experience.
```

## The solution
After some searching I found [this](https://github.com/catchorg/Catch2/issues/1218) thread which made me think I could solve this if I had access to the source. 

The source code is cached by Homebrew and the path to the location can be found using the command `brew --cache`. For me it was: `/Users/<user>/Library/Caches/Homebrew`. 

In that folder is a symlink `tbb-2019_U1.tar.gz` which pointed to a file located in `/Users/<user>/Library/Caches/Homebrew/downloads`. With the long name:

`299075a15748661fc223991466252f4966e8af064f28b77e889930b096b3001b--tbb-2019_U1.tar.gz`


I extracted that, and in the file `task_group.h` we find the culprit:

```c++
#if __TBB_CPP17_UNCAUGHT_EXCEPTIONS_PRESENT
            bool stack_unwinding_in_progress = std::uncaught_exceptions() > 0;
#else
            bool stack_unwinding_in_progress = std::uncaught_exception();
#endif
```

According to the link above, Clang on MacOS 10.11 isn't fully C++17 compliant. </br>¯\\\_(ツ)_/¯

After some digging I found that `__TBB_CPP17_UNCAUGHT_EXCEPTIONS_PRESENT` is defined on line 340 in `tbb_config.h`. I'm not proficient in C++, so I simply did an ugly hack and commented out the definition. 

```c++
// #define __TBB_CPP17_UNCAUGHT_EXCEPTIONS_PRESENT             (_MSC_VER >= 1900 || __GLIBCXX__ && __cpp_lib_uncaught_exceptions || _LIBCPP_VERSION >= 3700)
```

I saved the file and compressed it to a `tar.gz` file. I changed the name to the same name as above (`29907....tar.gz`). Before installing it using Homebrew, we have to edit what checksum it expectes.

First we get the checksum of the package we created: `shasum -a 256 29907...tar.gz`. And then we edit the homebrew file by typing: `brew edit tbb`. 

Finally! We can now run: `brew install --build-from-source tbb`

And you should now be up and running! 

Hope this helps someone. <br>
// Sebastian Grans :) 

