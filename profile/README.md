# How to get started

1. Start by preparing your system and follow https://developer.sony.com/open-source/aosp-on-xperia-open-devices/guides/aosp-build-instructions/build-aosp-android-13/
   * Instead of `https://github.com/sonyxperiadev/local_manifests` use `https://github.com/simonmicro-AndroidDevelopment-SODP/local_manifests` and checkout the alternative `simonmicro/android-13.0.0_r75` branch.
   * `lunch` the target `aosp_xqau52-userdebug` (or `aosp_xqau52-user` for a release build) and build the system.
2. Prepare your private singing keys (follow https://source.android.com/devices/tech/ota/sign_builds) and store them into `PRIVATE_KEYS` inside the workspace.
3. Run `make -j$(nproc) dist` to create the unsiged distribution files.
4. Run `sign_target_files_apks -o -d ./PRIVATE_KEYS out/dist/*-target_files-*.zip out/dist/signed-target_files.zip` to sign the distribution files.
5. ...and finalize by creating OTA and IMAGE archives:
    ```bash
    export RESULT_NAME=aosp-13-$(date '+%Y%m%d')-xqau52_DSDS-signed
    ota_from_target_files -k ./PRIVATE_KEYS/releasekey out/dist/signed-target_files.zip out/dist/$RESULT_NAME-ota_update.zip
    img_from_target_files out/dist/signed-target_files.zip out/dist/$RESULT_NAME.zip
    md5sum out/dist/$RESULT_NAME-ota_update.zip > out/dist/$RESULT_NAME-ota_update.zip.md5sum
    md5sum out/dist/$RESULT_NAME.zip > out/dist/$RESULT_NAME.zip.md5sum
    ```
6. Use the non-OTA package to flash all images to the system, as described in the Sony guide.
7. Do not forget updating **both** `oem` partitions (use `fastboot --set-active=a` and `fastboot reboot bootloader` to switch between them). Yes, you do not strictly need to do this, but before you break your system with the next update...

# How to modify / add root / Google Apps
For this you have to use another recovery, as the AOSP recovery is not capable of installing non-OTA / unsigned packages. I recommend using the recovery of the LOS build from here: https://xdaforums.com/t/rom-unofficial-lineageos-18-1-for-xperia-10-ii-gcam-performance.4219081/

To install it:
```bash
fastboot flash recovery recovery.img # from the LOS build
fastboot flash dtbo dtbo.img # this recovery is based on stock, so it needs another device tree
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
fastboot --disable-verity --disable-verification flash vbmeta_system vbmeta_system.img
fastboot reboot recovery
```

Now continue installing [NikGapps](https://nikgapps.com/) / [MindTheGapps](https://mindthegapps.com/) or [Magisk](https://github.com/topjohnwu/Magisk/releases).

**Before** booting back into the system, make sure to revert the recovery and verified-boot partitions:
```bash
fastboot flash recovery recovery.img # from the AOSP build
fastboot flash dtbo dtbo.img # also restore the original device tree
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
fastboot --disable-verity --disable-verification flash vbmeta_system vbmeta_system.img
fastboot reboot recovery # do this to ensure your system is not marked as corrupted
```