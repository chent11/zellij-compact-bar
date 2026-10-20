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
[`default-plugins/compact-bar`](https://github.com/zellij-org/zellij/tree/main/default-plugins/compact-bar)
at the zellij release named by the tag, with the patch kept to two places in
`src/main.rs` (search for `Patch:`).

## Use

Point the `compact-bar` alias at a release asset in `~/.config/zellij/config.kdl`;
the stock `compact` layout then uses it. Pick the tag that matches your zellij
version.

```kdl
plugins {
    compact-bar location="https://github.com/chent11/zellij-compact-bar/releases/download/v0.45.1/compact-bar.wasm"
}
default_layout "compact"
```

Unlike the built-in copy, a plugin loaded from a URL has to be granted
permissions, and a one-row bar cannot show the prompt. Grant them once in
zellij's permission cache (`~/.cache/zellij/permissions.kdl` on Linux,
`~/Library/Caches/org.Zellij-Contributors.Zellij/permissions.kdl` on macOS):

```kdl
"https://github.com/chent11/zellij-compact-bar/releases/download/v0.45.1/compact-bar.wasm" {
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

1. Replace `src/` with upstream `default-plugins/compact-bar/src` at the new tag.
2. Re-apply the two `Patch:` hunks in `src/main.rs`.
3. Bump `version` and the `zellij-tile` / `zellij-tile-utils` pins in `Cargo.toml`.
4. Build, then tag `v<zellij version>` and push the tag.

## License

MIT, as upstream zellij (see `LICENSE.md`).
