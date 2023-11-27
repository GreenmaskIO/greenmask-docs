# Architecture

### Common information

It is quite clear that the right way to perform logical backup dumping and restoration is using the core PostgreSQL
utils such as pg_dump and pg_restore. **Greenmask** was designed as compatible with PostgreSQL vanilla
utils, it performs only data dumping features by itself and delegates schema dumping and restoration to pg_dump and
pg_restore.

### Backing up

PostgreSQL backup is separated into three sections:

* **pre-data** - raw tables schema itself excluding PK, FK
* **data** - contains actual table data in COPY format, sequences current value setting up, and Large Objects data
* **post-data** - contains the definition of indexes, triggers, rules, constraints (such as PK, FK)

**Greenmask** operates in runtime only with _data_ section and delegates _pre-data_ and _post-data_ to pg_dump and
pg_restore.

**Greenmask** uses **directory** format of pg_dump/pg_restore.

_Directory_ format is more suitable for parallel execution, and partial restoration, and it contains clear meta-data
file that would be used for determining the backup and restoration steps. Greenmask significantly adapts them for
working with remote storages and obfuscation procedures.

_Greenmask_ performs the table data dumping using _COPY_ command in _TEXT_ format as well as _pg_dump_. It brings
reliability and compatibility with vanilla utils.

Greenmask supports parallel execution that significantly may decrease dumping time.

## Storages

PostgreSQL vanilla utils _pg_dump_ and _pg_restore_ operate with files in the directory without any alternatives.
Understanding modern backup requirements and delivering approaches **Greenmask** introducing _Storages_:

* **s3** - might be any S3-like storage, such as AWS S3
* **directory** - ordinary filesystem directory

Suggest any other storage that would be fine to implement.

## Restoration

Greenmask restores schema using pg_restore but applies COPY data by itself using COPY protocol. Due to supporting
a variety of storages and awareness of the restoration metadata, **Greenmask** may download only required data. It might
be useful in case of partial restoration, for instance restoring only one table from the whole backup.

Greenmask supports parallel restoration that significantly may decrease restoration time.

## Obfuscation and Validation

**Greenmask** operates with COPY lines, gathers schema metadata for the Golang driver, and uses this driver for
transformation in the encode-decoding procedure. The validate command allows you to check the schema affection (
validation
warnings) and data affection (performing transformation and showing diff)

## Customization

If your table schema has functional dependencies between columns it is possible to overcome this issue using
`TeamplteRecord` transformer. It allows to define transformation logic for the whole table with type-safe operations
when assigning new values.

**Greenmask** implements a framework for defining your custom transformers that might be reused later. It integrates
easily without recompiling - just PIPE (stdin/stdout) interaction.

Since Greenmask was implemented fundamentally as an extensible solution it is possible to introduce other
interaction protocols (HTTP, Socket, etc.) for performing obfuscation procedures.

## PostgreSQL version compatibility

**11 and higher**
