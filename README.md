# hashconsius

Hash consing for Rust, proved correct with Verus. It's a port of
[`hashconsing`](https://crates.io/crates/hashconsing) with two storage modes:
`HConsign` keeps values in heap `Arc`s, and `BHConsign` packs them into a
fixed-capacity bump arena backed by [`bumpus`](https://crates.io/crates/bumpus).

The proved contract: a consign never holds two values with the same key view,
and two handles from the same consign have equal uids exactly when their views
are equal. Keys implement the verified `HashKey` trait instead of `Hash + Eq`.
The crate docs in `src/lib.rs` list the trusted base and every departure from
`hashconsing`.

## Features

The arena and the bucket index work under `no_std` with `alloc`; build with
`--no-default-features`. The default `std` feature adds heap consing and the
`demo` module. `demo::heap_smoke()` returns `4`. `demo::arena_smoke()` returns
`Result<usize, AllocErr>` and is proved to give `Ok(4)` whenever runtime sizing
and construction succeed.

## Building and verifying

Rust 1.98.1 is pinned in `rust-toolchain.toml`. Building and testing need only
cargo; verification also needs Verus on `PATH`.

```sh
cargo clean && cargo verus verify
cargo test
cargo build --release --all-targets
cargo build --no-default-features --target thumbv7m-none-eabi
```

Starting from `cargo clean` makes every verified crate print its
`verification results` line.

Dependencies are pinned exactly: vstd `=0.0.0-2026-10-04-0306` and bumpus
`=0.1.1`. That vstd build matches the allocator and Verus 0.2026.10.04, and
`cargo verus verify --check-toolchain` accepts the pair.

## Trusted code

This crate has no `assume`, `admit`, or local trusted specifications, and
Verus enforces that:

```sh
cargo verus verify --fwd-verus-args-to roots -- --no-cheating
```

The flag applies only to this crate. Forwarding it everywhere
(`cargo verus verify -- --no-cheating`) fails inside vstd, which uses `assume`
itself. Dependencies stay trusted as written; bumpus contributes three trusted
bridges: `try_allocate`, `slice_ref`, and the `Layout` size/alignment
specifications.
