# winsnap

Save and restore Hyprland window layouts. A snapshot records every open
window's workspace, position, size, tiled/floating state and launch command.
Restore reopens anything that's missing and puts every window back, including
the dwindle split layout of tiled windows.

Built for [Omarchy](https://omarchy.org/) (Hyprland 0.56+ with Lua dispatchers).

## Install

```bash
git clone https://github.com/tqdesign/winsnap.git
ln -s "$PWD/winsnap/winsnap" ~/.local/bin/winsnap
```

Needs `python3`, `hyprctl`, `uwsm-app` and `notify-send` (all present on Omarchy).

## Use

```bash
winsnap save [name]      # save all windows (default name: "default")
winsnap restore [name]   # relaunch missing windows and put everything back
winsnap list             # list snapshots
winsnap show [name]      # print a snapshot's windows
winsnap delete <name>    # delete a snapshot
```

Keep as many layouts as you like: `winsnap save work`, `winsnap restore work`.

Snapshots are JSON in `~/.local/state/winsnap/<name>.json`. If a window's
auto-detected launch command is wrong, edit its `cmd` field.

### Keybindings

Add to `~/.config/hypr/bindings.lua`:

```lua
o.bind("SUPER + CTRL + S", "Save window layout", "winsnap save")
o.bind("SUPER + CTRL + R", "Restore window layout", "winsnap restore")
```

Check `omarchy menu keybindings --print` first and `hl.unbind(...)` any key
that's already taken.

## How restore works

1. **Match.** Each saved window is paired with an open window of the same
   class, preferring an exact title match.
2. **Launch.** Unmatched windows are started from their saved command and
   winsnap waits up to 30s for each to appear.
   - Chrome web apps (`chrome-<host>__<path>-Default`) reopen with
     `omarchy-launch-webapp`.
   - Chromium-family browsers get `--new-window`.
   - Everything else reruns the process's original command line.
3. **Rebuild.** Matched windows are parked on `special:winsnap`. For each
   workspace, winsnap finds the guillotine cuts between the saved rectangles,
   then re-inserts windows using dwindle `preselect` so the split tree matches.
   It then resizes each window to its saved size. Floating windows are moved
   and resized exactly.
4. Focus returns to the workspace that was active when you saved.

## Limits

- Restores windows, not their contents. Browsers open a fresh window and
  terminals open a new shell in the original working directory.
- Windows that aren't in the snapshot are left in place and can squeeze the
  restored layout on their workspace.
- Fullscreen and pinned state are recorded but not reapplied.
- Tuned for the dwindle layout. Other layouts get windows on the right
  workspaces, but the split structure may differ.

## License

MIT. See [LICENSE](LICENSE).
