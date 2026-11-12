# Graphite-gtk-theme - fork notes

Personal fork of [vinceliuice/Graphite-gtk-theme](https://github.com/vinceliuice/Graphite-gtk-theme).

- `origin`   https://github.com/herveblondeau/Graphite-gtk-theme
- `upstream` https://github.com/vinceliuice/Graphite-gtk-theme

This file documents how this fork differs from upstream, and the local
system setup used to install it.

## Divergence from upstream

### 1. New `whitesur` tweak - macOS traffic-light window buttons

Replaces Graphite's circular window control buttons (close / minimize /
maximize) with WhiteSur's macOS-style traffic-light PNG buttons, for both
GTK3 and GTK4 / libadwaita.

Enable it with:

```sh
./install.sh --tweaks whitesur
```

It combines with every other tweak (nord, rimless, darker, dracula, ...).

Files changed:

- `src/sass/_tweaks.scss` - new `$window_button` variable (`graphite` | `whitesur`)
- `src/sass/gtk/_common-3.0.scss` - `@if $window_button == 'whitesur'` branch
  styling `button.titlebutton` (GTK3)
- `src/sass/gtk/_common-4.0.scss` - same branch for `windowcontrols > button` (GTK4)
- `install.sh` - parse `whitesur`, flip the variable, copy `windows-assets/`
  into the built theme (gated on the tweak)
- `src/assets/gtk/windows-assets/` - WhiteSur button PNGs (light + dark, @1 / @2)
- `README.md` - documents the tweak

Design notes / gotchas discovered while building this:

- The WhiteSur branch is dark/light aware via Graphite's existing
  `$asset-suffix` (`-dark` for the dark variant, empty for light)
- It does NOT use `!important`: GTK4 silently drops any declaration with
  `!important` as junk. The branch replaces Graphite's native rules at the
  Sass level instead, so it wins by source order with zero parse errors
- The close button explicitly resets `background-color` so Graphite's native
  `.close:hover` / `.close:active` white circle does not bleed through

### 2. Removed the bundled `gnome-theme-switcher` app

Upstream installs a small GTK4 theme-switcher GUI into `~/.local/bin` and
`~/.local/share/applications` on every run. This fork removes that feature.

Files changed:

- `install.sh` - removed `install_app`, its call, the `-u app` uninstall case,
  and the unused `APP_DIR` / `BIN_DIR` variables
- deleted `other/gnome-theme-switcher/`

## Local install (reproduce this machine's look)

Dependencies (Fedora):

```sh
sudo dnf install gtk-murrine-engine gnome-themes-extra sassc
```

Build and install the exact variant in use (Nord + rimless + WhiteSur buttons,
dark, with libadwaita/GTK4 linking):

```sh
./install.sh -c dark --tweaks nord rimless whitesur -l
```

> If the repo path contains spaces, run the installer through a space-free
> symlink (upstream `install.sh` does not quote `$SRC_DIR`):
> `ln -sfn "$PWD" /tmp/graphite-src && cd /tmp/graphite-src && ./install.sh ...`

After `-l`, the installer links the **light** folder into `~/.config/gtk-4.0`.
For a dark libadwaita look, re-point the links at the dark folder:

```sh
G=~/.themes/Graphite-Dark-nord
ln -sfn "$G/gtk-4.0/gtk.css"      ~/.config/gtk-4.0/gtk.css
ln -sfn "$G/gtk-4.0/gtk-dark.css" ~/.config/gtk-4.0/gtk-dark.css
ln -sfn "$G/gtk-4.0/assets"       ~/.config/gtk-4.0/assets
```

Activate the theme:

```sh
gsettings set org.gnome.desktop.interface gtk-theme    "Graphite-Dark-nord"
gsettings set org.gnome.desktop.wm.preferences theme   "Graphite-Dark-nord"
gsettings set org.gnome.desktop.interface color-scheme "prefer-dark"
```

GTK4 / libadwaita apps do not live-reload: close and reopen them to see changes.

## What this theme does and does not cover

- Native GTK3 / GTK4 apps: themed
- Flatpak GTK apps: need the theme in `~/.local/share/themes` plus a flatpak
  override and a matching `org.gtk.Gtk3theme.*` extension (see `pakitheme`)
- Chromium / Electron (Brave, Zen, Bitwarden, Bruno), Flutter (LocalSend),
  Java (Android Studio, DBeaver): not themeable by a GTK theme; they only
  follow the dark / light `color-scheme`

## Syncing upstream changes later

```sh
git fetch upstream
git rebase upstream/main
```
