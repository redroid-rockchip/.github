# Redroid Rockchip

## Features


<details>
<summary> ✅ GPU (mali-G610) </summary>

<img src="https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/mali.png"/>
</details>


<details>
<summary> ✅ GApps 👉 https://gitlab.com/MindTheGapps/vendor_gapps </summary>

<img src="https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/gapps.png" width="432px" height="768px"/>
</details>


<details>
<summary> ✅ Magisk Delta 👉 https://github.com/KitsuneMagisk/Magisk </summary>

<img src="https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/magisk.png" width="432px" height="768px"/>
</details>


<details>
<summary> ✅ Virtual wifi 👉 https://github.com/redroid-rockchip/vendor_redroid_ext/tree/master/wifi </summary>

##### Required
1. `mac80211_hwsim` kernel module in host
2. switch to `iptables-legacy` in host

<img src="https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/wifi.png" width="432px" height="768px"/>
</details>


<details>
<summary> ✅ Virtual gps 👉 https://github.com/redroid-rockchip/vendor_redroid_ext/tree/master/gps </summary>

##### Update latitude and longitude
```bash
adb shell 'echo "LatitudeDegrees=30.281026818001678" > /data/vendor/gps/gnss'
adb shell 'echo "LongitudeDegrees=120.01934876982831" >> /data/vendor/gps/gnss'
adb shell 'echo "AltitudeMeters=1.60062531" >> /data/vendor/gps/gnss'
adb shell 'echo "BearingDegrees=0" >> /data/vendor/gps/gnss'
adb shell 'echo "SpeedMetersPerSec=0" >> /data/vendor/gps/gnss'
```

<img src="https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/gps.png" width="432px" height="768px"/>
</details>


<details>
<summary> ✅ Virtual battery 👉 https://github.com/redroid-rockchip/vendor_redroid_ext/tree/master/battery </summary>

##### Update battery capacity
```bash
adb shell 'echo 88 > /data/vendor/battery/power_supply/battery/capacity'
```

<img src="https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/battery.png" width="432px" height="768px"/>
</details>


<details open>
<summary> Virtual camera </summary>
</details>


<details open>
<summary> Virtual sensor </summary>
</details>


<details open>
<summary> Enable webview debug </summary>
</details>


<details open>
<summary> Spoof device ids </summary>
</details>


<details open>
<summary> Disable ssl pinning </summary>
</details>


<details>
<summary> ✅ Disable window flag FLAG_SECURE </summary>
</details>


<details open>
<summary> ... </summary>
</details>

## Getting Started

```bash
# Add androidboot.redroid_gpu_mode=mali to enable GPU hardware decoding.
# Add androidboot.redroid_virtual_wifi=1 to enable virtual WiFi.
sudo docker run -itd --privileged \
    --name redroid \
    -v ~/data:/data \
    -v /dev/mali0:/dev/mali0 \
    -p 5555:5555 \
    iceblacktea/redroid-arm64:12.0.0-240328 \
    androidboot.redroid_gpu_mode=mali \
    androidboot.redroid_virtual_wifi=1
```

## Experimental

### [Experimental] Dependency

1. Install Python3, Git, [Docker](https://docs.docker.com/engine/install/ubuntu/#install-using-the-convenience-script), [Docker Compose](https://docs.docker.com/compose/install/linux/)

```bash
wget -qO- get.docker.com | bash
sudo apt-get update
sudo apt-get install -y python3 git docker-compose-plugin
```

2. Install linktools library and add redroid repository

```bash
python3 -m pip install -U "linktools[container]"
ct-cntr repo add https://github.com/ice-black-tea/cntr-mobile  # add remote repository
ct-cntr repo update  # update code from remote repository
```

### [Experimental] Run redroid in arm64 Board

```bash
ct-cntr add redroid  # add redroid to the installation list
ct-cntr up  # start redroid containers
```

### [Experimental] Build in x86_64 PC

```bash
ct-cntr add redroid-builder  # add redroid-builder to the installation list

#####################
# create and start builder
#####################
# We are compiling for rk3588, using the Mali-G610 GPU.
# However, in hardware/rockchip/librkvpu/omx_get_gralloc_private/Android.go, Mali-G610 is not defined.
# Therefore, we are selecting Mali-G52, which also utilizes the Bifrost library, for this purpose.
ct-cntr exec redroid-builder set-env TARGET_BOARD_PLATFORM_GPU=mali-G52 TARGET_RK_GRALLOC_VERSION=4
ct-cntr config set REDROID_BUILD_PATH=~/redroid # set the path to store source code
ct-cntr config  # check whether the docker configuration is correct
ct-cntr up  # start redroid-builder container

#####################
# fetch code
#####################
ct-cntr exec redroid-builder init-repo -u https://github.com/redroid-rockchip/platform_manifests.git -b redroid-12.0.0
ct-cntr exec redroid-builder sync-repo

#####################
# build redroid
#####################
ct-cntr exec redroid-builder build-arm64

#####################
# create redroid image
#####################
ct-cntr exec redroid-builder make-arm64-image
```

Export redroid image to rockchip in x86_64 PC
```bash
docker save redroid | ssh root@rock.huji.show docker load
```

## Build redroid

```bash
#####################
# fetch code
#####################
mkdir ~/redroid && cd ~/redroid

repo init -u https://github.com/redroid-rockchip/platform_manifests.git -b redroid-12.0.0 --depth=1 --git-lfs
repo sync -c

# apt install git-lfs and then
repo forall -g lfs -c git lfs pull

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
cd ~/redroid/out/target/product/redroid_arm64

sudo mount system.img system -o ro
sudo mount vendor.img vendor -o ro
sudo tar --xattrs -c vendor -C system --exclude="./vendor" . | DOCKER_DEFAULT_PLATFORM=linux/arm64 docker import -c 'ENTRYPOINT ["/init", "androidboot.hardware=redroid"]' - redroid
sudo umount system vendor
```

Export redroid image to board
```bash
docker save redroid | ssh root@rock.huji.show docker load
```

## Issues

click 👉 https://github.com/redroid-rockchip/.github/issues
