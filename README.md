<!-- markdownlint-disable MD013 -->
# OpenWrt package feed for [luainkernel](https://github.com/luainkernel)

The feed is comprised by the following packages:

- lunatik
- lua5.5, as a dependency for executing Lunatik user-space utilities

## Feed configuration

In order to use this feed into an OpenWrt build, add the following line to your `feeds.conf.default`:

```sh
src-git luainkernel https://github.com/luainkernel/openwrt_feed.git;lunatik-5
```

After that, update and install the feed:

```sh
./scripts/feeds update luainkernel
./scripts/feeds install -a -p luainkernel
```

> [!NOTE]
> Refer to [OpenWrt Feeds](https://openwrt.org/docs/guide-developer/feeds) for more information about how feeds work.

## Build instructions

> [!NOTE]
> The build configuration runs Lua 5.5 on the build machine.
> The feed builds it as a host package (`lua5.5/host`), so no Lua installation is required.

### Configure buildroot

Setup the target platform configuration as usual but make sure to select `kmod-lunatik` under the following path:

```txt
Kernel modules --->
    Lunatik  --->
        <*> kmod-lunatik................. Lunatik Lua-in-kernel runtime (core)
        <*> kmod-lunatik-byteorder....... Lunatik byteorder bindings
        <*> kmod-lunatik-completion...... Lunatik completion bindings
        ...
```

> [!NOTE]
> Each binding is a separate `kmod-lunatik-<name>` package under the same `Lunatik` submenu. Select only the bindings you need; `kmod-lunatik` (the core runtime) is required by all of them.

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
