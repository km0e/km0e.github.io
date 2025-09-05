+++
date = 2025-03-06T17:03:15Z
description = "Force Cargo to use a specific version of a dependency"
draft = false
title = "Cargo Dependency Version"

[taxonomies]
tags = ["RUST","FAQ"]
+++

## First

Check the version of the dependency you want to use. Get the repository URL of the dependency from `crates.io` and `tag`(or `commit` hash) of the version you want to use.

## Second

Write the dependency in the `Cargo.toml` file as follows:

```toml
[patch.crates-io]
crate_name = { git = "url",tag = "version" }
```

Or

```toml
crate_name = { git = "url",rev = "commit" }
```

## Third

Run the following command to update the dependency:

```bash
cargo update -p crate_name
```
