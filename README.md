# db

The common database interface for [ris](https://github.com/rislang/ris),
ported from crystal-db 0.15.0. Import a driver to use a database.

## Requirements

ris 0.1.0 or later.

## Installation

```sh
ris get github.com/domify/db --as db
```

Import modules as `db/db/...` in a project that requires this package.

## Usage

```ris
module app

import db/db/db
import db/db/errors::{DBError}
import sqlite/sqlite/sqlite

def contacts : Nil | DBError
  Db::Db.open("sqlite3:./file.db") do |db|
    db.exec("create table contacts (name text, age integer)")!
    db.exec("insert into contacts values (?, ?)", "Sarah", 33)!
    puts db.scalar("select max(age) from contacts", as: Int64)!
    db.query("select name, age from contacts") do |rs|
      rs.each do
        puts "#{rs.read(String)!} (#{rs.read(Int32)!})"
      end!
    end!
  end
end

contacts.or_panic
```

`open` and `connect` require a block. Each opens one connection and closes it
when the block ends, including early exits. Each task opens its own connection
inside that task. `Database` is an alias for `Connection`.

There is no pool, checkout, release, automatic reconnect or retry. Pool URI
options are gone. `prepared_statements` defaults to true; use
`prepared_statements=false` in the URI to disable it. `db.prepared` and
`db.unprepared` override this choice for a call. There is no statement cache.

`query` yields a result set and closes it on exit. `with_statement` prepares a
statement for its block and closes it on exit. A statement can run more than
once within that block. Query arguments are positional or passed as `args:`.

Read text and bytes inside the block that reads them. Use the block forms of
`scalar`, `query_one`, `query_one?` and `query_each` for values containing text.
The `as:` forms return flat values such as numbers and tuples of numbers.
`rs.column_names { |names| ... }` keeps generated column text inside its block.
`Type.from_db_row(rs) { |object| ... }` and `Type.from_rs(rs) { |object| ... }`
read mapped objects inside their text scope. The latter visits every row and
closes the result set.

Nested array arguments use `DBArray` positions in a caller-owned `DBArrays`.

## Errors and transactions

Fallible calls return `T | DBError`; `!` propagates the error. Errors are flat
structs with at most 240 message bytes. Use `DBFailure.new(message)` for a plain
failure, or define a struct under `DBError` for a driver error. Errors retain
no connection, nested error cause or tape-backed text.

A transaction block commits on normal completion. An error or early exit
(`return`, `break`, `!` or panic) rolls it back. Return `Rollback.new` from the
block to roll back and return nil. Call `tx.commit` before an early return that
should commit. Nested `transaction` blocks use savepoints.

## Drivers

Subclass `Driver`, `Connection`, `Statement` and `ResultSet` from `db/core`.
Register the driver with `Db::Db.register_driver("name", MyDriver)`.

`Driver#new_connection(uri, options)` builds a connection without IO;
`Connection#open` opens it. `Connection#build_statement(query, prepared)`
builds a statement without IO; `Statement#prepare` prepares it. A connection
also implements `driver_name` and `transaction_state`, which reports actual
server state including transactions started by raw SQL. A statement implements
`perform_query(args, arrays)`, `new_result_set` and `perform_exec(args, arrays)`.
A result set implements `move_next`, `column_count`, `column_name(index)`,
`next_column_index`, `read` and the text reads `read(String)` and
`read(String?)`.

The shared `DriverSpecs` use a `SpecSyntax` subclass for SQL syntax. See
`db/driver_specs_test.ri` for an example. `CustomValue` supports driver values.
`db/serializable` provides `db_serializable`, `DBField` and `mapping` for reading
objects within result blocks.

Statements log queries and arguments at debug level through `Db::Core::LOG`.
The old statement hook registry, connection builders, pool types, statement
cache and class error hierarchy are gone.

## Development

```sh
ris test db
ris fmt --check db manual
ris build manual/load.ri -o bin/load
```
