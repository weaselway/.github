# Weaselway

A GPU-accelerated GNOME desktop on WSL2, shipped as a NixOS-WSL image, in a
fullscreen window on Windows.

It is for people who have to use a Windows machine, but would much rather have
the comforts of GNOME while they do.

WSLg already puts individual Linux app windows on the Windows desktop, using a
compositor of its own. Weaselway replaces that with a whole session: mutter runs
headless and serves it over RDP on a vsock, a FreeRDP client on the Windows side
shows it, and Mesa's `d3d12` Gallium driver renders on the GPU Windows exposes.

Acceleration goes all the way up: Chromium runs fully GPU-accelerated, and the
GNOME session holds 60 fps at 2560x1440 on a ten-year-old laptop.

## Getting it

Download `nixos-weaselway-<version>.wsl` from the [releases] and import it:

```powershell
wsl --install --from-file nixos-weaselway-<version>.wsl --name Gnome
```

Everything is in the image: the patched mesa and mutter, the `dxgdrm` kernel
module, the PipeWire audio bridge, and the Windows viewer itself. The one piece
outside it is a small WSLg system distro, which `install-system-image` fetches
for you. After that, `start-gnome-shell` and `start-viewer` bring up the
desktop. The [weaselway] README walks through it step by step.

The system is a NixOS flake in `/etc/nixos`, so updating is
`nix flake update` plus `nixos-rebuild switch`. The patched packages come
prebuilt from [weaselway.cachix.org][cachix], and a bad update is one
`--rollback` away.

## The repos

- **[weaselway]** — the NixOS module and the image flake, plus the older Ubuntu
  setup scripts.
- **[mutter]** — GNOME's compositor, serving the session over RDP on a vsock
  instead of drawing to a display.
- **[mesa]** — the `d3d12` Gallium driver, with the dma-buf and sync-file work
  needed to share buffers and fences on WSL.
- **[dxgdrm]** — a small kernel module giving `d3d12` a real
  `/dev/dri/renderD128`, which WSL otherwise never creates.
- **[wslg]** — a slimmed WSLg system distro that publishes the transport details
  and then stays out of the way. WSL only sets up the shared memory mutter hands
  its frames over on when a system distro is configured.
- **[freerdp]** — the SDL FreeRDP client on the Windows side. It ships inside
  the image; `start-viewer` runs it from there.

The image is x86_64, built on nixos-26.05 and an unmodified [NixOS-WSL]. The
older Ubuntu 26.04 setup, with packages from a PPA and install scripts, is still
described in [README-ubuntu.md][ubuntu].

## A note on AI

AI was used heavily throughout this project. This is not meant to be a
beautiful piece of software — it is meant to solve a problem I have: I want
to be able to use GNOME on my Windows machine.

[weaselway]: https://github.com/weaselway/weaselway
[releases]: https://github.com/weaselway/weaselway/releases
[mutter]: https://github.com/weaselway/mutter
[mesa]: https://github.com/weaselway/mesa
[dxgdrm]: https://github.com/weaselway/dxgdrm
[wslg]: https://github.com/weaselway/wslg
[freerdp]: https://github.com/weaselway/freerdp
[cachix]: https://weaselway.cachix.org
[NixOS-WSL]: https://github.com/nix-community/NixOS-WSL
[ubuntu]: https://github.com/weaselway/weaselway/blob/main/README-ubuntu.md
