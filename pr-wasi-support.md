# PR draft: find: add WASI support for -ls output

Status (June 2026): after rebasing onto current upstream main, almost all of the original wasm32-wasi work has landed upstream — findutils gained generic `not(any(unix, windows))` fallbacks covering user/group matching, name matching, and xargs sizing, and uucore 0.9.0 ships its own WASI support. The remaining PR content is the single commit below. The branch's second commit (vendored uucore with a `feature(wasip2)` crate attribute) exists only for local wasm32-wasip2 builds and is not part of the PR.

## Summary

This PR adds a WASI implementation of the `-ls` output. On `wasm32-wasi` targets, `find` already compiles and runs against uucore 0.9.0, but `-ls` falls back to the generic non-Unix printer, which has no metadata access. The WASI implementation prints real file size and modification time; fields WASI does not provide are fixed placeholders (permissions display as `----------`, inode and user/group as zeros).

## Changes

All changes are in `src/find/matchers/ls.rs`:

- Add a `#[cfg(target_os = "wasi")]` `print` implementation that reads size and mtime from `fs::Metadata` and formats the same column layout as the Unix printer.
- Exclude WASI from the generic fallback by changing its gate from `#[cfg(not(unix))]` to `#[cfg(not(any(unix, target_os = "wasi")))]`, so exactly one implementation matches per platform.

## Build and verification

The commit builds standalone against crates.io uucore 0.9.0 on stable Rust:

```
cargo build --target wasm32-wasip1 --release
```

`CC`/`CFLAGS` must point at a wasi-sdk sysroot for the oniguruma C library used by the regex dependency. Verified with `find / -name` pattern queries under wasmtime.
