# OpenWrt Tailscale MIPSel

Minimal Tailscale build and deploy scripts for OpenWrt routers with very limited flash, focused only on `mipsel_24kc` devices.

This repository exists because the official Tailscale packages and many community builds are still too large for older 8MB flash routers, even when the runtime feature set needed is only:

- normal node connectivity
- subnet route advertisement
- exit node advertisement

The build here keeps those capabilities and strips a large set of optional features to make the binary small enough to fit on constrained OpenWrt systems.

## Scope

Target:

- `mipsel_24kc`

Validated environment:

- OpenWrt `24.10.x`
- `mipsel_24kc`
- packed binary size around `3.6MB`
- observed `tailscaled` memory usage around `27MB` after startup on the validated router

## What This Repo Provides

- `build.sh`
  Builds a stripped Tailscale multicall binary from upstream source and can optionally deploy it over SSH.

The built binary is a single executable that behaves as both:

- `tailscaled`
- `tailscale`

## Why Standalone

This repository started from experiments based on existing community projects, but it is kept standalone because the validated target, documentation, and build defaults are centered on older OpenWrt MIPS hardware rather than GL.iNet-specific ARM devices.

## Requirements

Build host:

- `git`
- `go`
- `ssh`
- `ssh-keygen`
- `sftp`

Optional:

- `ssh-copy-id`
- `upx`

Router:

- OpenWrt with `kmod-tun`
- SSH access as `root`

## Usage

Build:

```bash
chmod +x build.sh
./build.sh
```

The script will:

1. ask which upstream Tailscale ref to build
2. clone upstream `tailscale/tailscale`
3. build a stripped multicall binary
4. compress it with `upx` if available
5. optionally deploy it to the router over SSH

## GitHub Releases

This repository can publish release artifacts with GitHub Actions.

The workflow:

- checks for the latest stable upstream Tailscale tag every Monday
- can also be triggered manually from the Actions tab
- builds the stripped `mipsel_24kc` binary
- compresses it with `upx`
- publishes a GitHub release asset named like:

```text
tailscale-openwrt-mipsel-<version>.combined
```

The release asset is the packed multicall binary, not an `.ipk` package.

## MIPS Build Details

The validated build uses:

```text
GOOS=linux
GOARCH=mipsle
GOMIPS=softfloat
```

It intentionally keeps support for:

- subnet routes
- exit node advertisement
- router/firewall integration

It strips many optional features such as:

- desktop integrations
- debug tooling
- taildrop
- web UI
- Kubernetes/cloud extras

It does not intentionally omit Unix socket identity support, because doing so breaks LocalAPI permissions for `tailscale` talking to `tailscaled` on OpenWrt.

## Operational Notes

- The packed binary may fit on small flash, but runtime RAM is still tight on 128MB routers.
- In validation, `tailscaled` used about `27MB` RSS after startup, and total usable memory headroom was still limited.
- If memory pressure is high, try running `tailscaled` with `GOGC=10`.
- `/tmp` is RAM-backed and useful for testing state files, but not persistent.
- `/var/lib/tailscale` or another persistent path is required if you want login state to survive reboot.

## Contributors And Credits

This repository builds on ideas proven by earlier community work:

- [adoreste/gl-inet-tailscale-updater](https://github.com/adoreste/gl-inet-tailscale-updater)
  Original minimal build-and-deploy script structure that this repository was adapted from.
- [masarykadam/openwrt-tailscale-minimal](https://github.com/masarykadam/openwrt-tailscale-minimal)
  Small-package OpenWrt release approach and packaging inspiration.
- [tailscale/tailscale](https://github.com/tailscale/tailscale)
  Upstream source code.

Credits:

- adoreste
- Adam Masaryk (`masarykadam`)
- Tailscale contributors

## License

Upstream Tailscale source remains under its own license. This repository contains build and deployment scripts only.
