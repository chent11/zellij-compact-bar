# zellij-compact-bar

[zellij](https://zellij.dev)'s built-in `compact-bar` status bar with one change:
every tab title is prefixed with its number, so the bar doubles as a guide for
`GoToTab 1..9`.

```
 Zellij (work) NORMAL  ← +3  4:~/src/api  5:~/src/web  6:~/dotfile  +4 →
```

Everything else is upstream: the session name and mode on the left, and tabs
that collapse into `← +N` / `+N →` markers when they do not fit the width, so
the row never overflows. The sources are a copy of
[`default-plugins/compact-bar`](https://github.com/zellij-org/zellij/tree/v0.45.1/default-plugins/compact-bar)
at zellij v0.45.1, with the patch kept to two places in `src/main.rs` (search
for `Patch:`).

The tag names the zellij release the plugin is built against (`zellij-tile`).
From v0.46.0 on that is no longer the release the sources come from: zellij
0.46 folded its bars into one shared plugin (`default-plugins/bars`), and a
shared plugin loaded from a URL loses its first `ModeUpdate` while its
permissions are still being applied, so its bar stays empty. Until that works
upstream, this stays the standalone 0.45.1 bar, ported to the newer plugin API.
It does not have the bar features zellij added in 0.46.

## Use

Point the `compact-bar` alias at a release asset in `~/.config/zellij/config.kdl`;
the stock `compact` layout then uses it. Pick the tag that matches your zellij
version.

```kdl
plugins {
    compact-bar location="https://github.com/chent11/zellij-compact-bar/releases/download/v0.46.0/compact-bar.wasm"
}
default_layout "compact"
```

Unlike the built-in copy, a plugin loaded from a URL has to be granted
permissions, and a one-row bar cannot show the prompt. Grant them once in
zellij's permission cache (`~/.cache/zellij/permissions.kdl` on Linux,
`~/Library/Caches/org.Zellij-Contributors.Zellij/permissions.kdl` on macOS):

```kdl
"https://github.com/chent11/zellij-compact-bar/releases/download/v0.46.0/compact-bar.wasm" {
    ReadApplicationState
    ChangeApplicationState
    MessageAndLaunchOtherPlugins
    Reconfigure
}
```

`ReadApplicationState` and `ChangeApplicationState` draw the bar and switch tabs
on click; the other two are only used by upstream's tooltip feature.

## Build

```sh
rustup target add wasm32-wasip1
cargo build --release --target wasm32-wasip1
# target/wasm32-wasip1/release/compact-bar.wasm
```

Pushing a `v*` tag builds the wasm in GitHub Actions and attaches it to the
release.

## Updating to a new zellij release

1. Bump `version` and the `zellij-tile` / `zellij-tile-utils` pins in `Cargo.toml`.
2. Build and fix what the newer plugin API changed (for 0.46: `Text::new` became
   `Text::from`, client ids became `u32`, `InputMode::RenameSession` was added).
3. Run it in zellij from a `file:` location before tagging; the permission
   cache key for a file is its path, without the `file:` prefix.
4. Tag `v<zellij version>` and push the tag.

Upstream's bar now lives in `default-plugins/bars`. Moving to it means building
that whole plugin with three patches (default `role` of `compact-bar`, the
permission request with `RunActionsAsUser` added, and the tab number), and it
needs the startup problem described at the top to be solved first.

## License

MIT, as upstream zellij (see `LICENSE.md`).
