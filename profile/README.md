# Redroid Rockchip

## Features


<details>
<summary> ✅ GPU (mali-G610) </summary>

![mali](https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/mali.png)
</details>


<details>
<summary> ✅ GApps 👉 https://gitlab.com/MindTheGapps/vendor_gapps </summary>

![gapps](https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/gapps.png)
</details>


<details>
<summary> ✅ Magisk Delta 👉 https://github.com/KitsuneMagisk/Magisk </summary>

![magisk](https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/magisk.png)
</details>


<details>
<summary> ✅ Virtual wifi 👉 https://github.com/redroid-rockchip/vendor_redroid_ext/tree/master/wifi </summary>

##### Required in host
1. `mac80211_hwsim` kernel module
2. switch to `iptables-legacy`

![wifi](https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/wifi.png)
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

![gps](https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/gps.png)
</details>


<details>
<summary> ✅ Virtual battery 👉 https://github.com/redroid-rockchip/vendor_redroid_ext/tree/master/battery </summary>

##### Update latitude and longitude
```bash
adb shell 'echo 88 > /data/vendor/battery/power_supply/battery/capacity'
```

![battery](https://raw.githubusercontent.com/redroid-rockchip/.github/main/images/battery.png)
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


## Build redroid

```bash
#####################
# fetch code
#####################
mkdir ~/redroid && cd ~/redroid

repo init -u https://github.com/redroid-rockchip/platform_manifests.git -b redroid-12.0.0 --depth=1 --git-lfs
repo sync -c

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
# 添加androidboot.redroid_virtual_wifi=1开启虚拟wifi
sudo docker run -itd --privileged \
    --name redroid \
    -v ~/data:/data \
    -v /dev/mali0:/dev/mali0 \
    -p 5555:5555 \
    redroid \
    androidboot.redroid_gpu_mode=mali \
    androidboot.redroid_virtual_wifi=1
```

## Issues

click 👉 https://github.com/redroid-rockchip/.github/issues

## Other

Fix webview error: https://github.com/remote-android/redroid-doc/issues/464
```bash
## install lfs
# apt install git-lfs
## then run
repo forall -g lfs -c git lfs pull
```
