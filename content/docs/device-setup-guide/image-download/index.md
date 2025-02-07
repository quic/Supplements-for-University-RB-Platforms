+++
title = 'Downloading the Image'
date = 2024-09-24T11:15:25-07:00
draft = false
weight = 2
+++
To obtain the image that will be flashed onto the RB5 device, a tool called `sdkmanager` is used. This guide will go over installing `sdkmanager` in a docker container. If you've never used docker, then it's worth reading some of the [introductory documentation](https://docs.docker.com/get-started/). You do not need to be a docker expert to follow this guide, however.

## Install docker on the host system
{{< tabs items="Using docker-install,Manually" >}}
  {{< tab >}}
  ```sh
# from https://github.com/docker/docker-install
  curl -fsSL https://get.docker.com -o get-docker.sh
  sh get-docker.sh
  ```
  {{< /tab >}}
  {{< tab >}}
  The full procedure can be found [here](https://docs.docker.com/engine/install/ubuntu/).
  {{< /tab >}}
{{< /tabs >}}

### Add user to docker group (optional)
Adding your system user to the docker group makes it easier to run docker commands, but there are some security implications. The details are readily found online, so instead we'll just give the steps.
```sh
# from https://docs.docker.com/engine/install/linux-postinstall/
sudo usermod -aG docker $USER
# then log out and back in
```

## Set up and run sdkmanager
1. Create a new working directory
```sh
mkdir workdir
```
```sh
cd workdir
```
2. Download docker resources into the new directory
* [Dockerfile](Dockerfile)
* [docker-compose.yaml](docker-compose.yaml)
3. Create a directory for the build that sdkmanager will produce
```sh
mkdir build
```
4. Kick off the project containers
```sh
docker compose up -d --build
```
5. In another terminal from the same directory, attach to the `download_build` container
```sh
docker compose attach download_build
```
6. Start sdkmanager from within the container
```sh
sdkmanager
# You will mostly just follow the prompts inside sdkmanager from here
# Enter your Thundercomm credentials
# Absolute target directory for the build -> /build
# Product -> RB5
# Platform -> LU
# LU version -> LU2.0
# build -> The latest one by the date in the filename (20240603 at time of writing)
# You will be left with a prompt "> "
# Type 1 and press enter
# Get some coffee. This will take a long time (40 minutes or more).
# Type q to exit sdkmanager once it finishes
```
7. Exit the container
```sh
exit
```
8. From the original terminal, shut down all containers
```sh
docker compose down
```
At this point, the build files will be in a zip file in the `build` directory. At the time of writing, the file is called `QRB5165.LU2.0_20240603.2127_debug.zip`
