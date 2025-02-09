<!-- markdownlint-disable MD013 -->
# OpenWrt package feed for [luainkernel](https://github.com/luainkernel)

The feed is comprised by the following packages:

- lunatik
- lua5.4 (as a dependency)

> [!NOTE]
> This feed has been tested only on [OpenWrt 23.05.5](https://github.com/openwrt/openwrt/releases/tag/v23.05.5).

## Feed configuration

In order to use this feed into an OpenWrt build, simply add the following line to your `feeds.conf.default`:

```sh
src-git luainkernel https://github.com/luainkernel/openwrt_feed.git^<commit-hash>
```

> [!IMPORTANT]
> Make sure to set the `<commit-hash>` that corresponds to the lunatik release you'd like to use, which is defined at [`the package Makefile`](./kernel/lunatik/Makefile).

After that, update and install the feed:

```sh
./scripts/feeds update luainkernel
./scripts/feeds install -a -p luainkernel
```

> [!NOTE]
> Refer to [OpenWrt Feeds](https://openwrt.org/docs/guide-developer/feeds) for more information.

## Build

### Configure buildroot

Setup the target platform configuration as usual but make sure to select `kmod-lunatik` under the following path:

```txt
Kernel modules --->
    Other modules  --->
        <*> kmod-lunatik
```

> [!IMPORTANT]  
> Each platform may require some extra configuration.
> Some sample configuration may be found under [`./config`](./config).

## Build the image

```sh
time make -j$(nproc) V=s
```

> [!IMPORTANT]
> For build on WSL, make sure to follow [Build system setup WSL](https://openwrt.org/docs/guide-developer/toolchain/wsl).
