Source:
https://github.com/0atman/noboilerplate/blob/main/scripts/06-build-your-rust-lightsaber.md
https://m.youtube.com/watch?v=ifaLk5v3W90&list=PLZaoyhMXgBzoM9bfb5pyUOT3zjnaDdSEP&index=6&pp=iAQB

- Test Runner: [Bacon](https://crates.io/crates/bacon)
- Async Runtime: [TOKIO](https://crates.io/crates/tokio)
- better runtime exceptions: 
	- Eyre
	- [Color-Eyre](https://crates.io/crates/color-eyre)
- Better logging: [tracing](https://crates.io/crates/tracing)
- http requests: [Reqwest](https://crates.io/crates/reqwest)
- Easy parallelism: [Rayon](https://crates.io/crates/rayon)
- [AWS SDK](https://crates.io/crates/aws-sdk-rust)
- Command Line Parser: [Clap](https://crates.io/crates/clap)
- SQL Framework: [SQLX](https://crates.io/crates/sqlx)
- Datetimes: [Chrono](https://crates.io/crates/chrono)


## Clippy Sensible Defaults
```
cargo clippy --fix -- \
-W clippy::pedantic \
-W clippy::nursery \
-W clippy::unwrap_used \
-W clippy::expect_used
```

### Tests

Source: https://m.youtube.com/watch?v=JIvKgSyvtxI
- DocTests
- Proptest - deterministic tests
- sqlx - useful for integration testing (including mocking the database)
- pact_consumer - integration testing