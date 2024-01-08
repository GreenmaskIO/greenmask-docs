# Architecture

### General Information

It is evident that the most appropriate approach for executing logical backup dumping and restoration is by leveraging
the core PostgreSQL utilities, specifically pg_dump and pg_restore. **Greenmask** has been purposefully designed to
align with PostgreSQL's native utilities, ensuring compatibility. Greenmask primarily handles data dumping
operations independently and delegates the responsibilities of schema dumping and restoration to pg_dump and pg_restore,
maintaining seamless integration with PostgreSQL's standard tools.

### Backup Process

The process of backing up PostgreSQL databases is divided into three distinct sections:

* **Pre-data** - This section encompasses the raw schema of tables, excluding primary keys (PK) and foreign keys (FK).
* **Data** - The data section contains the actual table data in COPY format, including information about sequence
  current
  values and Large Objects data.
* **Post-data** - In this section, you'll find the definitions of indexes, triggers, rules, and constraints (such as PK
  and
  FK).

Greenmask focuses exclusively on the data section during runtime. It delegates the handling of the _pre-data_ and
_post-data_ sections to the core PostgreSQL utilities, _pg_dump_ and _pg_restore_.

Greenmask employs the **directory format** of _pg_dump_ and _pg_restore_. This format is particularly suitable for
parallel execution and partial restoration, and it includes clear metadata files that aid in determining the backup and
restoration steps. Greenmask has been optimized to work seamlessly with remote storage systems and obfuscation
procedures.

When performing data dumping, Greenmask utilizes the COPY command in TEXT format, maintaining reliability and
compatibility with the vanilla PostgreSQL utilities.

Additionally, Greenmask supports parallel execution, significantly reducing the time required for the dumping process.

## Storage Options

The core PostgreSQL utilities, _pg_dump_ and _pg_restore_, traditionally operate with files in a directory format,
offering no alternative methods. To meet **modern backup requirements** and provide flexible approaches,
Greenmask introduces the concept of **Storages**.

* **s3** - This option supports any S3-like storage system, including AWS S3, making it versatile and adaptable to
  various cloud-based storage solutions.
* **directory** - This is the standard choice, representing the ordinary filesystem directory for local storage.

!!! note
    If you have suggestions for additional storage options that would be valuable to implement, please feel free to
    share your ideas. Greenmask aims to accommodate a wide range of storage preferences to suit diverse backup needs.

## Restoration Process

In the restoration process, Greenmask combines the capabilities of different tools:

* **Schema Restoration** - Greenmask utilizes _pg_restore_ to restore the database schema. This ensures that the schema
  is accurately reconstructed.
* **Data Restoration** - For data restoration, Greenmask independently applies the data using the COPY protocol.
  This allows Greenmask to handle the data efficiently, especially when working with various storage solutions.
  Greenmask is aware of the restoration metadata, which enables it to download only the necessary data. This feature
  is particularly useful for partial restoration scenarios, such as restoring a single table from a complete backup.

Greenmask also **supports parallel restoration**, which can significantly reduce the time required to complete the
restoration process. This parallel execution enhances the efficiency of restoring large datasets.

## Data Obfuscation and Validation

Greenmask works with **COPY lines**, collects schema metadata using the Golang driver, and employs this driver in the
encoding and decoding process. The **validate command** offers a way to assess the impact on both schema
(**validation warnings**) and data (**transformation and displaying differences**). This command allows you to validate
the schema and data transformations, ensuring the desired outcomes during the obfuscation process.

## Customization

If your table schema relies on functional dependencies between columns, you can address this challenge using the
**TemplateRecord** transformer. This transformer enables you to define transformation logic for entire tables,
offering type-safe operations when assigning new values.

Greenmask provides a framework for creating your custom transformers, which can be reused efficiently. These
transformers can be seamlessly integrated without requiring recompilation, thanks to the PIPE (stdin/stdout)
interaction.

!!! note
    Furthermore, Greenmask's architecture is designed to be highly extensible, making it possible to introduce other
    interaction protocols, such as HTTP or Socket, for conducting obfuscation procedures.

## PostgreSQL Version Compatibility

**Greenmask** is compatible with PostgreSQL versions **11 and higher**.
