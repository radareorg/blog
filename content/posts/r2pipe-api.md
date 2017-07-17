+++
date = "2017-07-17T00:00:00+01:00"
draft = false
title = "r2pipe API"
slug = "r2api"
aliases = [
    "r2api"
    ]
+++

**r2pipe**

The r2pipe design comes from the fact that using native APIs is much more
complex and slower rather than using raw command strings and parsing the output.
It encourages users to write pipe implementations which interact with the
"quiet" mode of radare2 and use JSONs for easier deserialization. We have
multiple [implementations](https://github.com/radare/radare2-r2pipe) already
present with a number of users opting to use exploratory languages such as
Python and Ruby.

**Why r2api?**

The pipe ideally was envisioned to consist of a simple interface to spawn a r2
instance and setup remote communication with it either through pipes, HTTP, TCP
or what have you. But as the community decided to make more use of r2pipe they
also felt that the implementations should support certain higher level
abstractions through use of classes, helper functions(beyond the ones already
provided by r2) or structures to be more productive and expressive scripting.

This was implemented as a part of
[r2pipe.rs](https://github.com/radare/r2pipe.rs) which was written in Rust, a
language which heavily influences the style of programming due strong typing
features. Other rust-based projects such as radeco and rune which use r2 as
their binary loader so to say, shared certain requirements in terms of
high-level structures they wished to operate on. Hence those features were added
to r2pipe.rs as a part of the base implementation. But as the requirements
grew, the logic started moving away from the original idea and hence the
community decided to seperate the two and move these features into a seperate
playground.

[radare2-r2pipe-api](https://github.com/radare/radare2-r2pipe-api) is where you
can add support for abstractions in your favorite language while keeping the
underlying communication intact. This was recently taken up for Rust, and the
API changes have been propagated to (active :P)repositories. 

**Build fixes for Rust users:**

In case you are a r2pipe.rs/radeco-lib/rune user, you should update your
dependencies and the default rust toolchain as well.

Add the r2api dependecy in your Cargo.toml file:

```
[dependecies.r2api]
git = "https://github.com/radare/radare2-r2pipe-api"
```

Add crate usage in your lib.rs:

```rust
extern crate r2api;
```

All the structures have been moved under the `structs` module of r2api. Hence:

```rust
// use r2pipe::structs::LOpinfo;
// Replace the above line with:
use r2api::structs::LOpinfo;
```

The entire set of r2pipe functions operating on the r2 instance are now a part
of the `R2Api` trait implemented for `R2`. Hence, just add the follwing line to
files where you call functions on the r2 instance from r2pipe:

```rust
use r2pipe::r2::R2;
// Add the line below
use r2api::api_trait::R2Api;
```

Cargo deps can be updated as below:

```bash
$ cargo clean # Necessary for removing earlier conflicting versions
$ cargo update
$ cargo build
```

The community hopes to see better support for more languages being added to the
new repository. There are also plans to standardize much of the implementation
and design so make sure to contribute to the discussions! Till then, don't
forget to renew your r2 license regularly.

Relevant pull requests:

* [radare2-r2pipe-api](https://github.com/radare/radare2-r2pipe-api/pull/1)
* [radeco-lib](https://github.com/radare/radeco-lib/pull/64)
* [r2pipe.rs](https://github.com/radare/r2pipe.rs/pull/23)
* [rune](https://github.com/sushant94/rune/pull/29)
