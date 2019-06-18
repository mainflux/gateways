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
make
```

Produced library will be located in `src/.libs/`

##### Gateway
You need the same ZMQ on the gateway as well, as CGO will link it dynamically.
Static linking would demand linking of stdlib and then it demands toolchain and so on.

So easier way is just to compile and install same version of ZMQ on gateway:

```
sudo apt install -y libtool pkg-config build-essential autoconf automake uuid-dev
./autogen
./configure
make
sudo make install
```



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

This should be put for `core-data`, `core-metadata`, `export-client`, `support-scheduler` and `support-notifications` microservices.

`support-logging` does not yet have Redis enablement, so `file` setting can be used:

```
[Writable]
Persistence = 'file'
LogLevel = 'INFO'
```

## Docker
In order to build in Docker, `libzmq` needs to be cross-compiled in the builder container:
- If Debian builder is used (`golang:stretch`), then Linaro toolchain can be used
- If Alpine builder is used (`golang:alpine3.9`) then a [musl](https://www.musl-libc.org/) toolchain must be used,
and this one must be produced. Best is to use [musl-cross-make](https://github.com/richfelker/musl-cross-make).

Production container must be of a type `arm32v7/`:
- If Debian is used for builder, then production container must also be
Debian (because of `glibc`, and not `musl`), and reccomended container would be `arm32v7/stretch-slim`
- If Alpine is used for builder, then production container can be `arm32v7/alpine`

### Toolchain
In order to produce `arm-linux-musleabihf` toolchain needed for Alpine cross-compilation:
```
git clone https://github.com/richfelker/musl-cross-make
cd musl-cross-make
cp config.mak.dist config.mak
```

Edit `config.mak` and uncomment `arm-linux-musleabihf` target:
```
# TARGET = i486-linux-musl
# TARGET = x86_64-linux-musl
# TARGET = arm-linux-musleabi
TARGET = arm-linux-musleabihf
# TARGET = sh2eb-linux-muslfdpic
```

Then:
```
make -j 16
```

This will produce `musl-cross-make/output` directory which you should copy to `docker` dir of EdgeX:
```
cp musl-cross-make/output $GOPATH/src/github.com/edgexfoundry/edgex-go/docker/toolchain
```

This is where Dockerfile will look for it.

### qemu-arm-static
Production container must install `libzmq` (in the case of Debian this is `libzmq5`),
and since we are in ARM container executables like `apk` will not work.
For that reason we need to add `qemu-arm-static`, like explained [here](https://ownyourbits.com/2018/06/27/running-and-building-arm-docker-containers-in-x86/) or
[here](https://www.balena.io/blog/building-arm-containers-on-any-x86-machine-even-dockerhub/)

> Alpine images take ~24MB while Strech-Slim images might take ~75MB

### Transfering Docker image
Docker images must be transfered to gateway.

On host:
```
docker save -o export-distro.img edgex/export-distro
scp export-distro.img debian@sr-imx6.local:~/
```

On gateway:
```
docker load -i export-distro.img
docker run edgex/export-distro:1.0.0-dev
```
