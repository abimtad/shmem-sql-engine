# shmem-sql-engine

An embedded SQL database written in Rust. Several programs on the same machine
can open the same database file at once: they share it through memory, with no
server in between. It aims to behave like SQLite (same SQL, can read and write
SQLite files, can stand in for `libsqlite3`) while letting readers keep working
while someone else writes.

This repository is a copy of [MPEdb](https://github.com/punnerud/mpedb) by
Morten Punnerud-Engelstad, published under a different name. The engine, its
crates and its file names still use the original `mpedb` name. It is not a
server database, and it is not a networked replacement for PostgreSQL.

## How to run it

You need a Rust toolchain ([rustup](https://rustup.rs)). From the repo root:

```sh
cargo build --release            # build everything
cargo test --workspace           # run the test suite
cargo run -p mpedb-cli -- --help # the `mpedb` command-line tool
```

The SQLite drop-in library (a `.so` that programs using SQLite can load
instead of the real one):

```sh
cargo build --release --manifest-path crates/mpedb-capi/Cargo.toml
LD_PRELOAD=$PWD/crates/mpedb-capi/target/release/libmpedb_sqlite3.so python3 app.py
```

Python users can also install the original author's package from PyPI:

```sh
pip install mpedb
```

```python
import mpedb as sqlite3   # existing sqlite3 code runs unchanged
```

> These commands come from the original project's documentation. They have
> not yet been run in this copy of the repo.

## What it needs

- Rust (stable) to build. Python 3.10+ only for the Python module.
- No services, accounts or environment variables to run the engine itself.

## Where things are

| Path | What is there |
|---|---|
| `crates/mpedb-core` | The storage engine: pages, B+tree, shared memory, transactions |
| `crates/mpedb-sql` | SQL parser, planner and compiled plans |
| `crates/mpedb` | The main library you call from Rust |
| `crates/mpedb-cli` | The `mpedb` command-line tool |
| `crates/mpedb-capi` | The SQLite-compatible C library |
| `crates/mpedb-py` | The Python module |
| `crates/mpedb-mirror` | Syncing data to and from SQLite and PostgreSQL |
| `crates/mpedb-testkit` | Tests that compare results against real SQLite and PostgreSQL |
| `design/`, `docs/` | Design notes |
| `benchmarks/` | Performance measurements |

Longer guides: [GUIDE.md](GUIDE.md) (practical guide),
[COMPAT.md](COMPAT.md) (which SQL is supported),
[C-API-COMPAT.md](C-API-COMPAT.md) (the SQLite drop-in library),
[TODO.md](TODO.md) (open work).

## Known gaps

- The GitHub Actions workflows were written for the original repo. The PyPI
  publish and the GitHub Pages site need that project's accounts and secrets,
  so they are expected to fail here.
- The `foo` and `bar` files in the history are 64 MB each, which makes cloning
  slow.
- Unsupported SQL is refused with an error rather than answered wrongly. The
  full list is in [COMPAT.md](COMPAT.md).

## License

The [mpedb License 1.0](LICENSE), by the original author. It is free for
every person and organization, except that very large companies (over USD 5
billion in revenue or valuation) owe a one-time USD 0.07 per device. Any copy
must keep the LICENSE file and the copyright notice. Read [LICENSE](LICENSE)
for the exact terms.
