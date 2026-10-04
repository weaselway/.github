# Weaselway

A GPU-accelerated GNOME desktop on WSL2, shipped as a NixOS-WSL image, in a
fullscreen window on Windows.

It is for people who have to use a Windows machine, but would much rather have
the comforts of GNOME while they do.

WSLg already puts individual Linux app windows on the Windows desktop, using a
compositor of its own. Weaselway replaces that with a whole session. GNOME's
own compositor drives a virtual display, which a small kernel module provides,
the way it would drive a monitor, and Mesa's `d3d12` Gallium driver renders on the GPU Windows exposes.
A daemon picks up each frame and hands it through shared memory to a FreeRDP
client on the Windows side, which shows it.

Since the compositor needs nothing special, the same setup runs Plasma. It is
an option in the image's configuration rather than part of the image.

Acceleration goes all the way up: Chromium runs fully GPU-accelerated, and the
GNOME session holds 60 fps at 2560x1440 on a ten-year-old laptop.

## Getting it

Download `nixos-weaselway-<version>.wsl` from the [releases] and import it:

```powershell
wsl --install --from-file nixos-weaselway-<version>.wsl --name Gnome
```

Everything is in the image: the patched mesa, the `dxgdrm` kernel module, the
`weaselwayd` daemon, the PipeWire audio bridge, and the Windows viewer itself.
The one piece outside it is a small WSLg system distro, which
`install-system-image` fetches for you. After that, `start-session` and
`start-viewer` bring up the desktop. The [weaselway] README walks through it step by step.

The system is a NixOS flake in `/etc/nixos`, so updating is
`nix flake update` plus `nixos-rebuild switch`. The patched packages come
prebuilt from [weaselway.cachix.org][cachix], and a bad update is one
`--rollback` away.

## The repos

- **[weaselway]** — the NixOS module and the image flake, and `weaselwayd`:
  the daemon that reads the compositor's frames back, serves them over RDP on
  a vsock, and turns the client's keyboard, mouse and touchpad into ordinary
  input devices.
- **[dxgdrm]** — the kernel module. It gives `d3d12` a real
  `/dev/dri/renderD128`, which WSL otherwise never creates, and the compositor
  a virtual display to drive.
- **[mesa]** — the `d3d12` Gallium driver, with the dma-buf and sync-file work
  needed to share buffers and fences on WSL, and to scan out on dxgdrm.
- **[freerdp]** — the SDL FreeRDP client on the Windows side. It ships inside
  the image; `start-viewer` runs it from there.
- **[wslg]** — a slimmed WSLg system distro that stays out of the way. WSL
  only sets up the shared memory the frames are handed over on when a system
  distro is configured.
- **[mutter]** and **[kde-kwin]** — one fix each, neither specific to
  Weaselway, kept on a branch until it is upstream. The image applies them as
  patches to the compositors nixpkgs ships.

The image is x86_64, built on nixos-26.05 and an unmodified [NixOS-WSL].

## A note on AI

AI was used heavily throughout this project. This is not meant to be a
beautiful piece of software — it is meant to solve a problem I have: I want
to be able to use GNOME on my Windows machine.

[weaselway]: https://github.com/weaselway/weaselway
[releases]: https://github.com/weaselway/weaselway/releases
[mutter]: https://github.com/weaselway/mutter
[kde-kwin]: https://github.com/weaselway/kde-kwin
[mesa]: https://github.com/weaselway/mesa
[dxgdrm]: https://github.com/weaselway/dxgdrm
[wslg]: https://github.com/weaselway/wslg
[freerdp]: https://github.com/weaselway/freerdp
[cachix]: https://weaselway.cachix.org
[NixOS-WSL]: https://github.com/nix-community/NixOS-WSL
