This instruction explains step by step how to build a platform image including Chromium ozone-gbm using Yocto.

## Build a platform image with Yocto.
### Clone Yocto recipe repositories that need to build Chromium.
```
$ mkdir yocto
$ cd yocto/
$ git clone https://github.com/joone/poky.git
$ cd poky
$ git checkout pyro_gbm  (latest stable branch)
$ cd ..
$ git clone https://github.com/joone/meta-crosswalk
$ git clone https://github.com/imyller/meta-nodejs.git
$ git clone git://git.yoctoproject.org/meta-intel.git
$ git clone git://git.openembedded.org/meta-openembedded
```
### Checkout chromium58 branch.
```
$ cd meta-crosswalk
$ git checkout chromium58_gbm
$ cd ../yocto/poky
$ source oe-init-build-env
$ cd build
```
### Edit the Yocto build configuration
You can clone my build configuration. For this, remove the generated conf folder.
```
$ rm -rf conf
$ git clone https://github.com/joone/yocto-build-conf.git
```
Instead, you need to update the path of recipes in bblayer.conf.

You can also edit bblayer.conf and local.conf in the generated conf folder as follows:
```
$ vi conf/bblayer.conf
```
Add meta-crosswalk and meata-nodejs
```
BBLAYERS ?= " \
/home/joone/git/yocto/poky/meta \
/home/joone/git/yocto/poky/meta-yocto \
/home/joone/git/yocto/meta-crosswalk \
/home/joone/git/yocto/meta-nodejs \
/home/joone/otc/yocto/meta-intel \
/home/joone/otc/yocto/meta-openembedded/meta-oe \
/home/joone/otc/yocto/meta-openembedded/meta-filesystems \
"
```
```
$ vi conf/local.conf
```
Add following line to to include Chromium and its dependencies in the image.
```
IMAGE_INSTALL_append = " chromium"
CORE_IMAGE_EXTRA_INSTALL += "libgles2-mesa libegl"

```
Also, you can select a specific target machine as follows: (default: qemux86)
```
MACHINE ?= "intel-corei7-64"
```

Build Chromium and create an image using bitbak command.
```
$ bitbake core-image-base
```
In case of building only Chromium,
```
$ bitbake chromium
```

## Run your target image on Intel compute stick or MinnowBoard Max
First, you need to create a USB image as follows:
```
$ cd tmp/deploy/images/<build_arch_dir>
$ sudo dd if=<image_name>.hddimg of=/dev/sdc
```

If the board does not boot automactically, you can boot it manually from the EFI shell as follows:
```
Shell> fs0:
Shell> bootx64
```
You can test ozone-gbm by running ozone-demo or mus_demo:
```
$ /usr/lib/chromium/ozone-demo
$ /usr/lib/chromium/mash --service=mus_demo
```
Run chromium
```
$ /usr/lib/chromium/chrome https://github.com/joone/wiki/blob/master/performance_test_web_graphics.md
 --mash --no-sandbox --use-data-dir=/home/root/ --window-manager=simple_wm --ash-host-window-bounds=2160x1440

```
Run mash session
```
$ /usr/lib/chromium/mash https://github.com/joone/wiki/blob/master/performance_test_web_graphics.md
 --service=mash_session --no-sandbox --use-data-dir=/home/root/ --window-manager=simple_wm --ash-host-window-bounds=2160x1440
```
If your display resolution is 1920x1080, you don't need to pass --ash-host-window-bounds parameter.

Show frame rate counter and GPU memory usage
```
--show-fps-counter --vmodule="head*=1" --enable-logging=stderr
```
Enable zero-copy texture upload
```
--enable-native-gpu-memory-buffers  --use-gl=egl

```
Show log information
```
--enable-logging=stderr --v=2
```

Example
```
$ ./mash https://github.com/joone/wiki/blob/master/performance_test_web_graphics.md --service=mash_session  --no-sandbox --use-data-dir=/home/root/  --window-manager=simple_wm --enable-native-gpu-memory-buffers --use-gl=egl  --show-fps-counter --vmodule="head*=1" --enable-logging=stderr --v=2 --proxy-pac-url=http://autoproxy.intel.com 
```
 
## Build Chromium external source
The current Yocto recipe only works with the stable release of Chromium so there might be many build issues with ToT, but you can try it out.
```
INHERIT += "externalsrc"
EXTERNALSRC_pn-chromium = "/home/joone/otc/chromium/src"
```

## Tips

## Proxy support
When mash or chrome launches, we can pass a proxy auto-config (PAC) file as runtime parameter as follow:
```
--proxy-pac-url=URL
```

### Add truetype fonts
This is a temporary solution.
```
$ cd  ~yocto/poky/build/tmp/deploy/rpm 
$ rpm -ifvh corei7_64/fontconfig-utils-2.12.1-r0.corei7_64.rpm
$ rpm -ifvh noarch/ttf*
```
If there are not ttf rpm packages, build them as follows:
```
$ bitbake ttf-dejavu
$ bitbake ttf-hunkyfonts
```
### Find the build files
```
$ cd ~yocto/poky/build/tmp/work/core2-64-poky-linux/chromium/58.0.3029.110-r0/chromium-58.0.3029.110/out/Release
```
### To recompile your source code if you change a line in it.
```
$ bitbake -f -c compile chromium
```

## References
* https://layers.openembedded.org/layerindex/branch/master/recipes/
* http://www.yoctoproject.org/docs/latest/yocto-project-qs/yocto-project-qs.html
* https://github.com/ds-hwang/wiki/wiki/build-chromium-on-yocto
* https://github.com/rakuco/meta-crosswalk
* https://wiki.yoctoproject.org/wiki/Tracing_and_Profiling

