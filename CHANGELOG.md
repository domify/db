## v0.15.0

The first release of iyi-db: crystal-db 0.15.0 ported to iyi, in iyi, with no
Crystal code left. The version follows crystal-db's, so a driver ported from
Crystal can say which crystal-db it matches.

* `IyiDb::Db.open`, `.connect` and `.register_driver` (module `iyi_db/db`) are
  crystal-db's `DB.open`, `DB.connect` and `DB.register_driver`.
* Every fallible call answers `T | DBError`; crystal-db's exception classes are
  the `DBError` hierarchy in `iyi_db/errors`.
* Query arguments and read values are `DBValue`: the scalars, `Time`, `Span`,
  `Bytes`, arrays of them, and `CustomValue` for a driver's own types.
* Several columns are read as a `Tuple(...)` or `NamedTuple(...)` type, in
  `ResultSet#read` and in `as:`.
* `derive db_serializable`, `derive db_serializable_non_strict` and `@[DBField]`
  replace `include DB::Serializable`, `DB::Serializable::NonStrict` and
  `@[DB::Field]`; `mapping` is `DB.mapping`.
* A `transaction` block left early rolls back; answering `Rollback.new` rolls
  back and answers `nil`.
* `Statement.def_around_query_or_exec` is a hook registry, and
  `CustomValue#to_log` replaces `DB::MetadataValueConverter`.
* `Pool#prefill` and `Database.open` open the initial connections; the
  constructors open nothing.
* `DriverSpecs` (crystal-db's `db/spec`) runs the shared driver specs.
* `Connection::Options.from_http_params` reads the `prepared_statements_cache`
  default from `prepared_statements_cache`, not `prepared_statements`.

### Notes for driver implementors

* Driver methods answer their errors: `ConnectionBuilder#build`,
  `Connection#build_prepared_statement`, `#build_unprepared_statement`,
  `Statement#perform_query`, `#perform_exec`, `ResultSet#move_next` and `#read`
  return `... | DBError`, and `do_close` returns `Nil | DBError`.
* A transaction subclass implements `start`, `perform_commit` and
  `perform_rollback`; `Connection#create_transaction` and
  `TopLevelTransaction#create_save_point_transaction(parent, name)` build the
  driver's own transaction types without doing IO.
* A driver's own value types inherit `CustomValue`.

crystal-db's history before the port is in its
[CHANGELOG](https://github.com/crystal-lang/crystal-db/blob/master/CHANGELOG.md).
