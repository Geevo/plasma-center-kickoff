# plasma-center-kickoff

Make the KDE Plasma 6 Application Launcher (Kickoff) open **centred on its panel**, Windows 11 style, instead of over the launcher button.

Plasma has no option for this ([bug 485293](https://bugs.kde.org/show_bug.cgi?id=485293), [bug 499472](https://bugs.kde.org/show_bug.cgi?id=499472), [bug 518337](https://bugs.kde.org/show_bug.cgi?id=518337)). It turns out to be a one-line change.

## How it works

Plasma positions every panel widget's popup in
`/usr/share/plasma/shells/org.kde.plasma.desktop/contents/applet/CompactApplet.qml`,
anchoring it to the widget's button:

```qml
visualParent: root.compactRepresentation
```

libplasma then centres the popup on that anchor and keeps it inside the panel. This script changes the anchor to the whole panel, for Kickoff only:

```qml
visualParent: Plasmoid.pluginName === "org.kde.plasma.kickoff" ? (root.Window.window?.contentItem ?? root.compactRepresentation) : root.compactRepresentation
```

Plasma's own positioning code then centres the menu on the panel. So it follows panel length, alignment, floating margins, screen size and vertical panels with nothing to configure. Every other widget (system tray, clock, etc.) still opens over its button.

## Install

No clone needed, with curl:

```bash
curl -fsSL --create-dirs -o ~/.local/bin/plasma-center-kickoff https://raw.githubusercontent.com/Geevo/plasma-center-kickoff/main/plasma-center-kickoff && chmod +x ~/.local/bin/plasma-center-kickoff
```

or wget:

```bash
mkdir -p ~/.local/bin && wget -qO ~/.local/bin/plasma-center-kickoff https://raw.githubusercontent.com/Geevo/plasma-center-kickoff/main/plasma-center-kickoff && chmod +x ~/.local/bin/plasma-center-kickoff
```

Or from a clone:

```bash
install -Dm755 plasma-center-kickoff ~/.local/bin/plasma-center-kickoff
```

Then run `plasma-center-kickoff --set`. It's a short script, so feel free to read it before running it with sudo.

## Usage

```bash
plasma-center-kickoff          # show state: SET / NOT SET / UNKNOWN
plasma-center-kickoff --set    # apply the patch (asks for sudo, restarts plasmashell)
plasma-center-kickoff --unset  # restore stock behaviour
```

Exit codes for the status check: `0` set, `1` not set, `2` unknown.

## After Plasma updates

The patched file belongs to your distro's Plasma package (`plasma-desktop` on Fedora), so an update puts the stock line back. Run the script after updating:

```bash
plasma-center-kickoff --set
```

If it reports **UNKNOWN**, Plasma has changed that line upstream. The script won't touch the file until the patch is updated.

## Notes

- The edit is in place, keeping the file's owner, permissions and SELinux label.
- Before restarting plasmashell the script syncs `WAYLAND_DISPLAY` into the systemd user session. A stale value there makes plasmashell fall back to X11 and the panel disappears.
- To centre the classic Application Menu (Kicker) as well, change the condition to `["org.kde.plasma.kickoff", "org.kde.plasma.kicker"].includes(Plasmoid.pluginName)` in both the script and the patched file.

## Tested on

- Fedora 44 KDE, Plasma 6.7.5, Wayland

The file path is the same on other distros. Reports from other setups welcome.

## License

[MIT](LICENSE)
