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
  into the built theme (gated on the tweak); and, for the libadwaita `-l`
  link path, symlink `windows-assets/` into `~/.config/gtk-4.0` (with matching
  cleanup in `uninstall_link`) so the buttons' relative PNG URLs resolve
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

### 3. Nautilus (GTK4 / libadwaita) sidebar and selection restyle

Reworks Nautilus selection styling so the whole window shares one calm,
accent-free highlight and the selected sidebar row visually merges into the
main file view. All in `src/sass/gtk/apps/_gnome-4.0.scss`.

Three tunable colours, grouped at the top of the Nautilus section so they are
easy to play with (raise each to go darker):

- `$nautilus-sidebar-bg: mix($base, $sidebg, 25%)` - left-pane background +
  selected folders (the light highlight). Higher 2nd mix number = darker.
  Reused for the sidebar/left-pane background, the main-view list and grid
  selection, and the floating "N selected" bar
- `$nautilus-view-bg: darken($base, 3%)` - right pane (main file view)
  background, the column/sort headers, and the selected sidebar row's bleed
  fill (so the row still merges seamlessly into the darker view)
- `$nautilus-odd-row-bg: gtkalpha(black, 0.12)` - list-view odd-row zebra
  stripe. A black alpha overlay (was a faint `currentColor` lightening);
  raise the alpha to darken the stripe

What else it does:

- Unified corner radius `$nautilus-inv-radius: 10px` across the sidebar
  selection and the main-view list/grid items
- Selected sidebar row runs to the right edge (`margin-right: -$space-size`)
  with squared right corners, so it bleeds into the main view. Its right
  corners are concave "opening" cutouts and its left corners convex, composed
  as a multi-layer `radial-gradient` + `linear-gradient` fill (GTK4 has no
  4-value `background-position`, so corners are placed with `left top` /
  `left bottom`)
- Sidebar rows overlap by the radius with compensating padding, so the
  concave corners can curve into the neighbour row without changing spacing
- The selection fill is pinned on hover/focus so the selected row stays
  stable; only unselected rows react to hover
- Column/sort headers get `$nautilus-view-bg` to override Graphite's slightly
  lighter default header fill

Gotchas discovered while building this:

- Specificity: the rules lean on extra `.nautilus-window` / `listview.view` /
  `.sidebar-pane` prefixes to outrank Graphite's generic and `%sidebar-row`
  rules without `!important`
- Inner node structure matters: the overlap padding targets
  `row > revealer.sidebar-revealer > box` per `nautilus-sidebar-row.ui`

Commits: `07a74ea`, `f70386e`, `6c2ed69`, `354e2e8`, `9fdc731` (plus later
darker-tones + tunable-colours and column-header tweaks).

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
ln -sfn "$G/gtk-4.0/windows-assets" ~/.config/gtk-4.0/windows-assets  # WhiteSur buttons
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
