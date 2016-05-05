---
layout: post
title: "Rust on Rocketships"
author: Brian Anderson
description: ""
---

Rust is a software platform with the potential to run on anything with
a CPU. Think of anything that Runs software, and I bet we can get Rust
on it. Someday. Personally, I want to see Rust inside of rocketships
(call me, SpaceX!).

The reason Rust is so flexible is that it is carefully designed to
depend only on the capabilities of real hardware, and to make no
assumptions whatever of its software environment. Pursuing this design
was greatly aided by building off of the world-class hardware
abstraction layer, [LLVM].

[LLVM]: http://llvm.org

And so today Rust can already run on nearly every modern CPU one can
get their hands on. But beyond that the Rust model is simple enough to
target nearly any *software platform* as well. Today Rust runs in some
capacity on Windows, Linux, Mac OS X, Android, iOS, all the BSD's
(including [NetBSD rump kernels]), and in [operating systems written
entirely in Rust][redox].

[NetBSD rump kernels]: https://gandro.github.io/2015/09/27/rust-on-rumprun/
[redox]: http://redox-os.org/

Rust can do some things.

It makes a lot of sense as glue between other languages, [enhancing
your Ruby], or [enhancing your JavaScript], or [enhancing your Lua],
or [enhancing your Erlang], with Rust's screaming performance. Rust's
first use in production was to [speed up a Ruby codebase].

[enhancing your Ruby]: https://github.com/anima-engine/mrusty
[enhancing your JavaScript]: https://github.com/rustbridge/neon
[enhancing your Lua]: https://github.com/tomaka/hlua
[enhancing your Erlang]: https://github.com/hansihe/Rustler
[speed up a Ruby codebase]: http://blog.skylight.io/bending-the-curve-writing-safe-fast-native-gems-with-rust/

Rust can do some cool, cool things. It kinda just fits everywhere.

* Rust runs on the [seL4 microkernel].
* Rust runs on embedded Linux systems [like the Raspberry Pi 3], and the [Tessel].
* Rust can create [statically linked Linux binaries][musl] for easy deployment.
* Rust powers web servers, like [iron] and [nickel]. [crates.io] is written in Rust.
* Rust runs on [bare metal ARM] with tools like [xargo] and [zinc].
* Rust runs on [MIPS routers running OpenWRT][OpenWRT].

[like the Raspberry Pi 3]: http://mirrors.link/posts/cross-compiling-rust-on-os-x-for-raspberry-pi-3
[Tessel]: http://www.tessel.io
[bare metal ARM]: https://blog.thiago.me/raspberry-pi-bare-metal-programming-with-rust/
[xargo]: https://github.com/japaric/xargo
[zinc]: http://zinc.rs/
[musl]: https://www.reddit.com/r/rust/comments/4aq5bm/a_container_to_generate_static_rust_executables/
[seL4 microkernel]: https://air.mozilla.org/bay-area-rust-meetup-april-2016/
[iron]: http://ironframework.io/
[nickel]: http://nickel.rs/
[crates.io]: https://crates.io
[OpenWRT]: https://github.com/japaric/rust-on-openwrt

So, damn, Rust is already practically everywhere. That is, it's
*possible* to run Rust practically everywhere. It is not *easy*
though. There's a lot of tooling necessary to support every target:
compiler, linker, standard library, headers and binaries for system C
libraries. And when [considering cross-compilation][x], the exact set
of tools can be different for every pair of host platform (that which
your compiler runs on) and target platform (that which your compiled
programs run on).

[x]: https://github.com/japaric/rust-cross

But it's rapidly getting easier to apply Rust to anything you can
imagine. Exciting things are happening in Rust platform support right
now, and we want to share our excitement for them, with you.

Woo-hoo-hoo! 🚀

## Introducing rustup

[rustup] is a new installer for Rust. It does a few small things that
make working with Rust more convenient.

[rustup]: https://www.rustup.rs

We created this installer primarily to make a more consistent
development environment between Windows and Unix. Through rustup, all
Rust developers will share convenient access to all the binaries
published in each release, including nightlies and betas.

rustup gives you 2 cool superpowers:

* Install all Rust releases for any supported platform,
  and switch between them, including the beta and nightly releases.
* Install the standard library for a growing number of common
  platforms.

Today rustup is a command line application, and I'm going to show you
some examples of what it can do, [but it's also a Rust library], and
eventually these features are expected to be presented through a
graphical interface where appropriate, particularly on Windows.

## Superpower #1: toolchain multiplexing

rustup is an installer.

Why is this special? Even today you can install the Rust compiler in
a number of ways. If you go to [the rust-lang.org download page][dl]
there are several options presented right there; if you are on the
hippest Linux distributions it may have a package for you to install;
OS X operators can get Rust and Cargo through homebrew; etc.

[dl]: https://www.rust-lang.org/downloads.html

rustup is yet another way to install Rust!

How exciting.

So let's see something basic that rustup does: installing multiple
Rust toolchains. In this example I create a new library, 'hello',
then test it using rustc 1.8, then use rustup to install and test that
same crate on the 1.9 beta.

<script type="text/javascript"
	src="https://asciinema.org/a/4u36gqnx3w1fa6lhja8u9rlc2.js"
	id="asciicast-4u36gqnx3w1fa6lhja8u9rlc2"
	async
	data-speed="2"
></script>

<!--
```
$ cargo new hello && cd hello
$ rustc --version
$ cargo test
$ rustup install beta
$ rustup run beta rustc --version
$ rustup run beta cargo test
```
-->

That's an easy way to verify your code works on the next Rust
release. That's good Rust citizenship!

So this is already not so easy to replicate using other installers:
with one `rustup install` command we added the current Rust beta to
our local Rust arsenal. If we were using the individual release
tarballs we would need to download the beta from the website. We'd
install it to a different location than the stable release. Then, to
test against beta we would probably set the `RUSTC` environment
variable when calling `cargo`, or perhaps we would adjust the `PATH`
variable to find the beta toolchain instead of the stable
toolchain. On Windows, it would be similar, but we'd need to use the
Windows-specific installers, and the directory names would be
different.

TODO say something here

To extend the previous example, let's round out our collection of Rust
compilers by installing nightly as well:

```
$ rustup install nightly
info: syncing channel updates for 'nightly-x86_64-unknown-linux-gnu'
info: downloading component 'rustc'
info: downloading component 'rust-std'
info: downloading component 'rust-docs'
info: downloading component 'cargo'
info: installing component 'rustc'
info: installing component 'rust-std'
info: installing component 'rust-docs'
info: installing component 'cargo'

  nightly-x86_64-unknown-linux-gnu installed - rustc 1.10.0-nightly (8da2bcac5 2016-04-28)

$ rustup show
installed toolchains
--------------------

stable-x86_64-unknown-linux-gnu (default)
beta-x86_64-unknown-linux-gnu
nightly-x86_64-unknown-linux-gnu

active toolchain
----------------

stable-x86_64-unknown-linux-gnu (default)

rustc 1.8.0 (db2939409 2016-04-11)
```

`rustup show` indicates that, in addition to stable, we've also got
the beta and nightly toolchains at our disposal. These three
toolchains correspond to the Rust release channels and are updated
periodically. If we wait 24 hours we should expect the Rust developers
to publish a new nightly that we'll want to get ahold of. If we wait 6
weeks there will be a new stable release. We can update all of them
with `rustup update`:

<script type="text/javascript"
	src="https://asciinema.org/a/6tajyqzhhh90wuelptqdsdhn5.js"
	id="asciicast-6tajyqzhhh90wuelptqdsdhn5"
	async
	data-speed="2"
></script>

One last important feature: rustup can also change the default
toolchain with `rustup default`:

```
$ rustc --version
rustc 1.8.0 (db2939409 2016-04-11)
$ rustup default 1.7.0
info: syncing channel updates for '1.7.0-x86_64-unknown-linux-gnu'
info: downloading component 'rust'
info: installing component 'rust'
info: default toolchain set to '1.7.0-x86_64-unknown-linux-gnu'

  1.7.0-x86_64-unknown-linux-gnu installed - rustc 1.7.0 (a5d1e7a59 2016-02-29)

$ rustc --version
rustc 1.7.0 (a5d1e7a59 2016-02-29)
```

We can make `rustc` be any `rustc` we want it to be. It'll even be old
grandpa `rustc` 1.0 if we tell it to with `rustup default 1.0.0`. In
addition to being an installer, rustup is also a Rust toolchain
multiplexer: with rustup installed, when you call `rustc` or `cargo`
(or any other command distributed with Rust) you are calling a proxy
binary, installed by rustup, that in turn calls the compiler from a
particular Rust toolchain. In this way rustup is similar to the
[rbenv] and [pyenv] tools for Ruby and Python.

[rbenv]: https://github.com/rbenv/rbenv
[pyenv]: https://github.com/yyuu/pyenv

This layer of indirection enables some useful things. For example, on
Windows [where Rust supports both the GNU and MSVC ABI][abi], you
might want to switch from the default stable toolchain on Windows,
which targets the 32-bit x86 architecture and the GNU ABI, to the a
stable toolchain that targets the 64-bit, MSVC ABI.

[abi]: https://www.rust-lang.org/downloads.html#win-foot

```
$ rustup default stable-x86_64-pc-windows-msvc
info: syncing channel updates for 'stable-x86_64-pc-windows-msvc'
info: downloading component 'rustc'
info: downloading component 'rust-std'
info: downloading component 'rust-docs'
info: downloading component 'cargo'
info: installing component 'rustc'
info: installing component 'rust-std'
info: installing component 'rust-docs'
info: installing component 'cargo'

  stable-x86_64-pc-windows-msvc installed - rustc 1.8.0-stable (db2939409 2016-04-11)

```

Here the "stable" toolchain name is appended with an extra identifier
indicating the compiler's architecture, in this case
`x86_64-pc-windows-msvc`. This identifier is called a "target triple":
"target" because it specifies a platform for which the compiler
generates (targets) machine code; and "triple" for historical reasons
(in many cases "triples" are actually quads these days). Target
triples are the basic way we refer to particular common platforms,
`rustc` by default knows about 56 of them, and `rustup` today can
obtain compilers for 14, and standard libraries for 30 [†].

[†]: #appendix-rustup-supported-platforms

## Superpower #2: standard library installation

When you install the Rust compiler you also install the Rust standard
library for a single target. And even though you only have that one
standard library, your compiler can generate code for every platform
Rust supports.

The standand library is not (yet, yet, ..) distributed through Cargo like
most crates, so to get ahold of it you'll have to [build it yourself],
or use a tool like [xargo] to build it for you.

[build it yourself]: https://github.com/japaric/rust-cross
[xargo]: https://github.com/japaric/xargo

Or you can use one that *we* built for you!

Check out how I greedily acquire all these stds:

```
TODO: screencast of ridiculous combination of targets
$ rustup show
installed targets for stable-x86_64-unknown-linux-gnu
-----------------------------------------------------

x86_64-unknown-linux-gnu
i686-unknown-linux-gnu
etc.

active toolchain
----------------

stable-x86_64-unknown-linux-gnu (default)

rustc 1.8.0 (db2939409 2016-04-11)
```

TODO: implement 'show'

So it's easy to get those stds, but whether you can actually compile
to them still depends on a lot of factors. If you try to compile
for Linux PowerPC from Windows x86_64 (that is, `powerpc-unknown-linux-gnu`
from <code>x86&#95;64-pc-windows-gnu</code>) it's not going to just work:

```
$ rustup target add powerpc-unknown-linux-gnu
info: downloading component 'rust-std' for 'powerpc-unknown-linux-gnu'
info: installing component 'rust-std' for 'powerpc-unknown-linux-gnu'
$ cargo build --target=powerpc-unknown-linux-gnu
   Compiling hello v0.1.0 (file:///C:/Users/brian/Documents/dev/hello)
error: could not exec the linker `cc`: The system cannot find the file specified. (os error 2)
error: Could not compile `hello`.
```

That's an error, and it's not as incomprehensible as I might fear!
There are two things to note here. First, `rustc` failed to execute
the linker. Linking is the final stage of compilation, where the
object files filled with machine code produced by `rustc`, `gcc` and
other compilers, are combined into a final executable. Linking is
also shockingly difficult when cross-compiling.

Linkers have some baggage: there aren't a lot of them, they things
they do are not well documented, and they originate in a pre-LLVM era,
when cross-compilation was rare and crazy and difficult. As a result,
a single linker tends to target few platforms, whereas a single build
of `rustc` can generate machine code for any platform LLVM supports.

The second curious thing to note here is that `rustc` identified
the linker as `cc`: it is trying to use a C compiler to link! TODO

So `rustc` is ultimately subject to the whims of whatever C compiler
and linker happen to be available on your computer. Someday Rust will
have its own linker, and someday Rust won't require a gcc-compatible
compiler to drive it. In the meantime, rustup and other tools will
keep trying to patch together all the pieces to make cross-compilation
as easy as it should be.

For now though it's messy. Let's see some of that mess!

## Example: Building static binaries on Linux

Linux is everywhere now, and one of its unique features that has
become increasingly appreciated is its stable syscall
interface. Because the Linux kernel puts exceptional effort into
maintaining a backward compatible kernel interface, it's possible to
distribute [ELF] binaries with no dynamic library dependencies that
will run on any version of Linux. Besides being one of the features
that make Docker possible, it also allows developers to build
self-contained applications and deploy them to any machine running
Linux, regardless of whether it's Ubuntu or Fedora or any other
distribution, and regardless of exact mix of software libraries they
have installed.

[ELF]: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format

This is one of the features that makes Go very attractive for cloud
deployments: the Go standard library does not depend on the C
standard library (or any other library), so Go binaries are very
portable. In contrast, the Rust standard library depends on features
in libc (at least in all implementations that exist today and for the
foreseeable future). So to produce a Linux binary with no dependencies
on dynamic libraries, Rust needs to statically link to libc.

Typical Linux desktop systems build their software on [glibc], the GNU
C standard library, but it is common to find Linux systems that use
others: Android runs [BIONIC], and [Alpine Linux] runs [musl][]. The
systems we do our development on though are almost universally based
on glibc, and *one of glibc's support libraries, libgcc, cannot be
linked statically*, [due to TODO some issues involving stack
unwinding]. So by default, `rustc` produces software that depends on a
dynamically linked glibc. It's possible still to produce binaries that
are *highly compatible* with previous versions of glibc - we build our
Linux binaries in a [heavily modified CentOS 5 Docker
container][gross] to do so - but to produce maximally compatible binaries we
need to use a different C library.

[glibc]: https://www.gnu.org/software/libc/
[BIONIC]: https://github.com/android/platform_bionic
[musl]: http://www.musl-libc.org/
[Alpine Linux]: http://alpinelinux.org/
[gross]: https://github.com/rust-lang/rust-buildbot/tree/master/slaves/dist

In Today's Rust we do that by linking to [musl] instead. musl is
small, modern implementation of libc, and it can be linked entirely
statically. Rust has been compatible with musl since TODO, but until
recently developers have needed to build their own compiler to do so.

With that background, let's walk through compiling a statically-linked
Linux executable. For this example you'll want to be running Linux -
that is, your *host platform* will be Linux, and your *target
platform* will also be Linux, just a different flavor of Linux. It's
theoretically possible to cross-compile to Linux from any other
platform, but it's not yet well-supported by Rust.

I'm going to be running on Ubuntu 16.04 (using this TODO docker
image). We'll be building the basic hello world:

```
$ cargo new --bin hello && cd hello
cargo run
   Compiling hello v0.1.0 (file:///hello)
     Running `target/debug/hello`
Hello, world!
```

That's with the default `x86_64-unknown-linux-musl` target. And you can
see it's got lots of dynamic dependencies:

```
$ ldd target/debug/hello
        linux-vdso.so.1 =>  (0x00007ffe3d3b5000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f823d25b000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f823d03e000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f823ce27000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f823ca5e000)
        /lib64/ld-linux-x86-64.so.2 (0x0000555ac8913000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f823c755000)
```

To compile for musl instead call `cargo` with the argument
`--target=x86_64-unknown-linux-musl`. If we just go ahead and try that
we'll get an error:

```
$ cargo run --target=x86_64-unknown-linux-musl
   Compiling hello v0.1.0 (file:///C:/Users/brian/Documents/dev/hello)
error: can't find crate for `std` [E0463]
error: aborting due to previous error
error: Could not compile `hello`.
...

```

The error tells us that the compiler can't find `std`. That is of
course because we haven't installed it.

```
$ rustup target add x86_64-unknown-linux-gnu
info: downloading component 'rust-std' for 'x86_64-unknown-linux-musl'
info: installing component 'rust-std' for 'x86_64-unknown-linux-musl'
$ rustup show
installed targets for stable-x86_64-unknown-linux-gnu
-----------------------------------------------------

x86_64-unknown-linux-gnu
x86_64-unknown-linux-musl

active toolchain
----------------

stable-x86_64-unknown-linux-gnu (default)

rustc 1.8.0 (db2939409 2016-04-11)
```

So I'm running the 1.8 toolchain for 64-bit Linux on x86, as indicated
by the `x86_64-unknown-linux-gnu` target triple, and now I can also
target `x86_64-unknown-linux-musl`. Neat. Now we're surely ready to
build a slick statically-linked binary we can release into the
cloud. Let's try:

```
$ cargo run --target=x86_64-unknown-linux-musl
   Compiling hello v0.1.0 (file:///hello)
     Running `target/x86_64-unknown-linux-musl/debug/hello`
Hello, world!
```

And ... that, just worked? Hell, yeah! No need to mess with the linker
at all. This surprised me when I was writing this - I thought I would
have to install `musl-gcc` from the Ubuntu package repository, but
nope. Just works. Run `ldd` on it for proof that it's the real deal:

```
ldd target/x86_64-unknown-linux-musl/debug/hello
        not a dynamic executable
```

So, uh. That's it! take that `hello` binary and copy it to any
x86_64 machine running Linux and it'll just work.

Knock on wood(en rocketships). 🚀

For more advanced use of musl consider [rust-musl-builder], a Docker
image set up for musl development, which helpfully includes common C
libraries precompiled for musl.

## Example: Running Rust on Android

One more example. This time building for Android, from Linux,
that is, `arm-linux-androideabi` from `x86_64-unknown-linux-gnu`.
This can also be done from OS X or Windows, though on Windows
the process was less smooth [when I tried].

[when I tried]: https://gist.github.com/brson/a2b97ba8b3fbb7939143f584b6366751

To build for Android we need to install the Android target. For this
example we also need to be on nightly (older Rust builds are incompatible
with the latest Android NDK), so let's set up another 'hello,
world' project and use the nightly compiler with Android support:

```
$ cargo new --bin hello && cd hello
$ rustup override add nightly
info: syncing channel updates for 'nightly-i686-pc-windows-gnu'
info: downloading component 'rustc'
info: downloading component 'rust-std'
info: downloading component 'rust-docs'
info: downloading component 'cargo'
info: downloading component 'rust-mingw'
info: installing component 'rustc'
info: installing component 'rust-std'
info: installing component 'rust-docs'

info: installing component 'cargo'
info: installing component 'rust-mingw'
info: override toolchain for '/home/brian/hello' set to 'nightly-i686-pc-windows-gnu'

  nightly-i686-pc-windows-gnu installed - rustc 1.10.0-nightly (8da2bcac5 2016-04-28)

$ rustup target add arm-linux-androideabi
info: downloading component 'rust-std' for 'arm-linux-androideabi'
info: installing component 'rust-std' for 'arm-linux-androideabi'
$ rustup show
installed toolchains
--------------------

stable-x86_64-unknown-linux-gnu (default)
nightly-x86_64-unknown-linux-gnu

installed targets for nightly-x86_64-unknown-linux-gnu
-----------------------------------------------------

arm-linux-androideabi
i686-pc-windows-gnu

active toolchain
----------------

nightly-x86_64-unknown-linux-gnu (override)

rustc 1.10.0-nightly (8da2bcac5 2016-04-28)
```

Note that this time we did something new: we used `rustup override add
nightly` to configure *just this project* to use the nightly toolchain
instead of stable. Overrides in rustup configure specific projects
to build with specific toolchains.

So let's see what happens if we try to just build our 'hello'
project without installing anything further:

```
$ cargo build --target=arm-linux-androideabi
   Compiling hello v0.1.0 (file:///home/brson/hello)
   error: linking with `cc` failed: exit code: 1
   note: "cc" "-Wl,--as-needed" "-Wl,-z,noexecstack" "-Wl,--allow-multiple-definition" "-L" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib" "/home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o" "-o" "/home/brson/hello/target/arm-linux-androideabi/debug/hello" "-Wl,--gc-sections" "-pie" "-nodefaultlibs" "-L" "/home/brson/hello/target/arm-linux-androideabi/debug" "-L" "/home/brson/hello/target/arm-linux-androideabi/debug/deps" "-L" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib" "-Wl,-Bstatic" "-Wl,-Bdynamic" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib/libstd-cb705824.rlib" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib/libcollections-cb705824.rlib" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib/librustc_unicode-cb705824.rlib" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib/librand-cb705824.rlib" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib/liballoc-cb705824.rlib" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib/liballoc_jemalloc-cb705824.rlib" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib/liblibc-cb705824.rlib" "/root/.multirust/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/arm-linux-androideabi/lib/libcore-cb705824.rlib" "-l" "dl" "-l" "log" "-l" "gcc" "-l" "gcc" "-l" "c" "-l" "m" "-l" "compiler-rt"
   note: /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /usr/bin/ld: /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: Relocations in generic ELF (EM: 40)
   /home/brson/hello/target/arm-linux-androideabi/debug/hello.0.o: error adding symbols: File in wrong format
   collect2: error: ld returned 1 exit status

error: aborting due to previous error
error: Could not compile `hello`.
```

That! Is a bad error. TODO explain.

We don't have a linker that supports Android yet, so let's take a
moment's digression to talk about building for Android.

To develop for Android we need 2 things:

* [The Android SDK](https://developer.android.com/sdk/index.html).
* [The Android NDK](https://developer.android.com/ndk/guides/setup.html#install).

Experienced Android developers will know what all these pieces
are. The SDK contains the Android Java libraries and the Android
emulator that we're going to run our program on. The Android *NDK* is
the most important component for basic Rust development: it contains
the linker `rustc` needs to create Android binaries. Now, technically
to just *build* Rust code that runs on Android the only thing we
*need* is the NDK, but for practical development we'll want the SDK
too.

On Linux, download and unpack them with these commands:

```
$ curl -O https://dl.google.com/android/android-sdk_r24.4.1-linux.tgz
$ android-sdk_r24.4.1-linux.tgz
$ curl -O http://dl.google.com/android/repository/android-ndk-r11c-linux-x86_64.zip
$ unzip android-ndk-r11c-linux-x86_64.zip
```

Make the standalane toolchain: TODO explain

```
$ android-ndk-r11c/build/tools/make-standalone-toolchain.sh \
      --platform=android-18 --toolchain=arm-linux-androideabi-clang \
      --install-dir=android-18-toolchain --ndk-dir=android-ndk-r11c/ --arch=arm
HOST_OS=linux
HOST_EXE=
HOST_ARCH=x86_64
HOST_TAG=linux-x86_64
HOST_NUM_CPUS=4
BUILD_NUM_CPUS=8
Copying prebuilt binaries...
Copying sysroot headers and libraries...
Copying c++ runtime headers and libraries...
Copying files to: /home/brian/android-18-toolchain
Cleaning up...
Done.
```

Notice that "--platform=android-18". The "18" is the Android [API
level](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html#ApiLevels).
Today, Rust's `arm-linux-androideabi` target is built against Android
API level 18, and should theoretically be forwards-compatible with
subsequent Android releases.

The final thing for us to do is tell Cargo where to find the android linker,
which is in the standalone NDK toolchain we just created. To do that
we put the following in `.cargo/config`:

```toml
[target.arm-linux-androideabi]
linker = "/root/android-18-toolchain"
```

And run our build:

```
$ cargo build --target=arm-linux-androideabi
   Compiling hello v0.1.0 (file:///root/hello)
```

Success! Of course just getting something to build is not the end of
the story. You've also got to package your code up as an Android
APK. For that you can use [cargo-apk]. I'm not going to show you that
now because this post is already exceedingly long.

[cargo-apk]: https://users.rust-lang.org/t/announcing-cargo-apk/5501

<!--

```
$ export NDK_HOME=/root/android-ndk-r10e
$ export ANDROID_HOME=/root/android-sdk-linux
$ apt install openjdk-8jdk ant
```

Note that `NDK_HOME` is pointing to the NDK itself, not the
"standalone toolchain" we created earlier. `cargo apk` uses
the NDK directly

```
$ /root/android-sdk-linux/tools/android update sdk -a --no-ui \
      --filter android-18,sys-image-armeabi-v7a-android-18
```

Need to click through some licenses.

```
$ /root/android-sdk-linux/tools/android create avd \
      --name arm-18 --target android-18 --abi armeabi-v7a
```

When asked whether to "create a custom hardware profile", say "no".

TODO
-->


## Rust everywhere else

Rust is a software platform with the potential to run on anything with
a CPU. Today I showed you a little bit of what Rust can already do,
with the rustup tool. Today Rust runs on most of the platforms you use
daily. Tomorrow it will run everywhere.

So what should you expect next?

In the coming months we're going to continue removing barriers to Rust
cross-compilation. Today rustup provides access to the standard
library, but as we've seen in this post, there's more to
cross-compilation than rustc + std. It's acquiring and configuring the
linker and C toolchain that is the most vexing - each combination of
host and target platform requires something slightly different. We
want to make this easier, and will be adding "NDK support" to
rustup. What this means will again depend on the exact scenario, but
we're going to start working from the most demanded uses cases, like
Android, and try to automate as much of the detection, installation
and configuration of the non-Rust toolchain components as we can. On
Android for instance, the hope is to automate everything for a basic
initial setup except for accepting the licenses.

In addition to that there are multiple efforts to improve Rust
cross-compilation tooling, including the aforementioned [xargo], which
can be used to build the standard library for targets unsupported by
rustup or custom targets; and [cargo-apk], which builds Android
packages from Cargo packages.

[cargo-apk]: https://users.rust-lang.org/t/announcing-cargo-apk/5501
[xargo]: https://github.com/japaric/xargo

Finally, the most exciting platform on the horizon for Rust is not a
traditional target for systems languages: the web. With [emscripten]
today it's quite easy to run *C++* code on the web by converting LLVM
IR to JavaScript (or the asm.js subset of JavaScript). And the
upcoming [WebAssembly][] (wasm) standard will cement the web platform as a
first-class target for programming languages.

[WebAssembly]: https://en.wikipedia.org/wiki/WebAssembly

*Rust is uniquely-positioned to be the most powerful and usable
wasm-targetting language for the immediate future.* The same
properties that make Rust so portable to real hardware makes it nearly
trivial to port Rust to wasm. The same can't be said for languages
with complex runtimes that include garbage collectors.

Rust has [already been ported to emscripten][em] (at least twice),
but the code has not yet fully landed. This summer it's happening
though: Rust + Emscripten. Rust on the Web. Rust everywhere.

[em]: https://internals.rust-lang.org/t/need-help-with-emscripten-port/3154

## rustup - fuck yeah

While many people are reporting success with rustup, it
remains in beta, with some [key outstanding bugs], and is not yet the
officially recommended installation method for Rust (though you should
try it). We're going to keep soliciting feedback, applying polish, and
fixing bugs. Then we're going to improve the rustup installation
experience on Windows by [embedding it into a GUI that behaves like a
proper Windows installer][gui].

At that point we'll likely update the download instructions on
www.rust-lang.org to recommend rustup. I expect all the existing
installation methods to remain available, including the non-rustup
Windows installers, but at that point our focus will be on improving
the installation experience through rustup. It's also plausible
that rustup itself will be packaged for package managers like
Homebrew and dpkg.

If you want to try rustup for yourself, visit [www.rustup.rs] and
follow the instructions. Then leave feedback on the [dedicated forum
thread][irlo], or [file bugs] on the issue tracker.

[irlo]: https://internals.rust-lang.org/t/beta-testing-rustup-rs/3316/112
[gui]: https://github.com/rust-lang-nursery/rustup.rs/issues/253
[key outstanding bugs]: https://github.com/rust-lang-nursery/rustup.rs/issues?q=is%3Aopen+is%3Aissue+label%3A%22initial+release%22
[file bugs]: https://github.com/rust-lang-nursery/rustup.rs/issues
[www.rustup.rs]: https://www.rustup.rs
[emscripten]: https://kripken.github.io/emscripten-site/

## Thanks

Rust would not be half as capable a system as it is without the help
of many individuals. Thanks to Diggory Blake for writing rustup,
to Jorge Aparicio for fixing lots of cross-compilation bugs and
documenting the process, Tomaka for pioneering Rust on Android,
and Alex Crichton for creating the release infrastructure for Rust's
many platforms. And thanks to all the rustup contributors:

* Alex Crichton
* Brian Anderson
* Corey Farwell
* David Salter
* Diggory Blake
* Jacob Shaffer
* Jeremiah Peschka
* Joe Wilm
* Jorge Aparicio
* Kai Noda
* Kamal Marhubi
* Kevin K
* llogiq
* Mika Attila
* NODA, Kai
* Paul Padier
* Severen Redwood
* Tim Neumann
* trolleyman
* Vadim Petrochenkov
* V Jackson
* Vladimir
* Wayne Warren
* Y. T. Chung


