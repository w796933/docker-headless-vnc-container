docker run -d -p 5901:5901 -p 6901:6901 --user $(id -u):$(id -g) -e VNC_PW=1111 consol/centos-xfce-vnc  直接这个支持中文网页

然后在容器运行  yum install icedtea-web    

这个是打开  jnlp  IPMI的   远程连接文件   

用火狐浏览器打开  打开程序选择/usr/bin/javaws

参考   http://blog.51cto.com/harryzhang/1705654






# Docker container images with "headless" VNC session

The repository contains a collection of Docker images with headless VNC environments.

Each Docker image is installed with the following components:

* Desktop environment [**Xfce4**](http://www.xfce.org) or [**IceWM**](http://www.icewm.org/)
* VNC-Server (default VNC port `5901`)
* [**noVNC**](https://github.com/kanaka/noVNC) - HTML5 VNC client (default http port `6901`)
* Browsers:
  * Mozilla Firefox
  * Chromium

![Docker VNC Desktop access via HTML page](.pics/vnc_container_view.png)

## Current provided OS & UI sessions:
* `consol/centos-xfce-vnc`: __Centos7 with `Xfce4` UI session__ 

  [![](https://images.microbadger.com/badges/version/consol/centos-xfce-vnc.svg)](https://hub.docker.com/r/consol/centos-xfce-vnc/) [![](https://images.microbadger.com/badges/image/consol/centos-xfce-vnc.svg)](http://microbadger.com/images/consol/centos-xfce-vnc)

* `consol/ubuntu-xfce-vnc`: __Ubuntu with `Xfce4` UI session__

  [![](https://images.microbadger.com/badges/version/consol/ubuntu-xfce-vnc.svg)](https://hub.docker.com/r/consol/ubuntu-xfce-vnc/) [![](https://images.microbadger.com/badges/image/consol/ubuntu-xfce-vnc.svg)](http://microbadger.com/images/consol/ubuntu-xfce-vnc)

* `consol/centos-icewm-vnc`: __Centos7 with `IceWM` UI session__ 

  [![](https://images.microbadger.com/badges/version/consol/centos-icewm-vnc.svg)](https://hub.docker.com/r/consol/centos-icewm-vnc/) [![](https://images.microbadger.com/badges/image/consol/centos-icewm-vnc.svg)](http://microbadger.com/images/consol/centos-icewm-vnc)

* `consol/ubuntu-icewm-vnc`: __Ubuntu with `IceWM` UI session__

  [![](https://images.microbadger.com/badges/version/consol/ubuntu-icewm-vnc.svg)](https://hub.docker.com/r/consol/ubuntu-icewm-vnc/) [![](https://images.microbadger.com/badges/image/consol/ubuntu-icewm-vnc.svg)](http://microbadger.com/images/consol/ubuntu-icewm-vnc)

## OpenShift / Kubernetes

It's also possible to run the images in container orchestration platforms like [Kubernetes](https://kubernetes.io) or [OpenShift](https://openshift.io/). For more information how to deploy containers in the cluster, take a look at [OpenShift usage of "headless" VNC Docker images](/openshift/README.md) 

## Latest Changes
See the [**changelog.md**](./changelog.md).

## Usage
The usage is for all provide images **similar**, for instance see following the usage of the `consol/centos-xfce-vnc` image:

Run command with mapping to local port `5901` (vnc protocol) and `6901` (vnc web access):

    docker run -d -p 5901:5901 -p 6901:6901 consol/centos-xfce-vnc
  
Change the default user and group within a container to your own with adding `--user $(id -u):$(id -g)`:

    docker run -d -p 5901:5901 -p 6901:6901 --user $(id -u):$(id -g) consol/centos-xfce-vnc

If you want to get into the container use interactive mode `-it` and `bash`     

    docker run -it -p 5901:5901 -p 6901:6901 consol/centos-xfce-vnc bash

Build an image from scratch:

    docker build -t consol/centos-xfce-vnc centos-xfce-vnc

=> connect via __VNC viewer `localhost:5901`__, default password: `vncpassword`

=> connect via __noVNC HTML5 client__: [http://localhost:6901/?password=vncpassword]()


## Hints

### 1) Extend a Image with your own software
Since `1.1.0` all images run as non-root user per default, so that means, if you want to extend the image and install software, you have to switch in the `Dockerfile` back to the `root` user:

```bash
## Custom Dockerfile
FROM consol/centos-xfce-vnc:1.1.0
MAINTAINER Tobias Schneck "tobias.schneck@consol.de"
ENV REFRESHED_AT 2017-04-10

## Install a gedit
USER 0
RUN yum install -y gedit \
    && yum clean all
## switch back to default user
USER 1984
```

### 2) Change User of running Sakuli Container

Per default, since version `1.1.0` all container processes will executed with user id `1984`. You can change the user id like follow: 

#### 2.1) Using root (user id `0`)
Add the `--user` flag to your docker run command:

    docker run -it --user 0 -p 6911:6901 consol/centos-xfce-vnc

#### 2.2) Using user and group id of host system
Add the `--user` flag to your docker run command:

    docker run -it -p 6911:6901 --user $(id -u):$(id -g) consol/centos-xfce-vnc

### 3) Override VNC environment variables
The following VNC environment variables can be overwritten at the `docker run` phase to customize your desktop environment inside the container:
* `VNC_COL_DEPTH`, default: `24`
* `VNC_RESOLUTION`, default: `1280x1024`
* `VNC_PW`, default: `my-pw`

#### 3.1) Example: Override the VNC password
Simply overwrite the value of the environment variable `VNC_PW`. For example in
the docker run command:

    docker run -it -p 5901:5901 -p 6901:6901 -e VNC_PW=my-pw consol/centos-xfce-vnc

#### 3.2) Example: Override the VNC resolution
Simply overwrite the value of the environment variable `VNC_RESOLUTION`. For example in
the docker run command:

    docker run -it -p 5901:5901 -p 6901:6901 -e VNC_RESOLUTION=800x600 consol/centos-xfce-vnc
    
### 4) View only VNC
Since version `1.2.0` it's possible to prevent unwanted control over VNC. Therefore you can set the environment variable `VNC_VIEW_ONLY=true`. If set the docker startup script will create a random cryptic password for the control connection and use the value of `VNC_PW` for the view only connection over the VNC connection.

     docker run -it -p 5901:5901 -p 6901:6901 -e VNC_VIEW_ONLY=true consol/centos-xfce-vnc

## Contact
For questions, professional support or maybe some hints, feel free to contact us via **[testautomatisierung@consol.de](mailto:testautomatisierung@consol.de)** or open an [issue](https://github.com/ConSol/docker-headless-vnc-container/issues/new).

The guys behind:

**ConSol Software GmbH** <br/>
*Franziskanerstr. 38, D-81669 München* <br/>
*Tel. +49-89-45841-100, Fax +49-89-45841-111*<br/>
*Homepage: http://www.consol.de E-Mail: [info@consol.de](info@consol.de)*
