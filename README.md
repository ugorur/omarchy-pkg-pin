# omarchy-pkg-pin

Pin individual packages to a different [Omarchy](https://omarchy.org/)
channel than the rest of the system.

Omarchy's own channel (`omarchy channel set`) is system-wide. Pacman has no
apt-style pin file. Extra repos named `extra-rc` also do not work: pacman
would fetch `extra-rc.db`, but Omarchy mirrors still publish `extra.db`.

This tool keeps a user pin list and, on every channel change / update:

1. Puts those packages in `IgnorePkg` so the system channel will not upgrade
   them.
2. After `omarchy update`, downloads each pinned package from the channel
   mirror and installs it with `pacman -U`.

## Install

```bash
git clone https://github.com/ugorur/omarchy-pkg-pin.git
cd omarchy-pkg-pin
install -Dm755 omarchy-pkg-pin ~/.local/bin/omarchy-pkg-pin
omarchy-pkg-pin setup
```

`setup` writes two Omarchy hooks so pins survive `omarchy channel set` and
`omarchy update`:

- `~/.config/omarchy/hooks/pre-refresh-pacman.d/pkg-pin`
- `~/.config/omarchy/hooks/post-update.d/pkg-pin`

`~/.local/bin` should already be on `PATH` on Omarchy.

## Use

```bash
omarchy-pkg-pin add telegram-desktop rc
omarchy-pkg-pin add hyprland edge
omarchy-pkg-pin list
omarchy-pkg-pin status
omarchy-pkg-pin remove hyprland
omarchy-pkg-pin sync
```

Pin file: `~/.config/omarchy/pkg-pin/pins`

```
# Format: <package>  <stable|rc|edge>
telegram-desktop  rc
```

`add` / `sync` / `apply-config` need sudo because they edit
`/etc/pacman.conf`.

## How it works

```
pins: telegram-desktop rc
        │
        ▼
omarchy update / pacman -Syu
        │  IgnorePkg skips telegram-desktop
        ▼
post-update hook → omarchy-pkg-pin sync
        │  fetch extra.db from rc-mirror.omarchy.org
        │  download telegram-desktop-*.pkg.tar.zst
        ▼
pacman -U  (install that exact package)
```

`pre-refresh-pacman` runs after Omarchy overwrites `/etc/pacman.conf` (on
`omarchy channel set`) and writes the `IgnorePkg` block back before the
upgrade.

## Limits

- Mixing channels can still pull mismatched *dependencies* when the pinned
  package needs a newer library than the system channel has. Prefer pinning
  leaf packages, not `glibc` / `systemd` / `linux` / `omarchy`.
- Omarchy's own packages on `rc` share the `edge` package repo
  (`pkgs.omarchy.org/edge`). Arch packages on `rc` vs `edge` still differ
  (`rc-mirror.omarchy.org` vs `mirror.omarchy.org`).
- A pin to the same channel the system is already on is a no-op for
  `IgnorePkg`; the package follows the system channel.

## License

MIT. See [LICENSE](LICENSE).
