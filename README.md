<!-- markdownlint-disable MD013 -->
# OpenWrt package feed for [luainkernel](https://github.com/luainkernel)

The feed is comprised by the following packages:

- lunatik
- lua5.4 (as a dependency)

> [!NOTE]
> This feed has been tested only on [OpenWrt 23.05.5](https://github.com/openwrt/openwrt/releases/tag/v23.05.5).

## Feed configuration

In order to use this feed into an OpenWrt build, add the following line to your `feeds.conf.default`:

```sh
src-git luainkernel https://github.com/luainkernel/openwrt_feed.git;openwrt-23.05
```

After that, update and install the feed:

```sh
./scripts/feeds update luainkernel
./scripts/feeds install -a -p luainkernel
```

> [!NOTE]
> Refer to [OpenWrt Feeds](https://openwrt.org/docs/guide-developer/feeds) for more information.

## Build instructions

> [!IMPORTANT]
> Lua 5.4 is required for building Lunatik.
> Make sure to install it on the build machine.

### Configure buildroot

Setup the target platform configuration as usual but make sure to select `kmod-lunatik` under the following path:

```txt
Kernel modules --->
    Other modules  --->
        <*> kmod-lunatik
```

### Build an image for the target platform

In order to build an image, execute:

```sh
make -j$(nproc)
```

> [!IMPORTANT]
> For builds on WSL, make sure to follow [Build system setup WSL](https://openwrt.org/docs/guide-developer/toolchain/wsl).

### Compile Lunatik

Once a full image build is complete, Lunatik may be recompiled by executing the following command:

```sh
make package/feeds/luainkernel/lunatik/compile
```
