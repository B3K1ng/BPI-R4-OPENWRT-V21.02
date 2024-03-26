# BPI-R4-OPENWRT-V21.02  The BSP don't include any wifi driver and don't support BE19000, BE13500 WiFi Card。

![OpenWrt logo](include/logo.png)

OpenWrt Project is a Linux operating system targeting embedded devices. Instead
of trying to create a single, static firmware, OpenWrt provides a fully
writable filesystem with package management. This frees you from the
application selection and configuration provided by the vendor and allows you
to customize the device through the use of packages to suit any application.
For developers, OpenWrt is the framework to build an application without having
to build a complete firmware around it; for users this means the ability for
full customization, to use the device in ways never envisioned.

Sunshine!

## Development

To build your own firmware you need a GNU/Linux, BSD or MacOSX system (case
sensitive filesystem required). Cygwin is unsupported because of the lack of a
case sensitive file system.

### Requirements

You need the following tools to compile OpenWrt, the package names vary between
distributions. A complete list with distribution specific packages is found in
the [Build System Setup](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)
documentation.

```
gcc binutils bzip2 flex python3 perl make find grep diff unzip gawk getopt
subversion libz-dev libc-dev rsync which
```

### Alternative Quickstart

tl;dr

Building step you can find https://github.com/danpawlik/openwrt-builder

```sh
cat << EOF >> feeds.conf.default
src-git mtksdk https://git01.mediatek.com/openwrt/feeds/mtk-openwrt-feeds
EOF
git add feeds.conf.default; git commit -m "Add MediaTek feeds"

./scripts/feeds update -a && ./scripts/feeds install -a

sed -i 's@        --with-ncursesw@        --with-ncursesw \        --without-cryptsetup@g' package/utils/util-linux/Makefilea

### If still cryptsetup is needed:
# make -j $(nproc) package/cryptsetup/{clean,compile}

# build
## WARNING: there is some missing config param to set, so let's
## build it by using V=s, so it will ask if the module should be
## included. Otherwise, it will fail.
## Workaround:
echo "n" | make -j $(nproc)  defconfig download clean world V=s

# NOTE: build OpenWRT in openwrt dir

# more easy way
# tl;dr to create needed files:

git clone https://github.com/mtk-openwrt/u-boot && cd u-boot
make mt7988_sd_rfb_defconfig
make CROSS_COMPILE=/home/user/BPI-R4-OPENWRT-V21.02/staging_dir/toolchain-aarch64_cortex-a53_gcc-8.4.0_musl/bin/aarch64-openwrt-linux-

# get required files
# FIXME: Should it be with --sdmmc ?
python2 \
    ./openwrt/build_dir/target-aarch64_cortex-a53_musl/arm-trusted-firmware-mediatek-mt7988-sdmmc-comb/arm-trusted-firmware-mediatek-2023.10.13~0ea67d76/tools/dev/gpt_editor/mtk_gpt.py \
    --i ./openwrt/build_dir/target-aarch64_cortex-a53_musl/arm-trusted-firmware-mediatek-mt7988-sdmmc-comb/arm-trusted-firmware-mediatek-2023.10.13~0ea67d76/tools/dev/gpt_editor/example/mt7988-sd.json \
    --o /tmp/GPT_SD \
    --sdmmc

# sd
cp /tmp/GPT_SD BPI-R4-OPENWRT-V21.02/staging_dir/target-aarch64_cortex-a53_musl/image/GPT_SD

cp openwrt/build_dir/target-aarch64_cortex-a53_musl/u-boot-mt7988_bananapi_bpi-r4-sdmmc/u-boot-2024.01/u-boot.fip \
    BPI-R4-OPENWRT-V21.02/staging_dir/target-aarch64_cortex-a53_musl/image/fip_sd.bin

cp openwrt/build_dir/target-aarch64_cortex-a53_musl/arm-trusted-firmware-mediatek-mt7988-sdmmc-comb/arm-trusted-firmware-mediatek-2023.10.13~0ea67d76/build/mt7988/release/bl2.img \
    BPI-R4-OPENWRT-V21.02/staging_dir/target-aarch64_cortex-a53_musl/image/bl2_sd.img

sudo python2 ./openwrt/build_dir/target-aarch64_cortex-a53_musl/arm-trusted-firmware-mediatek-mt7988-sdmmc-comb/arm-trusted-firmware-mediatek-2023.10.13~0ea67d76/tools/dev/gpt_editor/mtk_gpt.py \
--i ./openwrt/build_dir/target-aarch64_cortex-a53_musl/arm-trusted-firmware-mediatek-mt7988-sdmmc-comb/arm-trusted-firmware-mediatek-2023.10.13~0ea67d76/tools/dev/gpt_editor/example/mt7988-emmc.json  \
--o /tmp/GPT_EMMC

# emmc
cp /tmp/GPT_EMMC BPI-R4-OPENWRT-V21.02/staging_dir/target-aarch64_cortex-a53_musl/image/GPT_EMMC

cp openwrt/build_dir/target-aarch64_cortex-a53_musl/arm-trusted-firmware-mediatek-mt7988-emmc-comb/arm-trusted-firmware-mediatek-2023.10.13~0ea67d76/build/mt7988/release/bl2.img \
   BPI-R4-OPENWRT-V21.02/staging_dir/target-aarch64_cortex-a53_musl/image/bl2_emmc.img

cp openwrt/build_dir/target-aarch64_cortex-a53_musl/u-boot-mt7988_bananapi_bpi-r4-emmc/u-boot-2024.01/u-boot.fip \
  BPI-R4-OPENWRT-V21.02/staging_dir/target-aarch64_cortex-a53_musl/image/fip_emmc.bin

# nand
cp openwrt/build_dir/target-aarch64_cortex-a53_musl/arm-trusted-firmware-mediatek-mt7988-spim-nand-ubi-comb/arm-trusted-firmware-mediatek-2023.10.13~0ea67d76/build/mt7988/release/bl2.img \
  BPI-R4-OPENWRT-V21.02/staging_dir/target-aarch64_cortex-a53_musl/image/bl2_nand.img

cp openwrt/build_dir/target-aarch64_cortex-a53_musl/u-boot-mt7988_bananapi_bpi-r4-snand/u-boot-2024.01/u-boot.fip \
   BPI-R4-OPENWRT-V21.02/staging_dir/target-aarch64_cortex-a53_musl/image/fip_nand.bin

```

### Quickstart

1. Run `./scripts/feeds update -a` to obtain all the latest package definitions
   defined in feeds.conf / feeds.conf.default

2. Run `./scripts/feeds install -a` to install symlinks for all obtained
   packages into package/feeds/

3. Run `make menuconfig` to select your preferred configuration for the
   toolchain, target system & firmware packages.

4. Run `make` to build your firmware. This will download all sources, build the
   cross-compile toolchain and then cross-compile the GNU/Linux kernel & all chosen
   applications for your target system.

### Related Repositories

The main repository uses multiple sub-repositories to manage packages of
different categories. All packages are installed via the OpenWrt package
manager called `opkg`. If you're looking to develop the web interface or port
packages to OpenWrt, please find the fitting repository below.

* [LuCI Web Interface](https://github.com/openwrt/luci): Modern and modular
  interface to control the device via a web browser.

* [OpenWrt Packages](https://github.com/openwrt/packages): Community repository
  of ported packages.

* [OpenWrt Routing](https://github.com/openwrt/routing): Packages specifically
  focused on (mesh) routing.

## Support Information

For a list of supported devices see the [OpenWrt Hardware Database](https://openwrt.org/supported_devices)

### Documentation

* [Quick Start Guide](https://openwrt.org/docs/guide-quick-start/start)
* [User Guide](https://openwrt.org/docs/guide-user/start)
* [Developer Documentation](https://openwrt.org/docs/guide-developer/start)
* [Technical Reference](https://openwrt.org/docs/techref/start)

### Support Community

* [Forum](https://forum.openwrt.org): For usage, projects, discussions and hardware advise.
* [Support Chat](https://webchat.oftc.net/#openwrt): Channel `#openwrt` on **oftc.net**.

### Developer Community

* [Bug Reports](https://bugs.openwrt.org): Report bugs in OpenWrt
* [Dev Mailing List](https://lists.openwrt.org/mailman/listinfo/openwrt-devel): Send patches
* [Dev Chat](https://webchat.oftc.net/#openwrt-devel): Channel `#openwrt-devel` on **oftc.net**.

## License

OpenWrt is licensed under GPL-2.0
