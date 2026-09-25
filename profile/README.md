# Weaselway

A GPU-accelerated GNOME desktop on Ubuntu under WSL2, in a fullscreen window
on Windows.

It is for people who have to use a Windows machine, but would much rather have
the comforts of GNOME while they do.

WSLg already puts individual Linux app windows on the Windows desktop, using a
compositor of its own. Weaselway replaces that with a whole session: mutter runs
headless and serves it over RDP on a vsock, a FreeRDP client on the Windows side
shows it, and Mesa's `d3d12` Gallium driver renders on the GPU Windows exposes.

Acceleration goes all the way up: Chromium runs fully GPU-accelerated, and the
GNOME session holds 60 fps at 2560x1440 on a ten-year-old laptop.

## The repos

Start at **[weaselway]** — the setup scripts, and a README that walks through the
whole thing step by step. The rest are the pieces it assembles:

- **[weaselway]** — setup scripts, systemd units, and the container that rebuilds
  Ubuntu's packages with the patches below.
- **[mutter]** — GNOME's compositor, serving the session over RDP on a vsock
  instead of drawing to a display.
- **[mesa]** — the `d3d12` Gallium driver, with the dma-buf and sync-file work
  needed to share buffers and fences on WSL.
- **[dxgdrm]** — a small kernel module giving `d3d12` a real
  `/dev/dri/renderD128`, which WSL otherwise never creates.
- **[wslg]** — a slimmed WSLg system distro that publishes the transport details
  and then stays out of the way.
- **[freerdp]** — the SDL FreeRDP client on the Windows side.

Everything targets Ubuntu 26.04 under WSL2. The installer uses prebuilt
artifacts by default: mesa and mutter from a PPA, the FreeRDP client and the
system distro image from GitHub releases. Only the dxgdrm module is built on
your machine, and mesa/mutter can be built locally instead.

## A note on AI

AI was used heavily throughout this project. This is not meant to be a
beautiful piece of software — it is meant to solve a problem I have: I want
to be able to use GNOME on my Windows machine.

[weaselway]: https://github.com/weaselway/weaselway
[mutter]: https://github.com/weaselway/mutter
[mesa]: https://github.com/weaselway/mesa
[dxgdrm]: https://github.com/weaselway/dxgdrm
[wslg]: https://github.com/weaselway/wslg
[freerdp]: https://github.com/weaselway/freerdp
