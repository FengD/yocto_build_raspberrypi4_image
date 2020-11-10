# Yocto Build Raspberrypi4 Image

- [Yocto Build Raspberrypi4 Image](#yocto-build-raspberrypi4-image)
  * [1. Yocto](#1-yocto)
    + [1.1. What is yocto?](#11-what-is-yocto-)
    + [1.2. How yocto works?](#12-how-yocto-works-)
      - [1.2.1. Introduction](#121-introduction)
      - [1.2.2. What's special of this project?](#122-what-s-special-of-this-project-)
      - [1.2.3. Prepare yocto project](#123-prepare-yocto-project)
  * [2. Raspberrypi4](#2-raspberrypi4)
    + [2.1. Build Raspberrypi4 image [7]](#21-build-raspberrypi4-image--7-)
      - [2.1.1. Clone all the meta-layers](#211-clone-all-the-meta-layers)
      - [2.1.2. Customize the configuration files](#212-customize-the-configuration-files)
      - [2.1.3. Edit local.conf](#213-edit-localconf)
      - [2.1.4. Build](#214-build)
      - [2.1.5. Deploy](#215-deploy)
    + [2.2. Build Your own meta-layer](#22-build-your-own-meta-layer)
    + [2.3. Build third-party library](#23-build-third-party-library)
    + [2.4. Add other layers and recipes](#24-add-other-layers-and-recipes)
      - [2.4.1. Add docker](#241-add-docker)
    + [2.5. Cross build application](#25-cross-build-application)
  * [3. Dependencies](#3-dependencies)
  * [4. Tips](#4-tips)
    + [Error List[10]](#error-list-10-)
  * [5. References](#5-references)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## 1. Yocto

### 1.1. What is yocto?

The Yocto Project (YP) is an open source collaboration project that helps developers create custom Linux-based systems regardless of the hardware architecture.

The project provides a flexible set of tools and a space where embedded developers worldwide can share technologies, software stacks, configurations, and best practices that can be used to create tailored Linux images for embedded and IOT devices, or anywhere a customized Linux OS is needed. [1]

### 1.2. How yocto works?
#### 1.2.1. Introduction
The Yocto Project is a collection of tools and meta-data (defined in a bit) that allows a developer to build their own custom distribution of Linux for their embedded platform. This could be a developer at a semi-conductor company, who wishes to develop board support for one of their hardware platforms, or it could be an independent developer writing a complete software stack for a product they are making. It could also be a group of engineers developing a distribution for use in multiple devices or products -- such as an embedded Linux distribution company, or the "systems" team at a company that produces multiple embedded Linux products.[14]

The main parts of the Yocto Project are the build system, the package meta-data, and the developer tools. The build system uses a tool called "bitbake" to process the meta-data and produce a complete Linux distribution. By design, the build system produces not just the software that will run on the target, but also the development tools used to build that software. It basically starts completely from scratch, building all the tools needed to construct the software, and then using those to build the kernel, libraries, and programs that comprise a Linux distribution. Finally, it prepares the resulting software by placing it into appropriate bundles (including packages, images, or both) for deployment to the target device and in preparation for application development and debugging. The Yocto Project also includes various additional tools used to develop embedded Linux or applications on top of it. This includes things such as emulators, IDEs and host/target (cross) agents and debug tools.[14]

#### 1.2.2. What's special of this project?
To use yocto project to build your customized Linux os. You need to have an ubuntu 14.04 or 16.04 `(yocto only support for these two versions)` system and install all the dependencies in it. For example,

```sh
# Install dependencies
sudo apt install wget git-core unzip make gcc g++ build-essential subversion sed autoconf automake texi2html texinfo coreutils diffstat python-pysqlite2 docbook-utils libsdl1.2-dev libxml-parser-perl libgl1-mesa-dev libglu1-mesa-dev xsltproc desktop-file-utils chrpath groff libtool xterm gawk fop patch makeinfo git bzip2 cpio

# Create a no root user
useradd <username>

# Change Lang
export LANGUAGE=en_US.UTF-8

#etc.
```

And to simplify the steps above, I created a docker image based on ubuntu16.04 image, which provides you a good enviroment.
```sh
# Fetch the image
docker pull dingfengffffff/yocto:latest
# Create the container
docker run -it -v <local_yocto_project_path>:/yocto dingfengffffff/yocto:latest /bin/bash
```

#### 1.2.3. Prepare yocto project
With the docker command above you could go into the docker container.
```sh
# Go to the yocto fold
cd /yocto
# I use the dunfell branch, so create a folder
mkdir dunfell & cd dunfell
# Fetch the poky-dunfell
git clone -b dunfell git://git.yoctoproject.org/poky.git poky-dunfell
# Tips: all the 3rd-party meta-layers should on the same branch to void build errors.
# user ding is created for you to build
su ding
# Change the envrionment
source /yocto/dunfell/poky-dunfell/oe-init-build-env build
```
After the stemps below you will create a build folder and get into the folder automatically. At this time, the yocto environment is ready for you to use.

## 2. Raspberrypi4
There are several models of Raspberry Pi, and for most people Raspberry Pi 4 Model B is the one to choose. Raspberry Pi 4 Model B is the newest, fastest, and easiest to use.[16]

Raspberry Pi 4 comes with 2GB, 4GB, or 8GB of RAM. For most educational purposes and hobbyist projects, and for use as a desktop computer, 2GB is enough.[16]

Raspberry Pi Zero, Raspberry Pi Zero W, and Raspberry Pi Zero WH are smaller and require less power, so they’re useful for portable projects such as robots. It’s generally easier to start a project with Raspberry Pi 4, and to move to Raspberry Pi Zero when you have a working prototype that a smaller Raspberry Pi would be useful for.[16]

### 2.1. Build Raspberrypi4 image [7]
#### 2.1.1. Clone all the meta-layers
With the step [1.2.3](#123-prepare-yocto-project), you had a complete yocto environment. So let's follow that step.
```sh
# create a folder to store all the 3rd-party layers
mkdir -p /yocto/dunfell/layers/ & cd /yocto/dunfell/layers/
# Fetch all the meta needed
git clone -b dunfell git://git.openembedded.org/meta-openembedded
git clone -b dunfell https://github.com/meta-qt5/meta-qt5
git clone -b dunfell git://git.yoctoproject.org/meta-raspberrypi
git clone -b dunfell git://git.yoctoproject.org/meta-security.git
git clone -b dunfell https://github.com/jumpnow/meta-jumpnow
git clone -b dunfell git://github.com/jumpnow/meta-rpi64
```
#### 2.1.2. Customize the configuration files
```sh
cp meta-rpi64/conf/local.conf.sample build/conf/local.conf
cp meta-rpi64/conf/bblayers.conf.sample build/conf/bblayers.conf

# or `bitbake-layers add-layer <layer need>`
```

#### 2.1.3. Edit local.conf
The variables you may want to customize are the following:
> MACHINE
>
> TMPDIR
>
> DL_DIR
>
> SSTATE_DIR

* MACHINE
  > The MACHINE variable is used to determine the target architecture and various compiler tuning flags.

  > See the conf files under `meta-raspberrypi/conf/machine` for details.

  > The only choice for MACHINE that I have tested with 64-bit builds is raspberrypi4-64.

* TMPDIR
  > This is where temporary build files and the final build binaries will end up. Expect to use around 20GB.

  > The default location is under the build directory, in this example ~/rpi64/build/tmp.

  > If you specify an alternate location as I do in the example conf file make sure the directory is writable by the user running the build.

* DL_DIR
  > This is where the downloaded source files will be stored. You can share this among configurations and builds so I always create a general location for this outside the project directory. Make sure the build user has write permission to the directory you decide on.

  > The default location is in the build directory, ~/rpi64/build/sources.

* SSTATE_DIR
  > This is another Yocto build directory that can get pretty big, greater then 4GB. I often put this somewhere else other then my home directory as well.

  > The default location is in the build directory, ~/rpi64/build/sstate-cache.

* KERNEL VERSION
  > The default is 5.4.

  > Comment this line

  ```
  PREFERRED_VERSION_linux-raspberrypi = "5.4.%" and uncomment this one
  ```

  ```
  # PREFERRED_VERSION_linux-raspberrypi = "4.19.%" to use a 4.19 kernel.
  ```

* ROOT PASSWORD
 > There is only one login user by default, root. The default password is set to jumpnowtek by these two lines in the local.conf file
```
INHERIT += "extrausers"
EXTRA_USERS_PARAMS = "usermod -P jumpnowtek root; "
```
> These two lines force a password change on first login
```
INHERIT += "chageusers"
CHAGE_USERS_PARAMS = "chage -d0 root; "
```
> You can comment them out if you do not want that behavior. If you want no password at all (development only hopefully), comment those four lines and uncomment this line

  ```
  EXTRA_IMAGE_FEATURES = "debug-tweaks"
  #INHERIT += "extrausers"
  #EXTRA_USERS_PARAMS = "usermod -P jumpnowtek root; "
  #INHERIT += "chageusers"
  #CHAGE_USERS_PARAMS = "chage -d0 root; "
  ```
> You can always add or change the password once logged in.

#### 2.1.4. Build
```sh
bitbake console-image
```

#### 2.1.5. Deploy
If [2.1.4](#215-deploy) finished with no error. You need to prepare a SD card.
```sh
# check sdcard
lsblk
# Use the script in the meta-rpi64/scripts to format
sudo ./mk2parts.sh [dev]
# Use the script in the meta-rpi64/script to cp root and image
./copy_boot.sh [dev]
./copy_rootfs.sh [dev] console
```

### 2.2. Build Your own meta-layer
You cloud follow the link [11. meta-test-layer example](https://github.com/FengD/meta-test-layer).

### 2.3. Build third-party library
You can use devtool to add the recipe if you are no 2.4+ version of yocto release
```sh
devtool add libsml https://github.com/dailab/libsml
```
it will create a recipe template `workspace/recipes/libsml/libsml_git.bb` this is nearly what you need but sometimes you have to tweak it a bit to ensure cross compiling.

in this case it builds and runs the tests, obviously when cross building we can build the tests but we can run them on build machine, so you have to disable that. you can do so in recipe or via a patch. e.g. via recipe you will change do_configure function to something like this
```sh
do_configure () {
    # Specify any needed configure commands here
    sed -i -e "s#@./test##g" ${S}/test/Makefile
}
```
may be change do_install as well so it can install the files you need on target
```sh
do_install () {
    install -d ${D}${libdir} ${D}${includedir}
    install -m 0644 ${B}/sml/lib/libsml.* ${D}${libdir}
    rm -rf ${D}${libdir}/libsml.o
    cp -R --no-dereference --preserve=mode,links ${S}/sml/include/* ${D}${includedir}
    install -D -m 0644 sml.pc ${D}${libdir}/pkgconfig/sml.pc
}
```
to build and see if all is ok
```sh
devtool build libsml
```
if all builds you can then apply the recipe to a layer of your choice ( say meta-oe )
```sh
devtool finish libsml meta-oe -f
```
Thats it, now you should see the recipe in meta-oe layer, you can try to build it
```sh
bitbake libsml
```

### 2.4. Add other layers and recipes
#### 2.4.1. Add docker
Reference [8] gives you an example of how to add docker in your image. In brief, I give you the important steps below.

* Clone and add the layers `meta-virtualization` in your project.

```sh
git clone -b dunfell git://git.openembedded.org/meta-openembedded
# bblayers.conf is like
BBLAYERS ?= " \
  /yocto/dunfell/poky-dunfell/meta \
  /yocto/dunfell/poky-dunfell/meta-poky \
  /yocto/dunfell/poky-dunfell/meta-yocto-bsp \
  /yocto/dunfell/layers/meta-openembedded/meta-oe \
  /yocto/dunfell/layers/meta-openembedded/meta-multimedia \
  /yocto/dunfell/layers/meta-openembedded/meta-networking \
  /yocto/dunfell/layers/meta-openembedded/meta-perl \
  /yocto/dunfell/layers/meta-openembedded/meta-python \
  /yocto/dunfell/layers/meta-openembedded/meta-filesystems \
  /yocto/dunfell/layers/meta-qt5 \
  /yocto/dunfell/layers/meta-raspberrypi \
  /yocto/dunfell/layers/meta-security \
  /yocto/dunfell/layers/meta-jumpnow \
  /yocto/dunfell/layers/meta-rpi64 \
  /yocto/dunfell/layers/meta-test-layer \
  /yocto/dunfell/layers/meta-virtualization \
  "
```

* Add `docker-ce` in your .conf

```sh
#you could find the docker-ce
bitbake -s |grep docker
# add image install in your local.conf file
IMAGE_INSTALL_append = " docker-ce"
```

* Errors might meet

```sh
#ERROR. input file "cfg/virtio.scc" does not exist is a error.
#go to the file /home/ding/Documents/yocto/dunfell/layers/meta-virtualization/recipes-kernel/linux/linux-yocto_virtualization.inc. Replace the following line
KERNEL_FEATURES_append = " cfg/virtio.scc"
# with
KERNEL_FEATURES_append += "${@bb.utils.contains('DISTRO_FEATURES', 'cfg', ' features/cfg/virtio.scc', '', d)}"
```

### 2.5. Cross build application

```sh
ding@f763b617ea24:/yocto/rpi-build$  bitbake -s |grep toolchain
meta-extsdk-toolchain                                 :1.0-r0                          
meta-go-toolchain                                     :1.0-r0                          
meta-toolchain                                        :1.0-r7                          
nativesdk-icecc-toolchain                             :0.1-r0


#use
bitbake meta-toolchain
```

After finished successfully, you will find a `.sh` file in the `/deploy/sdk`

## 3. Dependencies
* ubuntu system or [docker image](https://hub.docker.com/repository/docker/dingfengffffff/yocto)
* yocto project
* The other meta layer projects

## 4. Tips
### Error List[10]
* 1. Root build error

```
ERROR:  OE-core's config sanity checker detected a potential misconfiguration.
    Either fix the cause of this error or at your own risk disable the checker (see sanity.conf).
    Following is the list of potential problems / advisories:

    Do not use Bitbake as root.
```

```
Solution:
in file "yocto-bsp/sources/poky/meta/classes/sanity.bbclass"
uncomment
#if 0 == os.getuid():
#    raise_sanity_error("Do not use Bitbake as root.", d)

or

use user no root
```

* 2. Branch not same

```
ERROR: gnu-config-native-20150728+gitAUTOINC+b576fa87c1-r0 do_unpack: Function failed: Fetcher failure: Fetch command failed with exit code 128, output:

fatal: the '--set-upstream' option is no longer supported. Please use '--track' or '--set-upstream-to' instead.

ERROR: Logfile of failure stored in: /home/leo/work/imx6/genivi-imx6/build-genivi/tmp/work/x86_64-linux/gnu-config-native/20150728+gitAUTOINC+b576fa87c1-r0/temp/log.do_unpack.3948
```

```
Solution:
1. update git
2. change branche
```


## 5. References
* [1. yocto](https://www.yoctoproject.org/)
* [2. bitbake](https://www.yoctoproject.org/docs/1.6/bitbake-user-manual/bitbake-user-manual.html)
* [3. openembedded layers list](http://layers.openembedded.org/layerindex/branch/dunfell/layers/)
* [4. meta-rpi64](https://github.com/jumpnow/meta-rpi64)
* [5. create your own meta-layers](https://blog.csdn.net/azure_2010/article/details/89381196)
* [6. default raspberrypi linux kernal](github.com/raspberrypi/linux)
* [7. yocto build Raspberrypi4 image](https://jumpnowtek.com/rpi/Raspberry-Pi-4-64bit-Systems-with-Yocto.html)
* [8. run docker in Raspberrypi4](https://medium.com/@shantanoodesai/run-docker-on-a-raspberry-pi-4-with-yocto-project-551d6b615c0b)
* [9. study note](https://blog.csdn.net/weixin_42275611/article/details/105909067)
* [10. yocto build error summery](https://blog.csdn.net/bird_fly1024/article/details/81451662)
* [11. meta-test-layer example](https://github.com/FengD/meta-test-layer)
* [12. build 3rd party library](https://stackoverflow.com/questions/52059266/how-to-add-a-third-party-library-as-a-package-in-yocto-build)
* [13. yocto environment docker image](https://hub.docker.com/repository/docker/dingfengffffff/yocto)
* [14. yocto project introduction](https://elinux.org/Yocto_Project_Introduction)
* [15. raspberrypi4](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/?resellerType=home)
* [16. raspberrypi4 startup](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up)
