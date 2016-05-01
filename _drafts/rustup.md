---
layout: post
title: "Rust on Rocketships"
author: Brian Anderson
description: ""
---

Rust is a software platform with the potential to run on anything with
a CPU. Think of anything that Runs software, and I bet we can get Rust
on it. Someday. Personally, I want to see Rust inside of spaceships.

The reason Rust is so flexible is that it is carefully designed to
depend only on the capabilities of real hardware, and to make no
assumptions whatever of its software environment. Pursuing this design
was greatly aided by building off of the world-class hardware
abstraction layer, [LLVM]. (gush about llvm)

And so today Rust can already run on nearly every modern CPU one can get
their hands on. But beyond that the Rust model is simple enough to
target nearly any *software platform* as well. Today Rust runs in some
capacity on Windows, Linux, Mac OS X, Android, iOS, all the BSD's,
including [NetBSD rump kernels], in [custom operating systems][redox].
It makes a lot of sense as [glue between other languages][skylight].

* Rust runs on embedded Linux systems ([tessel], [rpi]).
* Rust runs on ARM bare metal ([zinc]).
* Rust is easy to deploy to the cloud with [statically linked linux binaries][musl],
* Rust runs on [seL4 unikernels]
* Rust powers web servers, like [iron] and [nickel]. Both [crates.io]
  and [crater] are written in Rust.
* Rust has been compiled to video game systems (ps),
* Rust runs on mips routers.

[redox]:
[LLVM]:
[rump kernels]:
[skylight]:
[interpret]: TODO that mir report

So, damn, Rust is already practically everywhere. That is, it's
*possible* to run Rust damn near everywhere. It is not *easy*
though. There's a lot of tooling necessary to support every target:
compiler, linker, standard library, headers and possibly binaries for
system C libraries. And when considering cross-compilation, the exact
set of tools can be different for every pair of host platform (that
which the compiler runs on) and target platform (that which the
compiler's output runs on).

It's rapidly getting easier to apply Rust to anything you
can imagine.

Exciting things are happening in Rust platform support right now, and
we want to share our excitement for them, with you. Woo!

As of Rust *1.10* The Rust Project publishes [`rustc` binaries for 14
platforms, and `std` binaries for 30][platforms].

Needs to be easier to access the tools for these platforms. linkers ndks

Todo mention nightlies.



## Introducing rustup

`rustup` is a new installer for Rust. It does a few small things that
make working with Rust simpler.

We created this installer primarily to create a more consistent
development environment between Windows and Unix. Through rustup, all
Rust developers will share convenient access to all the binaries
published in each release, including nightlies and betas.

rustup gives you 2 cool superpowers:

* Install all Rust releases for any supported platform,
  and switch between them, including the beta and nightly releases.
* Install the standard library for any platform your mind can
  imagine. No, not for real. But maybe.

Today rustup is a command line program, but it's also a [library],
intended for use by graphical installers, particularly on Windows,
where it's not expected to need to use the command line, even for
development.

Just watch this thing:

TODO: screencast

### Superpower #1

So let's talk about the first superpower and why it's so super-duper:
installing the Rust compiler.

That doesn't sound very special. You can install the Rust compiler
today in a number of ways. If you go to [the rust-lang.org download
page] there are several options presented right there. If you are
on the hippest Linux distributions it may have a package for you
to install. OS X operators can get Rust and Cargo through homebrew.

rustup is yet another way to install Rust!

Exciting.

So let's see something basic that rustup does: installing multiple
Rust toolchains. In this example I create a new flibrary, 'hello',
then test it using rustc 1.8, then use rustup to install and test that
same crate on the 1.9 beta.

<script type="text/javascript"
	src="https://asciinema.org/a/4u36gqnx3w1fa6lhja8u9rlc2.js"
	id="asciicast-4u36gqnx3w1fa6lhja8u9rlc2"
	async
	data-speed="2"
></script>

```
$ cargo new hello && cd hello
$ rustc --version
$ cargo test
$ rustup install beta
$ rustup run beta rustc --version
$ rustup run beta cargo test
```

TODO: Add `rustup install`

An easy way to verify your code works on the next Rust release. That's
good Rust citizenship! If your stuff breaks on beta please report it
to the the local authorities.

So that is already not so easy to replicate using other installers:
with one `rustup install` command we added the current Rust beta to
our local Rust arsenal. If we were using the individual release
tarballs we would need to download the beta from the website. We'd
install it to a different location than the stable release then, to
test against beta would probably set the `RUSTC` environment variable
when calling `cargo`. On Windows, it would be similar, but we'd need
to use the Windows-specific installers, and the directory names would
be different.

To round out our collection of Rust compilers, let's go ahead and
install the nightly build too:

```
$ rustup install nightly
$ rustup show
```

So in addition to stable, we've also got the beta and nightly
toolchains at our disposal. These three toolchains correspond to the
Rust release channels and are updated periodically. If we wait 24
hours we should expect the Rust developers to publish a new nightly
that we'll want to get ahold of. If we wait 6 weeks there will be a
new stable release. We can update all of them with `rustup update`:

```
TODO screencast
```

rustup can also change the default toolchain:

```
$ rustup default 1.8.0
$ rustc --version
$ rustup default 1.7.0
$ rustc --version
$ rustup default 1.6.0
$ rustc --version
$ rustup default 1.5.0
$ rustc --version
$ rustup default 1.4.0
$ rustc --version
$ rustup default 1.3.0
$ rustc --version
$ rustup default 1.2.0
$ rustc --version
$ rustup default 1.1.0
$ rustc --version
$ rustup default 1.0.0
$ rustc --version
```

TODO: screencast

Woah, dude! That's totally gnarly-rad! We can make `rustc` be any
`rustc` we want it to be. It'll even be old grandpa `rustc` 1.0 if we
tell it to. You're old, 1.0. Die alone.

In addition to being an installer, rustup is also a Rust toolchain
*multiplexer*: with rustup installed, when you call `rustc` or `cargo`
(or any other command distributed with Rust) you are calling a proxy
binary, installed by rustup, that in turn calls the compiler from a
particular Rust toolchain. In this way rustup is similar to the
[rbenv] and [pyenv] tools for Ruby and Python.

This layer of indirection makes some useful things possible. For
example, on windows where Rust supports both the GNU and MSVC ABI, you
might want to switch from the default stable toolchain on Windows,
which targets the 32-bit x86 architecture and the GNU ABI, to the a
stable toolchain that targets the 64-bit, MSVC ABI.

```
$ rustup default stable-x86_64-pc-windows-msvc
$ TODO
```

Here the "stable" toolchain name is appended with an extra identifier
indicating the compiler's architecture, in this case
`x86_64-pc-windows-msvc`. This identifier is called a "target triple":
"target" because it specifies a platform for which the compiler
generates machine code; and "triple" for historical reasons. Target
triples are the basic way we refer to particular common platforms,
`rustc` by default knows about 56 of them, and `rustup` today can
obtain binaries of some kind for 30.

# Superpower #2







* Keeps Windows installation at feature parity with Unix
* Switch between versions of Rust easily
* Access to the beta and nightly release channels
* Access to the nightly archives
* Install binary standard libraries
* Keeps release channels updated
* Set toolchain per directory
* It centralizes Rust tools in ~/.cargo/bin
* Written in Rust for portability and maintainability
* Written as a library for use in GUI installers and IDEs



Some history.

On Unix
have been distributing Rust with a custom installer written in shell
script, but on Windows with `.msi` packages built with TODO, tools
that deal with installation blah blah.


and in particular to
bring toolchain multiplexing in the style of [rbenv] and [multirust]
to Windows. 


TODO: Talk about history and Diggsey.

Introduce basics, why a new
installer, what problems does it solve.

```
$ rustup install
```

TODO: just make install work



Talk about cross-compilation and linking.

Talk about host and target, target triples.

All examples will be using the basic hello world
project created by calling `cargo new --bin hello`.

TODO: Show animation of rustup in action

Mention MSVC is default (and make it so).

If you are already convinced and just want to get on with it,
visit [www.rustup.rs] to install `rustup`.

This is kinda just the start of what we're going to be able to do
now. Over time it should result in a smooth experience for all
Rust users. TODO: about Windows GUI


Reletaionship between rust / cargo.


## Example: Building static binaries on Linux

TODO: Explain that this example intentionally shows
a bunch of errors for reasons

Linux is everywhere now, and one of its unique features that has
become increasingly appreciated is its stable syscall
interface. Because the Linux kernel [puts exceptional effort into
maintaining a backward compatible kernel interface], it's possible to
distribute [ELF] binaries with no dynamic library dependencies that
will run on any version of Linux. Besides being one of the feaatures
that make [Docker] possible, it also allows developers to build
self-contained applications and deploy them to any machine running
Linux, regardless of whether it's Ubuntu or Fedora or any other
distribution, and regardless of exact mix of software libraries they
have installed.

This is one of the features that [makes Go very attractive for cloud
deployments]: the Go standard library does not depend on the C
standard library (or any other library), so Go binaries are very
portable. In contrast, the Rust standard library depends on features
in libc (at least in all implementations that exist today and for the
forseeable future). So to produce a Linux binary with no dependencies
on dynamic libraries, Rust needs to statically link to libc.

Typical Linux desktop systems build their software on [glibc], the GNU
C standard library, but it is common to find Linux systems that use
others: Android runs [BIONIC], and [Alpine Linux] runs [musl]. The
systems we do our development on though are almost universally based
on glibc, and *one of glibc's support libraries, libgcc, cannot be
linked statically*, [due to TODO some issues involving stack
unwinding]. So by default, rustc produces software that depends on a
dynamically linked glibc. It's possible still to produce binaries that
are *highly compatible* with previous versions of glibc - we build our
Linux binaries in a [TODO grotesquely modified CentOS 5 Docker
container] to do so - but to produce maximally compatible binaries we
need to use a different C library.

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

I'm going to be running on Ubuntu TODO (using this TODO docker image
if you want to follow along). That detail will be important when we
talk about linking.

The musl target triple is `x86_64-unknown-linux-musl` (there is also
`i686-uknown-linux-musl` but it is not yet available on the stable
channel). To compile for musl call `cargo` with the argument
`--target=x86_64-unknown-linux-musl`. If we just go ahead and try that
we'll get an error:

```
$ cargo build --target=x86_64-unknown-linux-musl
   Compiling hello v0.1.0 (file:///C:/Users/brian/Documents/dev/hello)
error: can't find crate for `std` [E0463]
error: aborting due to previous error
error: Could not compile `hello`.
...

```

TODO: omg 'aborting due to previous error', inconsistent capitalization

The error tells us that the compiler can't find `std`. That is because
we haven't installed it.

```
$ rustup target add x86_64-unknown-linux-gnu
TODO
```

The new libraries are downloaded and added to our existing toolchain,
which we can inspect with `rustup show`:

```
$ rustup show
stable-x86_64-unknown-linux-gnu (default toolchain)
TODO: Display more useful info.
```

So I'm running the stable toolchain for 64-bit Linux on x86, as indicated
by the `x86_64-unknown-linux-gnu` target triple. The exact version
of Rust is TODO. *And I am reportedly in possession of target support
for `x86_64-unknown-linux-musl`!*

Neat. Now we're surely ready to build a slick statically-linked binary
we can send out into the cloud to blossom. Let's try:

```
$ cargo build --target=x86_64-unknown-linux-musl
TODO
```

Linker failure. This is a horrible error. It happens because
`rustc` is trying to run `musl-gcc`, and I don't have that installed:

```
$ which musl-gcc
```

Yes, that's right, `musl-`*gcc*. TODO what? gcc explain.

So on Ubuntu TODO to get `musl-gcc` I do this crap:

```
$ whatever
```

Now can we create a Rust binary? IDK gah!

```
$ cargo build --target=x86_64-unknown-linux-musl
TODO
```

Woohoo! And of course since we've created a Linux binary,
we can just run it:

```
$ target/x86_64-unknnown-linux-musl/hello
TODO
```

Does it have any dynamic dependencies?

```
$ TODO ldd
TODO
```

Nope. You can run this binary on nearly anything
that supports the Linux kernel interface.

Like Windows:

TODO: gif of Rust running on ubuntu on windows. No way you can get this
in time.

TODO: Commands summary
TODO: Commands in fedora

## Example: Running Rust on Android from Windows

## Example: Zephyr hello world (nightly)

## Rust everywhere else

future work

learn more

Today Rust runs on ..... Tomorrow it will run everywhere, even in spaceships.

Tomorrow it will run on the web [via web web assembly and asm.js].

tommorow: wasm, interpreters

contributing

thanks


# notes

updating
cross-compiling
brag about platform support
future ndk support
emscripten
embedded screen-captures
source

see what we can use from japaric
https://github.com/emk/rust-musl-builder
https://www.reddit.com/r/rust/comments/4f6bs9/rustmuslbuilder_a_docker_image_for_building/
