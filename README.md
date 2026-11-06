# shmem-sql-engine

An embedded SQL database written in Rust. Several programs on the same machine
can open one database file at the same time and share it through memory, with
no server in between. Readers never wait for writers: each reader sees a
consistent snapshot while someone else is writing, the way PostgreSQL works.

On the outside it tries to look like SQLite: the same SQL, it can read and
write real SQLite files, and it can stand in for `libsqlite3` or Python's
`sqlite3` module.

What it is **not**: a network database server, or a full replacement for
PostgreSQL. SQL it does not support is refused with an error, never answered
wrongly.

This project is based on [MPEdb](https://github.com/punnerud/mpedb) by Morten
Punnerud-Engelstad and is distributed under its license (see
[License](#license)). The crates, binaries and file formats keep the `mpedb`
name.

## How to run it

You need a stable Rust toolchain ([rustup](https://rustup.rs)). From the repo
root:

```sh
cargo build --release              # build the main workspace
cargo test --workspace             # run the test suite
cargo run -p mpedb-cli -- --help   # the `mpedb` command-line tool
```

A few crates are their own separate workspaces and are built on their own:

```sh
# SQLite drop-in library: a libsqlite3 replacement other programs can load
cargo build --release --manifest-path crates/mpedb-capi/Cargo.toml
LD_PRELOAD=$PWD/crates/mpedb-capi/target/release/libmpedb_sqlite3.so python3 app.py

# PostgreSQL wire protocol, so Postgres clients can connect
cargo build --release --manifest-path crates/mpedb-pg/Cargo.toml
```

Python, as a drop-in for the standard `sqlite3` module:

```sh
pip install mpedb
```

```python
import mpedb as sqlite3   # existing sqlite3 code runs unchanged

conn = sqlite3.connect("app.db")
conn.execute("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)")
conn.commit()
```

The file name picks the engine: `*.db` is read as a real SQLite file,
`*.mpedb` is the native multi-process format, `:memory:` stays inside one
process, and `*.toml` is a config file with a declared schema.

> **Not yet verified here:** Rust is not installed on the machine this README
> was written on, so the commands above come from the project's own docs and
> have not been run in this repo yet.

## What it needs

- Rust (stable) to build.
- Python 3.10 or newer, only for the Python module.
- No services, accounts or environment variables to run the engine itself.

## Where things are

| Path | What is there |
|---|---|
| `crates/mpedb-core` | Storage engine: pages, B+tree, shared memory, transactions |
| `crates/mpedb-sql` | SQL parser, planner and compiled query plans |
| `crates/mpedb` | The main Rust library |
| `crates/mpedb-cli` | The `mpedb` command-line tool |
| `crates/mpedb-capi` | SQLite-compatible C library (separate workspace) |
| `crates/mpedb-pg` | PostgreSQL wire protocol (separate workspace) |
| `crates/mpedb-py` | Python module |
| `crates/mpedb-mirror` | Syncing data to and from SQLite and PostgreSQL |
| `crates/mpedb-wasm` | The engine compiled for the browser |
| `crates/mpedb-fs` | Read-only FUSE view of a database file (separate workspace) |
| `crates/mpedb-testkit` | Tests that compare answers against real SQLite and PostgreSQL |
| `crates/*bench` | Benchmarks |
| `design/`, `docs/` | Design notes |
| `benchmarks/` | Measured results |
| `web/` | Browser playground |
| `deploy/` | Example systemd and nginx setups |

Longer docs:

- [GUIDE.md](GUIDE.md): practical guide (opening a database, queries, migrating from SQLite)
- [COMPAT.md](COMPAT.md): which SQLite SQL is supported
- [COMPAT-PG.md](COMPAT-PG.md): which PostgreSQL features are supported
- [C-API-COMPAT.md](C-API-COMPAT.md): the SQLite drop-in library
- [PY-COMPAT.md](PY-COMPAT.md): the Python module
- [TODO.md](TODO.md): open work

## Known gaps

- The GitHub Actions workflows came from the original project. The PyPI
  publish and GitHub Pages jobs need that project's accounts and secrets, so
  they are expected to fail in this repo.
- `pip install mpedb` installs the original author's published package, not a
  build of this repo.
- Unsupported SQL returns an error. The full list is in [COMPAT.md](COMPAT.md).

## License

Released under the mpedb License 1.0, copyright (c) 2026 Morten
Punnerud-Engelstad. It is free for every person and organization, except that
very large companies (over USD 5 billion in revenue or valuation) owe a
one-time fee of USD 0.07 per device. Every copy must keep the `LICENSE` file
and the copyright notice. Read [LICENSE](LICENSE) for the exact terms.
