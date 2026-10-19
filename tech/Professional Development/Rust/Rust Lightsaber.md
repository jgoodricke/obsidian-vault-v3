

- Test Runner: [Bacon](https://crates.io/crates/bacon) - Build, clippy, test, run watcher.
- Async Runtime: [TOKIO](https://crates.io/crates/tokio)
- better runtime exceptions: 
	- Eyre
	- [Color-Eyre](https://crates.io/crates/color-eyre)
- Better logging: [tracing](https://crates.io/crates/tracing) - Async-native logging
- http requests: [Reqwest](https://crates.io/crates/reqwest)
- Easy parallelism: [Rayon](https://crates.io/crates/rayon)
- [AWS SDK](https://crates.io/crates/aws-sdk-rust)
- Command Line Parser: [Clap](https://crates.io/crates/clap)
- SQL Framework: [SQLX](https://crates.io/crates/sqlx)
- Datetimes: [Chrono](https://crates.io/crates/chrono)
- iRust: Fully-featured REPL, debug, asm inspection.
- rstest: testing framework with fixtures
- const_panic: string formatting for asserts in a const format
- tailcall: adss tail call recursion.

- Poem-openapi: Fast, correct, and ergonomic REST builder
- Serde: Data serialisation
- CARGO-SEEK: A TUI for finding rust packages.


## Clippy Sensible Defaults (OLD)
```
cargo clippy --fix -- \
-W clippy::pedantic \
-W clippy::nursery \
-W clippy::unwrap_used \
-W clippy::expect_used
```

## Clippy Config
### cargo.toml
```toml
[lints.clippy]
pedantic = { level = "deny", priority = -1 }   # UM, ACTUALLY
nursery = { level = "deny", priority = -1 }    # BETA LINTS
# DENY PANICS
unwrap_used = "deny"
expect_used = "deny"
indexing_slicing = "deny"
arithmetic_side_effects = "deny"
unreachable = "deny"
unimplemented = "deny"
unchecked_time_subtraction = "deny"
todo = "deny"
string_slice = "deny"
panic_in_result_fn = "deny"
panic = "deny"
exit = "deny"
as_conversions = "deny"
```
### clippy.toml
```toml
allow-unwrap-in-tests = true
allow-expect-in-tests = true
allow-panic-in-tests = true
allow-indexing-slicing-in-tests = true
```


### Linting Rules
```toml
# Linting Superpowers
[lints.rust]
unsafe_code = "forbid"

[lints.clippy]
enum_glob_use = "deny"
pedantic = "deny"
nursery = "deny"
unwrap_used = "deny"

# Build Size Optimisation
[profile.release]
opt-level = 'z'     # Optimise for size.
lto = true          # Enable Link Time Optimisation
codegen-units = 1   # Reduced to increase optimisations.
panic = 'abort'     # Abort on panic
strip = "symbols"   # Strip symbols from binary
```


### Useful Commands
```Bash
# cargo clippy
bacon clippy


```
### Tests

Source: https://m.youtube.com/watch?v=JIvKgSyvtxI
- DocTests
- Proptest - deterministic tests
- sqlx - useful for integration testing (including mocking the database)
- pact_consumer - integration testing

## Tips
- Use const functions wherever possible. These work like pure functions (kind of).
- use Compiler-Driven development (similar to Type-Driven Development)

Source:
https://github.com/0atman/noboilerplate/blob/main/scripts/06-build-your-rust-lightsaber.md
http://namtao.com/rust-toolkit-2026/