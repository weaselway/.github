# Weaselway

A GPU-accelerated Linux desktop on WSL2, shown in a window on Windows and
distributed as a NixOS-WSL image.

Weaselway is for people who have to work on a Windows machine but would rather
use a Linux desktop while they do.

WSLg puts individual Linux application windows on the Windows desktop, using a
compositor of its own. Weaselway runs a complete session instead. An unmodified
Wayland compositor drives a virtual display, which a small kernel module
provides, the same way it would drive a monitor. Mesa's `d3d12` Gallium driver
renders on the GPU that Windows exposes. A daemon reads each frame and passes
it through shared memory to a FreeRDP client on the Windows side.

Any compositor with a KMS backend can run this way. The image includes GNOME,
Plasma is an option in its configuration, and sway, Weston, Hyprland and
others start through a session script.

Applications are accelerated as well: Chromium runs fully GPU-accelerated, and
a GNOME session holds 60 fps at 2560x1440 on a ten-year-old laptop.

## Getting it

Download `nixos-weaselway-<version>.wsl` from the [releases] and import it:

```powershell
wsl --install --from-file nixos-weaselway-<version>.wsl --name Weaselway
```

The image contains the patched Mesa, the `dxgdrm` kernel module, the
`weaselwayd` daemon, the audio configuration and the Windows viewer. The only
separate download is a small WSLg system distro, which
`ww-install-system-image` fetches. After that, `ww-start-session` and
`ww-start-viewer` bring up the desktop. The [weaselway] README has the
step-by-step instructions.

The system is a NixOS flake in `/etc/nixos`. To update it, run
`nix flake update` and `nixos-rebuild switch`. The patched packages are
prebuilt on [weaselway.cachix.org][cachix], and
`nixos-rebuild switch --rollback` undoes an update.

## Repositories

- **[weaselway]**: the NixOS module, the image flake and `weaselwayd`, the
  daemon that reads the compositor's frames back, serves them over RDP on a
  vsock, and turns the client's keyboard, mouse and touchpad into ordinary
  input devices.
- **[dxgdrm]**: the kernel module. It gives `d3d12` a real
  `/dev/dri/renderD128`, which WSL does not create, and gives the compositor a
  virtual display to drive.
- **[mesa]**: the `d3d12` Gallium driver, with the dma-buf and sync-file
  changes needed to share buffers and fences on WSL and to scan out on dxgdrm.
- **[freerdp]**: the SDL FreeRDP client for Windows. It is part of the image,
  and `ww-start-viewer` runs it from there.
- **[wslg]**: a reduced WSLg system distro. WSL sets up the shared memory used
  for the frames only when a system distro is configured.
- **[mutter]** and **[kde-kwin]**: one fix each, neither specific to
  Weaselway, kept on a branch until it is merged upstream. The image applies
  them as patches to the compositors from nixpkgs.

The image is x86_64 and is built on nixos-26.05 with an unmodified
[NixOS-WSL].

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
