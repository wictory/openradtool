# Introduction

**openradtool** ("**ort**") translates business logic of a database application---objects and
queries---into C, SQL ([SQLite](https://sqlite.org)), and TypeScript/JavaScript code.
The business logic is given as a configuration file, for example:

```
struct user {
  field name text null comment "full name (or null)";
  field email text unique comment "unique e-mail";
  field hash password comment "hashed password";
  field id int rowid comment "unique identifier";
  search id: comment "search by user identifier";
  search email,password: name creds comment "lookup by credentials";
  comment "a system user";
};

struct session {
  field user struct uid;
  field uid:user.id int comment "user bound to session";
  field token int comment "unique session token";
  field id int rowid;
  comment "web session";
};
```

This configuration may then be translated into a C API (header file and implementation) and an SQL
schema or update sequence.
The API consists of "getters", "setters", updaters, and deleters; and is implemented in
straight-forward C code linked directly into applications.

The generated files currently use [ksql(3)](https://kristaps.bsd.lv/ksql) to wrap around SQLite and
(optionally) [kcgi(3)](https://kristaps.bsd.lv/kcgi) for JSON or validation output.

**ort requires the newest library versions to operate**:  all of these tools are built in tandem.

Why is **ort** handy?
It removes a lot of "boilerplate" code querying the database and allocating objects.
Some additional features:

- Functions generated for modifying (updates), allocating (inserts), and accessing (queries).
- Automagic foreign key "INNER JOIN" when accessing structures.
  The configuration specifies nested structures and **ort** creates the "join" functionality when
  querying for those objects.
- Several types of accessors: iterator-based (function callback), list-based (queue), and unique
  (selecting on unique fields).
- Well-formed, readable C and SQL output describing functions (if applicable), variables,
  structures, and so on.
- "Difference calculator" between configuration files helps database updates be reasonable by
  providing the necessary "ALTER TABLE" and so on to help with versioning. 
  (Also protects against accidental incompatible changes.)
- Beyond the usual native type support (int, text, real, blob), also supports "password" type that
  has automatic hashing mechanism built-in during selection from and insertion into the database.
- Several different types of SQL query (and update and delete) operators.
- Optional JSON output functions.
- Optional JSON input functions.
- Optional field validation ([kcgi(3)](https://kristaps.bsd.lv/kcgi)) functions.
- Well-documented JavaScript or TypeScript handling of the JSON output generator.

See the [TODO](TODO.md) for what still needs to be done.

This repository mirrors the main repository on [BSD.lv](https://www.bsd.lv).
**ort** is still very much under development!

# Installation

**ort** runs best on OpenBSD, since this system offers the most comprehensive security.
Why is this significant?
With OpenBSD, processes may stipulate resource constraints effectively making them unable to change
their running environment: for example, an application process may disable the facility to read
files after spawning the database process.
This effectively walls away the database from the application.

The sources generated by **ort** will also work on Linux, FreeBSD, and Mac OSX.

If your operating system doesn't already have **ort** bundled as a third-party system, simply
download and run the usual `./configure` and `doas make install`.

# License

All sources use the ISC (like OpenBSD) license.
See the [LICENSE.md](LICENSE.md) file for details.
