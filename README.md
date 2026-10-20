# hashconsius

Verified heap and fixed-capacity arena hash consing, ported from `hashconsing`.
See the crate documentation in `src/lib.rs` for the contract, trust boundary,
and deliberate compatibility differences.

Requires Rust 1.98.1 and Verus on PATH; `verus-bumpalo` is pinned from git.
The arena and bucket index support `no_std` with `alloc` (`--no-default-features`);
the default `std` feature also exposes heap consing and the demos.
The vstd pin matches the current allocator, not the newer verifier (see the
trust inventory in `src/lib.rs`). `demo::arena_smoke()` returns
`Result<usize, AllocErr>`, proving `Ok(4)` when runtime sizing and construction
succeed. `demo::heap_smoke()` returns `4`.

```sh
cargo clean && cargo verus verify
cargo test
cargo build --release --all-targets
cargo build --no-default-features --target thumbv7m-none-eabi
```

The clean run makes both crates' `verification results` lines visible.
The intended stricter command is
`cargo verus verify --fwd-verus-args-to roots -- --no-cheating`, but this Verus
version also checks imported trusted items and rejects bumpus's `Layout` bridge
even with root-only forwarding; it does not currently enforce a local-only
zero-trust policy. Bumpus retains its three trusted bridges unchanged.
`cargo verus verify --check-toolchain` separately rejects the verifier/vstd pair;
ordinary verification does not check that compatibility by default.
