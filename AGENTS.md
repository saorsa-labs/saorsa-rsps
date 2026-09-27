# saorsa-rsps

Root-Scoped Provider Summaries: space-efficient summaries of the content IDs
under a root CID, built on Golomb Coded Sets, for DHT lookups and cache
admission in P2P networks.

## Build and test
- CI (`.github/workflows/rust.yml`): `cargo fmt -- --check`,
  `cargo clippy --all-targets --all-features -- -D warnings`, `cargo test --all --verbose`.
- Locally prefer `cargo nextest run` (filter by module: `cache`, `gcs`, `ttl`, `witness`, `proptest`).
- Benchmarks: `cargo bench --features bench`.

## Layout
- `src/gcs.rs` — `GolombCodedSet` / `GcsBuilder`. Golomb-Rice parameters must be
  powers of two so encode/decode is deterministic; verification enforces this.
- `src/lib.rs` — `Rsps` (root CID + epoch + GCS of child CIDs), digests for DHT
  advertisement keyed by root_cid+epoch. `Result<T>` alias over `RspsError` (thiserror).
- `src/cache.rs` — `RootAnchoredCache`: admission by RSPS membership, root depth and pledge ratio.
- `src/ttl.rs` — `TtlEngine`: TTL extended by cache hits and witness receipts, receipts bucketed in time.
- `src/witness.rs`, `src/crypto/` — VRF witness receipts and signature/VRF provider traits
  (`ed25519-dalek`, `vrf-r255`). Older docs described the witness crypto as a
  placeholder; check its soundness before relying on it for security.

## Defaults
Target FPR 5e-4; base TTL 2 h; +30 min per hit (max 12 h); +10 min per receipt
(max 2 h); receipt buckets 5 min.
