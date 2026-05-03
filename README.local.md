# abtop (local patch)

Fork of [graykode/abtop](https://github.com/graykode/abtop) at tag **v0.4.0** with one local patch on branch `mac-claude-detection`.

## Why this fork exists

Upstream abtop's `cmd_has_binary("claude")` only matches the binary basename. Claude Code 2.x ships an autoupdater layout where the running executable is named after its version (e.g. `~/.local/share/claude/versions/2.1.121`), not `claude`. The result: on macOS, `ps -ef` lists the version path, abtop's matcher returns false, and **no Claude sessions show up in the session list**.

The patch extends `cmd_has_binary` to also match `<...>/<name>/versions/<filename>` paths, restoring detection without the false-positive risk of a generic substring match.

A second pass at adding macOS env discovery (`KERN_PROCARGS2`/`ps eww`) was reverted: macOS truncates env output to ~120 chars for non-root callers, so the read is unreliable. abtop's existing `libproc` open-FD inspection in `discover_active_session_paths` partially compensates when env is absent.

## Files changed

- `src/collector/process.rs` — `cmd_has_binary`: add path-segment match for `<name>/versions/<file>`. Tests added.
- `src/collector/claude.rs` — comment update on the non-Linux `read_env_var_from_proc` stub explaining the macOS limitation.

## Install

```sh
cd ~/workspace/abtop-patched
cargo install --path .
```

Installs to `~/.cargo/bin/abtop`, replacing whatever's there.

## Do NOT run `cargo install abtop`

That fetches upstream from crates.io and reverts the patch. To reinstall the local fork, always use `cargo install --path .` from this directory.

## Updating against upstream

```sh
cd ~/workspace/abtop-patched
git fetch origin
git rebase origin/main mac-claude-detection   # or the new tag
cargo test --release
cargo install --path .
```

## Runtime requirement

For `abtop` to see Claude sessions in non-default profile dirs (e.g. `~/.config/idealab/claude-profile-official/`), launch it from a shell where `CLAUDE_CONFIG_DIR` is exported. `~/.zshrc` already sets this. Spotlight / GUI launchers strip env and will partially miss profile sessions — abtop's libproc fallback finds the actively-held ones but not idle ones.

## Upstream

If the patch is accepted upstream, this fork can be deleted and replaced with the published `abtop >= <version>`.
