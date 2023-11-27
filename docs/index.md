# Introduction

**Greenmask** is an open-source util written on go that provides features for logical backup dumping, obfuscation and
restoration. Brings wide functionality for backing up, anonymization and masking. It is written fully in pure go
with ported required PostgreSQL library.
It is stateless util (is not required any database schema changes), has a variety of storages and provide comprehensive
obfuscation features. Was designed as easy customizable and backward compatible with PostgreSQL utils.

# Features

* **Cross-platform** - may be built and run on any platform because fully written on go without platform dependencies.
* **Database type safe**: Greenmask validated data and uses the driver for encode-decoding operation from byte representation
  into real Golang type. Can work strictly with real types thereby guaranteeing the data format.
* **Transformation validation and easy maintainable**: Greenmask allows you check the transformation result during
  obfuscation development providing validation warnings and transformation diff. That allows you to keep the transformation
  fresh in the whole software lifecycle
* **Partitioned tables transformation inheritance**: Define the transformation config once and inherit it for all
  partitions in partitioned tables
* **Stateless**: it will not affect existing schema, it is just a logical dump
* **Backward compatible** - support the same features and protocols as existing vanilla PostgreSQL utils. The dump made
  by the Greenmask might be successfully restored by `pg_restore` util
* **Extensible** - bring users the possibility to implement their own domain-based transformation in any language or use
  templates
* **Declarative** - define config in a structured easily parsed and recognizable format
* **Integrable** - integrable with your CI/CD system
* **Parallel execution** - perform parallel dumping and restoration that may significantly decrease the time to delivery
* **Provide variety of storages** - provide storages for local and remote data storing such as Directory or S3-like

## When to use?

* Daily routines with dumping and restoring a logical backup. Such as table restoration after truncation. It works in
  the
  same way as pg_dump or pg_restore, and it might be used in the same ways without struggling
* Anonymization/transformation/masking backup for staging environment and analytics purposes. It might be useful
  for deploying a pre-production environment that contains consistent anonymized data. It allows for decreased
  time to market in the development life-cycle.

## Our purpose

Greenmask util is going to be a core system within the Greenmask environment. We are trying to develop a comprehensive
**UI-based** solution for managing obfuscation procedures. We understand that it is difficult to maintain an obfuscation
state during the software lifecycle and Greenmask is trying to provide useful tools and features for keeping
the obfuscation procedure fresh, predictable, and clear.

## Links

* Email: **support@greenmask.io**
* [Twitter](https://twitter.com/GreenmaskIO)
* [Telegram](https://t.me/greenmask_community)
* [Discord](https://discord.gg/97AKHdGD)
