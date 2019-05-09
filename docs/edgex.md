# EdgeX Foundry
[EdgeX Foundry](https://www.edgexfoundry.org/) is an open-source edge system SW based on micro-services.

MFX-1 SW is based on [Go implementation](https://github.com/edgexfoundry/edgex-go) of EdgeX.

EdgeX system si monitored and controled via EdgeX SW Management Agent service (SMA),
which exposes HTTP REST API for sending commands and fetching logs and metrics.

Additionally, Mainflux Edged agent runs on a same gateway and:
- Connects to MQTT broker in the cloud on one side
- Connects to EdgeX SMA via HTTP on the other side

It serves as a mqtt2http proxy to connect whole gateway to the Mianflux cloud and manipulate EdgeX SMA.

## Install
### Compilation
#### Cross-Compiler
ZMQ demand C++11-compliant cross-compiler, and Debian base `arm-linux-gnueabihf` will not work.
Linaro cross compiler can be used - more info [here](https://www.linaro.org/downloads/).
It can be fetched from [here](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-a/downloads),
you need to look for `AArch32 target with hard float (arm-linux-gnueabihf)`

#### ZMQ
In order to cross-compile ZMQ:

Fetch he last release from [ZMQ GitHub releases](https://github.com/zeromq/libzmq/releases):
```
wget https://github.com/zeromq/libzmq/archive/v4.3.1.tar.gz
```

Follow the instructions [here](http://zeromq.org/build:arm):
```
./autogen.sh
./configure --host=arm-none-linux-gnueabihf CC=arm-linux-gnueabihf-gcc CXX=arm-linux-gnueabihf-g++
```

Produced library will be located in `src/.libs/`

#### EdgeX Foundry
EdgeX Foundry must be cross-compiled for ARM (in order to speed-up the time of native compilation on resource-modest ARM gateway).

A [Makefile](edgex/Makefile) in [edgex](edgex) dir can be used, but we need to define ARM based build:
```
drasko@Marx:~/go/src/github.com/edgexfoundry/edgex-go$ git diff
diff --git a/Makefile b/Makefile
index 928c1650..05127a48 100644
--- a/Makefile
+++ b/Makefile
@@ -14,12 +14,12 @@ BUILD_DIR := build
 ###
 # For ARM cross-compilation add something like:
 #
-# CC = arm-linux-gnueabi-gcc
-# CGO_LDFLAGS = -L/<path_to_libzmq>/src/.libs
-# CGO_CFLAGS = -I/<path_to_libzmq>/include
-# GOOS = linux
-# GOARCH = arm
-# GOARM = 7
+CC = arm-linux-gnueabi-gcc
+CGO_LDFLAGS = -L/home/drasko/edgex/libzmq-4.3.1/src/.libs
+CGO_CFLAGS = -I/home/drasko/edgex/libzmq-4.3.1/include
+GOOS = linux
+GOARCH = arm
+GOARM = 7
 ###
 FLAGS = GOOS=${GOOS} GOARCH=${GOARCH} GOARM=${GOARM} GO111MODULE=on
 GO_FLAGS =  $(FLAGS) CGO_ENABLED=0 
```

##### Configuration
Redis should be used. Example:
```
[Databases]
  [Databases.Primary]
  Host = 'localhost'
  Name = 'coredata'
  Password = ''
  Port = 6379
  Username = ''
  Timeout = 5000
  Type = 'redisdb'
```

This should be put for `core-data`, `core-metadata`, `export-client`, `support-logging` and `support-notifications` microservices. 