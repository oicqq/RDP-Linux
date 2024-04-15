# RDP-Linux

Rdesktop - Containers containing full desktop environments in many popular flavors for Alpine, Ubuntu, Arch, and Fedora accessible via RDP.
来源：https://github.com/linuxserver/docker-rdesktop?tab=readme-ov-file

Supported Architectures
We utilise the docker manifest for multi-platform awareness. More information is available from docker here and our announcement here.

Simply pulling lscr.io/linuxserver/rdesktop:latest should retrieve the correct image for your arch, but you can also pull specific arch images via tags.

The architectures supported by this image are:

Architecture	Available	Tag
x86-64	✅	amd64-<version tag>
arm64	✅	arm64v8-<version tag>
armhf	❌	
Version Tags
This image provides various versions that are available via tags. Please read the descriptions carefully and exercise caution when using unstable or development tags.

Tag	Available	Description
latest	✅	XFCE Alpine
ubuntu-xfce	✅	XFCE Ubuntu
fedora-xfce	✅	XFCE Fedora
arch-xfce	✅	XFCE Arch
alpine-kde	✅	KDE Alpine
ubuntu-kde	✅	KDE Ubuntu
fedora-kde	✅	KDE Fedora
arch-kde	✅	KDE Arch
alpine-mate	✅	MATE Alpine
ubuntu-mate	✅	MATE Ubuntu
fedora-mate	✅	MATE Fedora
arch-mate	✅	MATE Arch
alpine-i3	✅	i3 Alpine
ubuntu-i3	✅	i3 Ubuntu
fedora-i3	✅	i3 Fedora
arch-i3	✅	i3 Arch
alpine-openbox	✅	Openbox Alpine
ubuntu-openbox	✅	Openbox Ubuntu
fedora-openbox	✅	Openbox Fedora
arch-openbox	✅	Openbox Arch
alpine-icewm	✅	IceWM Alpine
ubuntu-icewm	✅	IceWM Ubuntu
fedora-icewm	✅	IceWM Fedora
arch-icewm	✅	IceWM Arch
Application Setup
The Default USERNAME and PASSWORD is: abc/abc

Unlike our other containers these Desktops are not designed to be upgraded by Docker, you will keep your home directoy but anything you installed system level will be lost if you upgrade an existing container. To keep packages up to date instead use Ubuntu's own apt, Alpine's apk, Fedora's dnf, or Arch's pacman program

You will need a Remote Desktop client to access this container Wikipedia List, by default it listens on 3389, but you can change that port to whatever you wish on the host side IE 3390:3389. The first thing you should do when you login to the container is to change the abc users password by issuing the passwd command.

Modern GUI desktop apps (including some flavors terminals) have issues with the latest Docker and syscall compatibility, you can use Docker with the --security-opt seccomp=unconfined setting to allow these syscalls or try podman as they have updated their codebase to support them

If you ever lose your password you can always reset it by execing into the container as root:

docker exec -it rdesktop passwd abc
By default we perform all logic for the abc user and we reccomend using that user only in the container, but new users can be added as long as there is a startwm.sh executable script in their home directory. All of these containers are configured with passwordless sudo, we make no efforts to secure or harden these containers and we do not reccomend ever publishing their ports to the public Internet.

Hardware Acceleration (Ubuntu Container Only)
Many desktop application will need access to a GPU to function properly and even some Desktop Environments have compisitor effects that will not function without a GPU. This is not a hard requirement and all base images will function without a video device mounted into the container.

Intel/ATI/AMD
To leverage hardware acceleration you will need to mount /dev/dri video device inside of the conainer.

--device=/dev/dri:/dev/dri
We will automatically ensure the abc user inside of the container has the proper permissions to access this device.

Nvidia
Hardware acceleration users for Nvidia will need to install the container runtime provided by Nvidia on their host, instructions can be found here: https://github.com/NVIDIA/nvidia-docker

We automatically add the necessary environment variable that will utilise all the features available on a GPU on the host. Once nvidia-docker is installed on your host you will need to re/create the docker container with the nvidia container runtime --runtime=nvidia and add an environment variable -e NVIDIA_VISIBLE_DEVICES=all (can also be set to a specific gpu's UUID, this can be discovered by running nvidia-smi --query-gpu=gpu_name,gpu_uuid --format=csv ). NVIDIA automatically mounts the GPU and drivers from your host into the container.

Arm Devices
Best effort is made to install tools to allow mounting in /dev/dri on Arm devices. In most cases if /dev/dri exists on the host it should just work. If running a Raspberry Pi 4 be sure to enable dtoverlay=vc4-fkms-v3d in your usercfg.txt.

Usage
To help you get started creating a container from this image you can either use docker-compose or the docker cli.

docker-compose (recommended, click here for more info)
---
services:
  rdesktop:
    image: lscr.io/linuxserver/rdesktop:latest
    container_name: rdesktop
    security_opt:
      - seccomp:unconfined #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock #optional
      - /path/to/data:/config #optional
    ports:
      - 3389:3389
    devices:
      - /dev/dri:/dev/dri #optional
    shm_size: "1gb" #optional
    restart: unless-stopped
docker cli (click here for more info)
docker run -d \
  --name=rdesktop \
  --security-opt seccomp=unconfined `#optional` \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -p 3389:3389 \
  -v /var/run/docker.sock:/var/run/docker.sock `#optional` \
  -v /path/to/data:/config `#optional` \
  --device /dev/dri:/dev/dri `#optional` \
  --shm-size="1gb" `#optional` \
  --restart unless-stopped \
  lscr.io/linuxserver/rdesktop:latest
Parameters
Containers are configured using parameters passed at runtime (such as those above). These parameters are separated by a colon and indicate <external>:<internal> respectively. For example, -p 8080:80 would expose port 80 from inside the container to be accessible from the host's IP on port 8080 outside the container.

Parameter	Function
-p 3389	RDP access port
-e PUID=1000	for UserID - see below for explanation
-e PGID=1000	for GroupID - see below for explanation
-e TZ=Etc/UTC	specify a timezone to use, see this list.
-v /var/run/docker.sock	Docker Socket on the system, if you want to use Docker in the container
-v /config	abc users home directory
--device /dev/dri	Add this for GL support (Linux hosts only)
--shm-size=	We set this to 1 gig to prevent modern web browsers from crashing
--security-opt seccomp=unconfined	For Docker Engine only, many modern gui apps need this to function as syscalls are unkown to Docker
