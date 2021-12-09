+++
title = "Tracing Introduction"
description = ""
date = 2020-08-06
draft = true

[taxonomies]
categories = ["Programming"]
tags = ["post", "rust", "tracing"]
+++

# Tracing
[Tracing](https://github.com/tokio-rs/tracing) is an instrumentation library
that makes it easy to add traceable spans to your rust code.

# Just Print It
The simplest example is to just log your tracing output, that will result in
the same as using a logging crate, like [slog](https://github.com/slog-rs/slog).

```toml
[dependencies]
tracing = "0.1"
```

```rust
use tracing::info;

fn main() {
    info!("hello world");
}
```
When we run this...

{% term() %}
<span class="term-fgx76">❯</span> cargo run -p just-print-it
<span class="term-fg32 term-fg1">    Finished</span> dev [unoptimized + debuginfo] target(s) in 0.01s
<span class="term-fg32 term-fg1">     Running</span> `target&#47;debug&#47;just-print-it`
{% end %}

Nothing happens.

But that's easy to solve, if we look at the tracing examples, we will realize
that we need an subscriber to receive the tracing events.

Let's add the new dependency
```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = "0.2"
```

Setup the subscriber
```rust
use tracing::info;
use tracing_subscriber;

fn main() {
    tracing_subscriber::fmt::init();
    info!("hello world");
}
```

And now...
{% term()%}
<span class="term-fgx76">❯</span> cargo run -p just-print-it
<span class="term-fg32 term-fg1">    Finished</span> dev [unoptimized + debuginfo] target(s) in 0.02s
<span class="term-fg32 term-fg1">     Running</span> `target&#47;debug&#47;just-print-it`
{% end %}

Still nothing...

Oh, turns out we need to set the `RUST_LOG` environment variable to `info`, by
default the subscriber is printing only messages with level `error` and above.

{% term()%}
<span class="term-fgx76">❯</span> RUST_LOG=info cargo run -p just-print-it
<span class="term-fg32 term-fg1">    Finished</span> dev [unoptimized + debuginfo] target(s) in 0.02s
<span class="term-fg32 term-fg1">     Running</span> `target&#47;debug&#47;just-print-it`
<span class="term-fg2">Dec 09 09:03:18.860</span> <span class="term-fg32"> INFO</span> just_print_it: hello world
{% end %}

Lets add some functions
```rust
use tracing::{error, info};
use tracing_subscriber;

fn fn_a () {
    info!("hello from fn_a");
    for i in (0..3).rev() {
        let res = sub_one(i);
        info!(num=i, res, "{}-1={}", i, res);
    }
}

fn sub_one (value: u8) -> u8 {
    info!("hello from sub_one");
    match value.checked_sub(1) {
        Some(i) => i,
        None => {
            error!("subtraction failed!");
            0
        },
    }
}

fn main() {
    tracing_subscriber::fmt::init();
    info!("hello from main");
    fn_a();
}
```

{% term() %}
<span class="term-fgx76">❯</span> RUST_LOG=info cargo run -p just-print-it
<span class="term-fg32 term-fg1">    Finished</span> dev [unoptimized + debuginfo] target(s) in 0.02s
<span class="term-fg32 term-fg1">     Running</span> `target&#47;debug&#47;just-print-it`
<span class="term-fg2">Dec 09 09:05:32.934</span> <span class="term-fg32"> INFO</span> just_print_it: hello from main
<span class="term-fg2">Dec 09 09:05:32.934</span> <span class="term-fg32"> INFO</span> just_print_it: hello from fn_a
<span class="term-fg2">Dec 09 09:05:32.934</span> <span class="term-fg32"> INFO</span> just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:05:32.934</span> <span class="term-fg32"> INFO</span> just_print_it: 2-1=1 num=2 res=1
<span class="term-fg2">Dec 09 09:05:32.934</span> <span class="term-fg32"> INFO</span> just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:05:32.934</span> <span class="term-fg32"> INFO</span> just_print_it: 1-1=0 num=1 res=0
<span class="term-fg2">Dec 09 09:05:32.934</span> <span class="term-fg32"> INFO</span> just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:05:32.934</span> <span class="term-fg31">ERROR</span> just_print_it: subtraction failed!
<span class="term-fg2">Dec 09 09:05:32.934</span> <span class="term-fg32"> INFO</span> just_print_it: 0-1=0 num=0 res=0
{% end %}

Using `tracing::instrument`

```rust
use tracing::{error, info};
use tracing_subscriber;

#[tracing::instrument]
fn fn_a () {
    info!("hello from fn_a");
    for i in (0..3).rev() {
        let res = sub_one(i);
        info!(num=i, res, "{}-1={}", i, res);
    }
}

#[tracing::instrument]
fn sub_one (value: u8) -> u8 {
    info!("hello from sub_one");
    match value.checked_sub(1) {
        Some(i) => i,
        None => {
            error!("subtraction failed!");
            0
        },
    }
}

fn main() {
    tracing_subscriber::fmt::init();
    info!("hello from main");
    fn_a();
}
```

{% term()%}
<span class="term-fgx76">❯</span> RUST_LOG=info cargo run -p just-print-it
<span class="term-fg32 term-fg1">    Finished</span> dev [unoptimized + debuginfo] target(s) in 0.02s
<span class="term-fg32 term-fg1">     Running</span> `target&#47;debug&#47;just-print-it`
<span class="term-fg2">Dec 09 09:07:08.406</span> <span class="term-fg32"> INFO</span> just_print_it: hello from main
<span class="term-fg2">Dec 09 09:07:08.406</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: hello from fn_a
<span class="term-fg2">Dec 09 09:07:08.406</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=2<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:07:08.406</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 2-1=1 num=2 res=1
<span class="term-fg2">Dec 09 09:07:08.406</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=1<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:07:08.406</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 1-1=0 num=1 res=0
<span class="term-fg2">Dec 09 09:07:08.406</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=0<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:07:08.406</span> <span class="term-fg31">ERROR</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=0<span class="term-fg1">}</span>: just_print_it: subtraction failed!
<span class="term-fg2">Dec 09 09:07:08.407</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 0-1=0 num=0 res=0
{% end %}

{% term() %}
<span class="term-fgx76">❯</span> RUST_LOG=info cargo run -p just-print-it
<span class="term-fg32 term-fg1">   Compiling</span> just-print-it v0.1.0 (&#47;home&#47;peer&#47;Development&#47;rust&#47;tracing-the-tracer&#47;just-print-it)
<span class="term-fg32 term-fg1">    Finished</span> dev [unoptimized + debuginfo] target(s) in 0.98s
<span class="term-fg32 term-fg1">     Running</span> `target&#47;debug&#47;just-print-it`
<span class="term-fg2">Dec 09 09:08:47.666</span> <span class="term-fg32"> INFO</span> just_print_it: hello from main
<span class="term-fg2">Dec 09 09:08:47.666</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: hello from fn_a
<span class="term-fg2">Dec 09 09:08:47.666</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=2 secret=&quot;some secret&quot;<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:08:47.667</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 2-1=1 num=2 res=1
<span class="term-fg2">Dec 09 09:08:47.667</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=1 secret=&quot;some secret&quot;<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:08:47.667</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 1-1=0 num=1 res=0
<span class="term-fg2">Dec 09 09:08:47.667</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=0 secret=&quot;some secret&quot;<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:08:47.667</span> <span class="term-fg31">ERROR</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=0 secret=&quot;some secret&quot;<span class="term-fg1">}</span>: just_print_it: subtraction failed!
<span class="term-fg2">Dec 09 09:08:47.667</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 0-1=0 num=0 res=0
{% end %}

{% term() %}
<span class="term-fgx76">❯</span> RUST_LOG=info cargo run -p just-print-it
<span class="term-fg32 term-fg1">   Compiling</span> just-print-it v0.1.0 (&#47;home&#47;peer&#47;Development&#47;rust&#47;tracing-the-tracer&#47;just-print-it)
<span class="term-fg33 term-fg1">warning</span><span class="term-fg1">: unused variable: `secret`</span>
  <span class="term-fgx12 term-fg1">--&gt; </span>just-print-it&#47;src&#47;main.rs:16:24
   <span class="term-fgx12 term-fg1">|</span>
<span class="term-fgx12 term-fg1">16</span> <span class="term-fgx12 term-fg1">| </span>fn sub_one (value: u8, secret: &amp;str) -&gt; u8 {
   <span class="term-fgx12 term-fg1">| </span>                       <span class="term-fg33 term-fg1">^^^^^^</span> <span class="term-fg33 term-fg1">help: if this is intentional, prefix it with an underscore: `_secret`</span>
   <span class="term-fgx12 term-fg1">|</span>
   <span class="term-fgx12 term-fg1">= </span><span class="term-fg1">note</span>: `#[warn(unused_variables)]` on by default
&nbsp;
<span class="term-fg33 term-fg1">warning</span><span class="term-fg1">:</span> `just-print-it` (bin &quot;just-print-it&quot;) generated 1 warning
<span class="term-fg32 term-fg1">    Finished</span> dev [unoptimized + debuginfo] target(s) in 0.97s
<span class="term-fg32 term-fg1">     Running</span> `target&#47;debug&#47;just-print-it`
<span class="term-fg2">Dec 09 09:09:32.513</span> <span class="term-fg32"> INFO</span> just_print_it: hello from main
<span class="term-fg2">Dec 09 09:09:32.514</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: hello from fn_a
<span class="term-fg2">Dec 09 09:09:32.514</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=2<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:09:32.514</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 2-1=1 num=2 res=1
<span class="term-fg2">Dec 09 09:09:32.514</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=1<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:09:32.515</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 1-1=0 num=1 res=0
<span class="term-fg2">Dec 09 09:09:32.515</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=0<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:09:32.515</span> <span class="term-fg31">ERROR</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">sub_one{</span>value=0<span class="term-fg1">}</span>: just_print_it: subtraction failed!
<span class="term-fg2">Dec 09 09:09:32.515</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 0-1=0 num=0 res=0
{% end %}

{% term() %}
<span class="term-fgx76">❯</span> RUST_LOG=info cargo run -p just-print-it
<span class="term-fg32 term-fg1">    Finished</span> dev [unoptimized + debuginfo] target(s) in 0.02s
<span class="term-fg32 term-fg1">     Running</span> `target&#47;debug&#47;just-print-it`
<span class="term-fg2">Dec 09 09:11:39.232</span> <span class="term-fg32"> INFO</span> just_print_it: hello from main
<span class="term-fg2">Dec 09 09:11:39.233</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: hello from fn_a
<span class="term-fg2">Dec 09 09:11:39.233</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">decrement{</span>value=2 secret=*******<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:11:39.233</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 2-1=1 num=2 res=1
<span class="term-fg2">Dec 09 09:11:39.233</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">decrement{</span>value=1 secret=*******<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:11:39.233</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 1-1=0 num=1 res=0
<span class="term-fg2">Dec 09 09:11:39.233</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">decrement{</span>value=0 secret=*******<span class="term-fg1">}</span>: just_print_it: hello from sub_one
<span class="term-fg2">Dec 09 09:11:39.233</span> <span class="term-fg31">ERROR</span> <span class="term-fg1">fn_a</span>:<span class="term-fg1">decrement{</span>value=0 secret=*******<span class="term-fg1">}</span>: just_print_it: subtraction failed!
<span class="term-fg2">Dec 09 09:11:39.233</span> <span class="term-fg32"> INFO</span> <span class="term-fg1">fn_a</span>: just_print_it: 0-1=0 num=0 res=0
{% end %}

{% term() %}
{% end %}
# Process Tracing
# Distributed Tracing
# All Togheter
