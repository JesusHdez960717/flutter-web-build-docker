# Flutter web build docker

An example project on how to build Flutter web using docker (and also deploy it).

This example is deployed [HERE](https://flutter-web-build-docker.root101.dev/)

## Getting Started

To build a Flutter project you need, well, a project, so I used the default one when creating a new
project.

Once we have the project we have to configure the `Dockerfile` that will do the build and deploy.
For this, [Multi-Stage build](https://docs.docker.com/build/building/multi-stage/) is used, which
allows having two images in the same docker, one for the build, and another for the deploy.

### Dockerfile

```dockerfile
#STEP 1: BUILD
# Environemnt to install flutter and build web
FROM debian:latest AS build-env

#install all needed stuff
RUN apt-get update
RUN apt-get install -y curl git unzip

#define variables
ARG FLUTTER_SDK=/usr/local/flutter
ARG APP=/app/

#clone flutter
RUN git clone https://github.com/flutter/flutter.git $FLUTTER_SDK
#change dir to current flutter folder and make a checkout to the specific version
RUN cd $FLUTTER_SDK && git checkout efbf63d9c66b9f6ec30e9ad4611189aa80003d31

#setup the flutter path as an enviromental variable
ENV PATH="$FLUTTER_SDK/bin:$FLUTTER_SDK/bin/cache/dart-sdk/bin:${PATH}"

#Start to run Flutter commands
#doctor to see if all was installes ok
RUN flutter doctor -v

#create folder to copy source code
RUN mkdir $APP
#copy source code to folder
COPY . $APP
#stup new folder as the working directory
WORKDIR $APP

#Run build: 1 - clean, 2 - pub get, 3 - build web
RUN flutter clean
RUN flutter pub get
RUN flutter build web

#once heare the app will be compiled and ready to deploy

#STEP 2: DEPLOY

#use nginx to deploy
FROM nginx:1.25.2-alpine

#copy the info of the builded web app to nginx
COPY --from=build-env /app/build/web /usr/share/nginx/html

#Expose and run nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Use specific Flutter version

If you want to use a specific version of Flutter for the build, you must modify the line:

```dockerfile
RUN cd $FLUTTER_SDK && git checkout efbf63d9c66b9f6ec30e9ad4611189aa80003d31
```

Modifying the git checkout **hash** (`efbf63d9c66b9f6ec30e9ad4611189aa80003d31` in this example) to
the corresponding hash of the desired version.

Here are some of the latest stable versions and their corresponding hashes:

| Version  | Name    |  Hash    |
| -------- | ------- | -------- |
| 3.13.4  | [flutter_releases] Flutter stable 3.13.4 Framework Cherrypicks (#134599)   | 367f9ea16bfae1ca451b9cc27c1366870b187ae2  |
| 3.13.0 | [flutter_releases] Flutter stable 3.13.0 Framework Cherrypicks (#132610)     | efbf63d9c66b9f6ec30e9ad4611189aa80003d31  |
| 3.10.6    | [flutter_releases] Flutter stable 3.10.6 Framework Cherrypicks (#130446)    | f468f3366c26a5092eb964a230ce7892fda8f2f8  |
| 3.10.0    | [flutter_releases] Flutter stable 3.10.0 Framework Cherrypicks (#126304)    | 84a1e904f44f9b0e9c4510138010edcc653163f8  |
| 3.7.12    | [CP] Gradle 8.0 support in flutter 3.7 (#124990)    | 4d9e56e694b656610ab87fcf2efbcd226e0ed8cf  |
| 3.7.10    | [flutter_releases] Update Engine revision (...) for stable release 3.7.0 (#119030)    | b06b8b2710955028a6b562f5aa6fe62941d6febf  |
| 3.3.10    | CP: ci.yaml changes for packaging (#117133)    | 135454af32477f815a7525073027a3ff9eff1bfd  |

#### Obtain version hash

To obtain the hash of a specific version, you must go to
the [flutter repo](https://github.com/flutter/flutter), deploy the **branches combo**, go to the **
tags** tab, **select the tag** from the desired version, and **copy the hash** associated with the
committee of that tag.
, like this:

![flutter-version-tag](flutter-version-tag.png)
