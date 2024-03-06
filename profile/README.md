# Redroid Rockchip

- [x] GPU (mali-G610)
- [x] GApps: https://gitlab.com/MindTheGapps/vendor_gapps
- [x] Magisk Delta App 27.0: https://github.com/KitsuneMagisk/Magisk
- [x] Virtual wifi: enable the mac80211_hwsim module and switch to iptables-legacy
- [ ] Virtual gps
- [ ] Virtual camera
- [ ] Virtual battery
- [ ] Virtual sensor
- [ ] Spoof device ids
- [ ] ...

![rk3588](https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/redroid-rk3588.png)


## Build redroid with docker

```bash
#####################
# fetch code
#####################
mkdir ~/redroid && cd ~/redroid

repo init -u https://github.com/redroid-rockchip/platform_manifests.git -b redroid-12.0.0 --depth=1 --git-lfs
repo sync -c

# 修改build/soong/cc/config/global.go，向commonGlobalCflags数组添加全局cflags "-DANDROID_12"
vim build/soong/cc/config/global.go
# var (
#	// Flags used by lots of devices.  Putting them in package static variables
#	// will save bytes in build.ninja so they aren't repeated for every file
#	commonGlobalCflags = []string{
#		"-DANDROID_12",  <================================================ add this line
#		"-DANDROID",
#		"-fmessage-length=0",

#####################
# create builder
#####################
cd ~/ && git clone https://github.com/remote-android/redroid-doc.git
cd redroid-doc/android-builder-docker/
docker build --build-arg userid=$(id -u) --build-arg groupid=$(id -g) --build-arg username=$(id -un) -t redroid-builder .

#####################
# start builder
#####################
docker run -it --rm --hostname redroid-builder --name redroid-builder -v ~/redroid:/src redroid-builder

#####################
# build redroid
#####################
cd /src

. build/envsetup.sh
lunch redroid_arm64-userdebug

# 我们编译的是rk3588，用的gpu是mali-G610
# 但是hardware/rockchip/librkvpu/omx_get_gralloc_private/Android.go没有定义mali-G610
# 所以此处选择同用bifrost库的mali-G52
export TARGET_BOARD_PLATFORM_GPU=mali-G52
export TARGET_RK_GRALLOC_VERSION=4

# start to build
m

#####################
# create redroid image in *HOST*
#####################
cd ~/redroid/out/target/product/redroid_x86_64

sudo mount system.img system -o ro
sudo mount vendor.img vendor -o ro
sudo tar --xattrs -c vendor -C system --exclude="./vendor" . | docker import -c 'ENTRYPOINT ["/init", "androidboot.hardware=redroid"]' - redroid
sudo umount system vendor
```

Export redroid image to rk3588 board
```bash
docker save redroid | ssh root@rock.huji.show docker load
```

Run docker container with redroid image
```bash
# 添加androidboot.redroid_gpu_mode=mali开启gpu硬解
# 添加androidboot.redroid_virtiowifi=1开启虚拟wifi
sudo docker run -itd --privileged \
    --name redroid \
    -v ~/data:/data \
    -v /dev/mali0:/dev/mali0 \
    -p 5555:5555 \
    redroid \
    androidboot.redroid_gpu_mode=mali \
    androidboot.redroid_virtiowifi=1
```

## Other

Fix webview error: https://github.com/remote-android/redroid-doc/issues/464
```bash
## install lfs
# apt install git-lfs
## then run
repo forall -g lfs -c git lfs pull
```
