# hashconsius

Hash consing for Rust, proved correct with Verus. It's a port of
[`hashconsing`](https://crates.io/crates/hashconsing) with two storage modes:
`HConsign` keeps values in heap `Arc`s, and `BHConsign` packs them into a
fixed-capacity bump arena backed by [`bumpus`](https://crates.io/crates/bumpus).

The proved contract: a consign never holds two values with the same key view,
and two handles from the same consign have equal uids exactly when their views
are equal. Your keys implement the verified `HashKey` trait instead of
`Hash + Eq`. The crate docs in `src/lib.rs` spell out what is trusted and where
this crate departs from `hashconsing`. Read them before relying on the proofs.

## Features

The arena and the bucket index work under `no_std` with `alloc`; build with
`--no-default-features`. The default `std` feature adds heap consing and the
`demo` module. `demo::heap_smoke()` returns `4`. `demo::arena_smoke()` returns
`Result<usize, AllocErr>` and is proved to give `Ok(4)` whenever runtime sizing
and construction succeed.

## Building and verifying

You need Rust 1.98.1 (pinned in `rust-toolchain.toml`). Building and testing
work with plain cargo. Verification also needs Verus on your `PATH`.

```sh
cargo clean && cargo verus verify
cargo test
cargo build --release --all-targets
cargo build --no-default-features --target thumbv7m-none-eabi
```

Start from `cargo clean` so both crates print their `verification results`
line.

Dependencies are pinned exactly: vstd `=0.0.0-2026-10-04-0306` and bumpus
`=0.1.1`. That vstd build matches the allocator and Verus 0.2026.10.04, and
`cargo verus verify --check-toolchain` accepts the pair.

One gap remains. The command I actually want is
`cargo verus verify --fwd-verus-args-to roots -- --no-cheating`, but this Verus
version also checks trusted items imported from dependencies. It rejects
bumpus's `Layout` bridge even with root-only forwarding, so there's no way yet
to enforce zero-trust on this crate alone. Bumpus keeps its three trusted
bridges as they are.
