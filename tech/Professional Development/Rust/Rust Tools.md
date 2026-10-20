

- [Bacon](https://crates.io/crates/bacon) - Build, clippy, test, run watcher.
- [TOKIO](https://crates.io/crates/tokio):  Async Runtime:
- better runtime exceptions: 
	- Eyre
	- [Color-Eyre](https://crates.io/crates/color-eyre)
-  [tracing](https://crates.io/crates/tracing) - Async-native logging
- [Reqwest](https://crates.io/crates/reqwest):  http requests:
- [Rayon](https://crates.io/crates/rayon):  Easy parallelism:
- [AWS SDK](https://crates.io/crates/aws-sdk-rust)
- [Clap](https://crates.io/crates/clap): Command Line Parser:
-  [SQLX](https://crates.io/crates/sqlx): SQL Framework:
- [Chrono](https://crates.io/crates/chrono): Datetimes
- iRust: Fully-featured REPL, debug, asm inspection.
- rstest: testing framework with fixtures
- Nextest: Next generation test runner
- Cargo Generate: quickly scaffold larger frameworks such as a web app
- watchexec: execute commands in response to file modifications
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
# cargo clippy - use this instead of relying on IDE
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
- Follow the advice of cargo clippy. Remember, that the first error in your file is not always the one to fix - the first error in the COMPILER output usually is.

Source:
https://github.com/0atman/noboilerplate/blob/main/scripts/06-build-your-rust-lightsaber.md
https://www.namtao.com/rust-toolkit-2026/