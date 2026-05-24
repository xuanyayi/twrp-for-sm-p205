# TWRP for Samsung Galaxy Tab A 8.0 with S Pen (SM-P205)
![Galaxy Tab A8](https://fdn2.gsmarena.com/vv/pics/samsung/samsung-galaxy-tab-a-s-pen-sm-p205-2.jpg "Galaxy Tab A8")

## Sync source

```bash
mkdir -p ~/twrp && cd ~/twrp

repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-12.1
repo sync
```

## Clone device tree

```bash
git clone https://github.com/xuanyayi/twrp-for-sm-p205 device/samsung/p205
```

## Build

```bash
cd ~/twrp
source build/envsetup.sh
lunch twrp_p205-eng
mka recoveryimage
```

The output image is generated at:

```text
out/target/product/p205/recovery.img
```

## Notes

- Target device: Samsung SM-P205, codename `p205`
- Recovery variant: TWRP 12.1
- Odin flashing uses `recovery.img` packed as `recovery.img` inside a tar archive.
- If you want to silence the harmless `indeterminate014` progress animation probe in logs, patch the TWRP theme resources in `bootable/recovery`; this device tree does not vendor TWRP framework assets.
