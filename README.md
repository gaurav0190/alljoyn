# alljoyn

## Quick Start

Covers all steps required to install alljoin on Raspbian Lite 4.1.

### Dependencies

The first step is installing the dependencies.

```bash
pi@raspberrypi:~$ sudo apt-get update
pi@raspberrypi:~$ sudo apt-get install maven
pi@raspberrypi:~$ sudo apt-get install scons
pi@raspberrypi:~$ sudo apt-get install git
pi@raspberrypi:~$ sudo apt-get install curl
pi@raspberrypi:~$ sudo apt-get install openssl
pi@raspberrypi:~$ sudo apt-get install libssl-dev
pi@raspberrypi:~$ sudo apt-get install libjson0
pi@raspberrypi:~$ sudo apt-get install libjson0-dev
pi@raspberrypi:~$ sudo apt-get install libcap-dev
```

### Setting up Compiler

Next step is to soft link the compiler to a location expected by the build tools.

```bash
pi@raspberrypi:~$ sudo ln -s /usr/bin/g++ /usr/bin/arm-angstrom-linux-gnueabi-g++
pi@raspberrypi:~$ sudo ln -s /usr/bin/gcc /usr/bin/arm-angstrom-linux-gnueabi-gcc
```

### Obtaining source code

Next step is to obtain the source code

```bash
pi@raspberrypi:~$ mkdir -p ~/Code
pi@raspberrypi:~$ cd ~/Code
pi@raspberrypi:~/Code$ git config --global user.name "YOUR_NAME_HERE"
pi@raspberrypi:~/Code$ git config --global user.email "YOUR_EMAIL_HERE"
pi@raspberrypi:~/Code$ git clone https://github.com/kdavis-mozilla/alljoyn.git
pi@raspberrypi:~/Code$ cd alljoyn
pi@raspberrypi:~/Code/alljoyn$ git pull && git submodule init && git submodule update && git submodule status
pi@raspberrypi:~/Code/alljoyn$ export AJ_ROOT=$(pwd)
```

### Obtaining XULRunner-SDK

Next step is obtaining the XULRunner-SDK, the headers are required for compilation of the JavaScript bindings.

```bash
pi@raspberrypi:~/Code/alljoyn$ cd ~/Code
pi@raspberrypi:~/Code$ wget http://ftp.mozilla.org/pub/mozilla.org/xulrunner/releases/1.9.2/sdk/xulrunner-1.9.2.en-US.mac-i386.sdk.tar.bz2
pi@raspberrypi:~/Code$ tar xfvj xulrunner-1.9.2.en-US.mac-i386.sdk.tar.bz2
pi@raspberrypi:~/Code$ rm xulrunner-1.9.2.en-US.mac-i386.sdk.tar.bz2
pi@raspberrypi:~/Code$ export GECKO_BASE=~/Code/xulrunner-sdk
```

### Compiling

Compiling is separated into two steps, compiling the core and compiling services.

#### Core

Compiling the core and its C++, C, and JavaScript bindings takes some time.

```bash
pi@raspberrypi:~/Code$ cd ~/Code/alljoyn/core/alljoyn
pi@raspberrypi:~/Code/alljoyn/core/alljoyn$ scons OS=linux CPU=arm WS=off OE_BASE=/usr BR=on BINDINGS=cpp,c,js CROSS_COMPILE=/usr/bin/arm-linux-gnueabihf-
```

Next the compilation results must be linked into the expected location

```bash
pi@raspberrypi:~/Code/alljoyn/core/alljoyn$ sudo ln -sf ~/Code/alljoyn/core/alljoyn/build/linux/arm/debug/dist/cpp/lib/liballjoyn.so /lib/arm-linux-gnueabihf/liballjoyn.so
```

#### Services

Next one compiles the services about, notification, controlpanel, config, onboarding, sample_apps, and audio. This will also take some time.

```bash
pi@raspberrypi:~/Code/alljoyn/core/alljoyn$ scons OS=linux CPU=arm WS=off SERVICES=about,notification,controlpanel,config,onboarding,sample_apps,audio BINDINGS=core,cpp,js OE_BASE=/usr CROSS_COMPILE=/usr/bin/arm-linux-gnueabihf- 
```

#### AC Server Sample

Next one compiles the AC server sample.

```bash
pi@raspberrypi:~/Code/alljoyn/core/alljoyn$ cd ~/Code/alljoyn/services/base/sample_apps$
pi@raspberrypi:~/Code/alljoyn/services/base/sample_apps$ export AJ_ROOT=$(pwd)
pi@raspberrypi:~/Code/alljoyn/services/base/sample_apps$ scons OS=linux CPU=arm BINDINGS=cpp WS=off ALL=1 OE_BASE=/usr CROSS_COMPILE=/usr/bin/arm-linux-gnueabihf-
```

#### Ducktape

Next one has to download and install Ducktape, an embeddable Javascript engine with a focus on portability and compact footprint.

```bash
pi@raspberrypi:~/Code/alljoyn/core/alljoyn$ cd ~/Code
pi@raspberrypi:~/Code$ wget http://www.duktape.org/duktape-1.3.1.tar.xz
pi@raspberrypi:~/Code$ tar xvfJ duktape-1.3.1.tar.xz
pi@raspberrypi:~/Code$ rm duktape-1.3.1.tar.xz
pi@raspberrypi:~/Code$ cd duktape-1.3.1
pi@raspberrypi:~/Code/duktape-1.3.1$ make -f Makefile.cmdline
pi@raspberrypi:~/Code/duktape-1.3.1$ export DUKTAPE_DIST=$(pwd)
```

#### Alljoyn Thin Core

Next one has to compile the alljoan thin core, used to embedded devices with small footprints.

```bash
pi@raspberrypi:~/Code/duktape-1.3.1$ cd ~/Code/alljoyn
pi@raspberrypi:~/Code/alljoyn$ export AJ_ROOT=$(pwd)
pi@raspberrypi:~/Code/alljoyn$ cd core/ajtcl
pi@raspberrypi:~/Code/alljoyn/core/ajtcl$ scons OS=linux CPU=arm WS=off OE_BASE=/usr CROSS_COMPILE=/usr/bin/arm-linux-gnueabihf-
```

#### Base TCL code

Next one has to compile the base TCL code.

```bash
pi@raspberrypi:~/Code/alljoyn/core/ajtcl$ cd ~/Code/alljoyn/services/base_tcl
pi@raspberrypi:~/Code/alljoyn/services/base_tcl$ scons OS=linux CPU=arm WS=off OE_BASE=/usr CROSS_COMPILE=/usr/bin/arm-linux-gnueabihf-
```

#### Finally the alljoyn.js

```bash
pi@raspberrypi:~/Code/alljoyn/services/base_tcl$ cd ~/Code/alljoyn/core/alljoyn-js
pi@raspberrypi:~/Code/alljoyn/core/alljoyn-js$ export ALLJOYN_DIST=~/Code/alljoyn/core/alljoyn/build/linux/arm/debug/dist
pi@raspberrypi:~/Code/alljoyn/core/alljoyn-js$ scons OS=linux CPU=arm OE_BASE=/usr CROSS_COMPILE=/usr/bin/arm-linux-gnueabihf-
```

