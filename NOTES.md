# Uh oh, I don't know Go D:  

I'm not much of a linux guy, but I've tried running the minikube and such on Ubuntu running on WSL 2.  
One issue is that it's configured improperly, my default user is root, which minikube discourages, for good reason.  
  
I'm more familiar with DroneCI, and have my reservations about Github Actions. But I've chosen to use Github actions because of time constraints.  

I've also had a bad experience with minikube, we've had virtual machines that hardly ever worked. While docker-desktop seemed to work just fine.  
  
### Dockerfile considerations

Normally you'd use alpine, but because of needed ability to debug, a full image was chosen.  
In a prod-like setup, reducing privileges aswell as installed applications/tools is recommended, hence, alpine.  

The dockerfile is borrowed from https://hub.docker.com/_/golang  
  
If memory serves me correct, we used a kaniko-based approach at Nuuday for building images, the benefit was that the agents were not required to run as root.  

  
### Running the application  
The readme says "Build the image locally and verify that the server works." but simply doing 'docker run {image}' fails, as redis and the file secrets/users.json are required, possibly more.  
1. volume mount required. Fairly easily done.
2. Env variables loading, same as above.  
  
In a real scenario, I would have used docker-compose with all of it preconfigured.  
  
Not sure what to do about the .env file, i try to mount it, but it still can't find it.  
Trying to debug it using entrypoint overwrite and see the default env location for go.  
Which is done using this command:  
```docker run -it --mount type=bind,source="$(pwd)"/secrets/users.json,target=/secrets/users.json,readonly --mount type=bind,source="$(pwd)"/.env,target=/root/.config/go/.env --entrypoint /bin/bash```

result: GOENV="/root/.config/go/env"  
Despite putting the .env file into /root/.config/go/env/.env it still doesn't load. 

On further investigation, it does seem like the go env picks up the file:  
```docker run -it --mount type=bind,source="$(pwd)"/secrets/users.json,target=/secrets/users.json,readonly --mount type=bind,source="$(pwd)"/.env,target=/root/.config/go/env/.env --entrypoint /bin/sh 4b3734dcaf46```
```app```
```$go env
root@DESKTOP-LUSF7V5:/mnt/c/Users/dumbl/IdeaProjects/uniwise-devops-assignment# docker run -it --mount type=bind,source="$(pwd)"/secrets/users.json,target=/secrets/users.json,readonly --mount type=bind,source="$(pwd)"/.env,target=/root/.config/go/env/.env --entrypoint /bin/sh 4b373
4dcaf46
/usr/src/app # cat /root/.config/go/env/.env
PORT=4444
REDIS_URL=redis://localhost:6379/usr/src/app # go env
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/root/.cache/go-build"
GOENV="/root/.config/go/env"
GOEXE=""
GOEXPERIMENT=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOINSECURE=""
GOMODCACHE="/go/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GOVCS=""
GOVERSION="go1.19.4"
GCCGO="gccgo"
GOAMD64="v1"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD="/usr/src/app/go.mod"
GOWORK=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -Wl,--no-gc-sections -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build318482767=/tmp/go-build -gno-record-gcc-switches"
```
So it *does* load, but complains about it.
![Lesson learned](By_all_accounts_it_doesn't_make_sense_banner.jpg)

This is the part where I flip my desk. (╯‵□′)╯︵┻━┻
Turns out the godotenv is *current-path* aware, which explains why it could find the other file, but not the .env.

The winning script
```shell
docker run -it \
--mount type=bind,source="$(pwd)"/secrets/users.json,target=/secrets/users.json,readonly \
--mount type=bind,source="$(pwd)"/.env,target=/usr/src/app/.env,readonly \
uniwise
 ```
