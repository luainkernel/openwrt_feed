<!-- markdownlint-disable MD013 -->
# OpenWrt package feed for [luainkernel](https://github.com/luainkernel)

The feed is comprised by the following packages:

- lunatik
- lua5.4, as a dependency for executing Lunatik user-space utilities

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
> Refer to [OpenWrt Feeds](https://openwrt.org/docs/guide-developer/feeds) for more information about how feeds work.

## Build instructions

> [!IMPORTANT]
> Starting on Lunatik 4.0, Lua 5.4 is required for build configuration on the build machine.
> Make sure to have it installed.
>
> For example, on Debian/Ubuntu machines, run `sudo apt-get install lua5.4`.

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
