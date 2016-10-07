# Migration
[![GoDoc](https://godoc.org/github.com/Boostport/migration?status.png)](https://godoc.org/github.com/Boostport/migration)
[![wercker status](https://app.wercker.com/status/f4ba0d00eb6ed7ef404a11084507e09d/s/master "wercker status")](https://app.wercker.com/project/byKey/f4ba0d00eb6ed7ef404a11084507e09d)
[![Coverage Status](https://coveralls.io/repos/github/Boostport/migration/badge.svg?branch=master)](https://coveralls.io/github/Boostport/migration?branch=master)

Simple and pragmatic migrations for Go applications.

## Features
- Super simple driver interface to allow easy implementation for more database/migration drivers.
- Embeddable migration files.
- Support for up/down migrations.
- Atomic migrations (where possible, depending on database support).

## Drivers
- Apache Phoenix
- MySQL

## Quickstart
```go
// Create migration source
assetMigration := &migration.AssetMigrationSource{
    Asset:    Asset,
    AssetDir: AssetDir,
    Dir:      "test-migrations",
}

// Create driver
driver, err := migration.NewPhoenix("http://localhost:8765")

// Run all up migrations
applied, err := Migrate(driver, assetMigration, migration.Up, 0)

// Remove the last 2 migrations
applied, err := Migrate(driver, assetMigration, migration.Down, 2)
```

## Writing migrations
Migrations are extremely simple to write:
- Separate your up and down migrations into different files. For example, `1_init.up.sql` and `1_init.down.sql`.
- Prefix your migration with a number or timestamp for versioning: `1_init.up.sql` or `1475813115_init.up.sql`.
- The file-extension can be anything you want, but must be present. For example, `1_init.up.sql` is valid, but
`1_init.up` is not,

Let's say we want to write our first migration to initialize the database.

In that case, we would have a file called `1_init.up.sql` containing SQL statements for the
up migration:

```sql
CREATE TABLE test_data (
  id BIGINT NOT NULL PRIMARY KEY,
)
```

We also create a `1_init.down.sql` file containing SQL statements for the down migration:
```sql
DROP TABLE IF EXISTS test_data
```

## Embedding migration files
We use [go-bindata](https://github.com/jteeuwen/go-bindata) to embed migration files. In the
simpliest case, assuming your migration files are in `migrations/`, just run:
```
go-bindata -o bindata.go -pkg myapp migrations/
```

Then, use `AssetMigrationSource` to find the migrations:
```go
assetMigration := &migration.AssetMigrationSource{
    Asset:    Asset,
    AssetDir: AssetDir,
    Dir:      "test-migrations",
}
```

The `Asset` and `AssetDir` functions are generated by `go-bindata`.

## TODO (Pull requests welcomed!)
- [ ] Command line program to run migrations
- [ ] MigrationSource that uses migrations from the local file system
- [ ] More drivers

## Why yet another migration library?
We wanted a migration library with the following features:
- Open to extension for all sorts of databases, not just `database/sql` drivers or an ORM.
- Easily embeddable in a Go application.
- Support for embedding migration files directly into the app.

We narrowed our focus down to 2 contenders: [sql-migrate](https://github.com/rubenv/sql-migrate)
and [migrate](https://github.com/mattes/migrate/)

`sql-migrate` leans heavily on the [gorp](https://github.com/go-gorp/gorp) ORM library to perform migrations.
Unfortunately, this means that we were restricted to databases supported by `gorp`. It is easily embeddable in a
Go app and supports embedding migration files directly into the Go binary. If database support was a bit more flexible,
we would have gone with it.

`migrate` is highly extensible, and adding support for another database is extremely trivial. However, due to it using
the scheme in the dsn to determine which database driver to use, it prevented us from easily implementing an Apache
Phoenix driver, which uses the scheme to determine if we should connect over `http` or `https`. Due to the way the
project is structured, it was also almost impossible to add support for embeddable migration files without major
changes.

## License
This library is licensed under the Apache 2 License.