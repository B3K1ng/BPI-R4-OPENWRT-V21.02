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

Building step you can find [here](https://github.com/danpawlik/openwrt-builder)

Before you start, remember to comment:

```sh
## https://github.com/BPI-SINOVOIP/BPI-R4-OPENWRT-V21.02/blob/main/package/Makefile#L64-L65
#$(curdir)/compile: $(curdir)/cryptsetup/host/compile
#$(curdir)/compile: $(curdir)/dtc/host/compile
```

located in `package/Makefile`.

Add MediaTek feed:

```sh
cat << EOF >> feeds.conf
src-git-full mtk_openwrt_feed https://git01.mediatek.com/openwrt/feeds/mtk-openwrt-feeds
EOF
git add feeds.conf; git commit -m "Add MediaTek feeds"

./scripts/feeds update -a && ./scripts/feeds install -a

```

Fix `kmod-nf-flow-netlink` error:
```MakeFile
-- DEPENDS:=+libnfnetlink +libmnl +kmod-nf-flow-netlink
++ DEPENDS:=+libnfnetlink +libmnl +kmod-nf-flow +kmod-nfnetlink
```

WARNING: If GPT_SD or other files are missing, you can find in this project in `bpi-r4-files` dir.
The files are available in this [project](https://github.com/BPI-SINOVOIP/BPI-R4-OPENWRT-V21.02/tree/main/staging_dir/target-aarch64_cortex-a53_musl/image).
The `make clean` command remove them.
You can just restore the file before running `make world`:

```sh
git checkout staging_dir/target-aarch64_cortex-a53_musl/image
```

NOTE: Workaround for mt76 after pulling mediatek feed

If you choose some mt76 drivers, before executing `make world`, try:

```sh
make -j1 V=s package/kernel/mt76/{clean,compile}
```

Probably it will fail.
Here are two options:

* remove duplicated patches
* remove mediatek feed from mt76

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
