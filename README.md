# TWRP for Samsung Galaxy Tab A 8.0 with S Pen (SM-P205)

![Galaxy Tab A8](https://fdn2.gsmarena.com/vv/pics/samsung/samsung-galaxy-tab-a-s-pen-sm-p205-2.jpg "Galaxy Tab A8")

This device tree builds TWRP 12.1 for the Samsung Galaxy Tab A 8.0 with S Pen
LTE (`SM-P205`). Samsung codenames for this device family are `wisdom` and
`wisdomx`.

## Build Environment

Use a Linux build host. Ubuntu 20.04/22.04 or WSL2 Ubuntu is recommended.

Install common Android build packages:

```bash
sudo apt update
sudo apt install -y \
  bc bison build-essential ccache curl flex g++-multilib gcc-multilib \
  git git-lfs gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev \
  lib32z1-dev libelf-dev liblz4-tool libncurses5 libncurses5-dev \
  libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush \
  rsync schedtool squashfs-tools unzip xsltproc zip zlib1g-dev
```

Install the `repo` tool if it is not already available:

```bash
mkdir -p ~/.bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
chmod a+x ~/.bin/repo
export PATH="$HOME/.bin:$PATH"
```

For WSL2, build inside the Linux filesystem, for example under `~/twrp`.
Building under `/mnt/c` or another Windows-mounted path is much slower and can
cause file permission issues.

## Sync TWRP Source

```bash
mkdir -p ~/twrp
cd ~/twrp

repo init --depth=1 \
  -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git \
  -b twrp-12.1

repo sync -c --no-tags --no-clone-bundle --optimized-fetch --fail-fast -j"$(nproc)"
```

If the sync fails because of network instability, rerun the same `repo sync`
command. On very unstable networks, use `-j1`.

## Clone Device Tree

Clone this repository into the Android source tree exactly at
`device/samsung/p205`:

```bash
cd ~/twrp
rm -rf device/samsung/p205
git clone https://github.com/xuanyayi/twrp-for-sm-p205.git device/samsung/p205
```

The expected device tree files include:

```text
device/samsung/p205/AndroidProducts.mk
device/samsung/p205/BoardConfig.mk
device/samsung/p205/twrp_p205.mk
device/samsung/p205/prebuilt/Image
device/samsung/p205/prebuilt/recovery_dtbo
```

## Apply Required Patches

This device tree depends on a few small TWRP 12.1 base-tree patches for the
P205 build and recovery runtime. Apply them from the Android source root before
building:

```bash
cd ~/twrp
P205_TREE="$PWD/device/samsung/p205"

git -C build/make apply "$P205_TREE/patches/0001-build-make-p205-recovery-props.patch"
git -C system/sepolicy apply "$P205_TREE/patches/0002-system-sepolicy-drop-deprecated-board-plat-policy.patch"
git -C bootable/recovery apply "$P205_TREE/patches/0003-bootable-recovery-p205-twrp12-support.patch"
```

If you want to verify first without changing files:

```bash
cd ~/twrp
P205_TREE="$PWD/device/samsung/p205"

git -C build/make apply --check "$P205_TREE/patches/0001-build-make-p205-recovery-props.patch"
git -C system/sepolicy apply --check "$P205_TREE/patches/0002-system-sepolicy-drop-deprecated-board-plat-policy.patch"
git -C bootable/recovery apply --check "$P205_TREE/patches/0003-bootable-recovery-p205-twrp12-support.patch"
```

If a patch reports that it is already applied, skip that patch.

## Build

```bash
cd ~/twrp
export ALLOW_MISSING_DEPENDENCIES=true
source build/envsetup.sh
lunch twrp_p205-eng
mka recoveryimage
```

The output image is:

```text
out/target/product/p205/recovery.img
```

For a clean rebuild:

```bash
cd ~/twrp
rm -rf out/target/product/p205/recovery \
       out/target/product/p205/ramdisk-recovery.img \
       out/target/product/p205/recovery.img

export ALLOW_MISSING_DEPENDENCIES=true
source build/envsetup.sh
lunch twrp_p205-eng
mka recoveryimage
```

## Odin Package

To flash with Odin, pack `recovery.img` into a tar archive:

```bash
cd ~/twrp/out/target/product/p205
tar -H ustar -c recovery.img > twrp_p205_12.1.tar
```

Flash `twrp_p205_12.1.tar` with Odin's AP slot.

## Known Build Notes

- Correct manifest branch: `twrp-12.1`
- Correct device path: `device/samsung/p205`
- Correct lunch target: `twrp_p205-eng`
- Recovery kernel: `prebuilt/Image`
- Recovery DTBO: `prebuilt/recovery_dtbo`
- Recovery partition size: `39845888` bytes
- `ALLOW_MISSING_DEPENDENCIES=true` is expected for this minimal recovery tree.
- Optional TWRP extras are trimmed to keep `recovery.img` inside the stock
  recovery partition size.

## Common Problems

### `lunch twrp_p205-eng` is not listed

Check that the device tree is in the correct path:

```bash
ls device/samsung/p205/AndroidProducts.mk
```

If the file is missing, clone this repository again into `device/samsung/p205`.

### `vendor/twrp/config/common.mk` not found

The wrong manifest was synced. Reinitialize and sync the TWRP 12.1 AOSP
manifest:

```bash
repo init --depth=1 \
  -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git \
  -b twrp-12.1
repo sync -c --no-tags --no-clone-bundle --optimized-fetch --fail-fast -j"$(nproc)"
```

### Patch application fails

Make sure the patches are applied from the Android source root, not from inside
`device/samsung/p205`:

```bash
cd ~/twrp
```

If the source tree was synced from a different TWRP branch, re-sync `twrp-12.1`.

### `recovery.img` is larger than the recovery partition

Do not enable extra languages, NTFS/exFAT userspace tools, repack tools, bash,
nano, tzdata, or zip unless you also verify the final image size. The stock
recovery partition limit for this device is `39845888` bytes.

Check the image size with:

```bash
stat -c '%n %s bytes' out/target/product/p205/recovery.img
```

### `mka: command not found`

Load the Android build environment first:

```bash
source build/envsetup.sh
```

Then run `lunch twrp_p205-eng` again before building.

## Device Notes

- Device: Samsung Galaxy Tab A 8.0 with S Pen LTE
- Model: `SM-P205`
- Samsung codename: `wisdom` / `wisdomx`
- Platform: Exynos 7904 / universal7904
- Device tree path: `device/samsung/p205`
- Recovery variant: TWRP 12.1
- TWRP version from this branch: `3.7.1_12-0`
