--
layout: post
title: "Taking Rust everywhere with rustup"
author: Brian Anderson
description: "The rustup toolchain manager makes cross-compilation in Rust a breeze"
---

*Cross-compilation* is an imposing term for a common kind of desire:

* You want to build an app for Android, or iOS, or your router using your laptop.

* You want to write, test and build code on your Mac, but deploy it to your Linux server.

* You want your Linux-based build servers to produce binaries for all the platforms
  you ship on.

* You want to build an ultraportable binary you can ship to any Linux platform.

* You want to target the browser with [Emscripten] or [WebAssembly].

[Emscripten]: http://kripken.github.io/emscripten-site/
[WebAssembly]: https://webassembly.github.io/

In other words, **you want to develop/build on one "host" platform, but get a
final binary that runs on a different "target" platform**.

Thanks to the [LLVM] backend, it's always been possible *in principle* to
cross-compile Rust code: just tell the backend to use a different target! And
indeed, early adopters have shipped Rust code to embedded Linux systems
[like the Raspberry Pi 3], [bare metal ARM],
[MIPS routers running OpenWRT][OpenWRT], and many others.

[LLVM]: http://llvm.org
[like the Raspberry Pi 3]: http://mirrors.link/posts/cross-compiling-rust-on-os-x-for-raspberry-pi-3
[bare metal ARM]: https://blog.thiago.me/raspberry-pi-bare-metal-programming-with-rust/
[OpenWRT]: https://github.com/japaric/rust-on-openwrt

But in practice, there are a lot of ducks you have to get in a row to make it
work: the appropriate Rust standard library, a cross-compiling C toolchain
including linker, headers and binaries for C libraries, and so on.
As you might expect, this generally involves poring over various blog posts and
package installers to get everything "just so". And the exact set of tools can
be different for every pair of host and target platforms.

**The Rust community has been hard at work toward the goal of "push-button
cross-compilation"**. We want to provide a complete setup for a given
host/target pair with the run of a single command. Today we're happy to announce
that a major portion of this work is reaching beta status: we're building
binaries of the Rust standard library for a wide range of targets, and shipping
them to you via a new tool called **[rustup]**.

[rustup]: https://www.rustup.rs

## Introducing rustup

At its heart, **rustup is a *toolchain manager* for Rust**. It can download and
switch between copies of the Rust compiler and standard library for the full set
of supported platforms, and track Rust's nightly, beta, and release [channels],
as well as specific versions. In this way rustup is similar to the [rvm],
[rbenv] and [pyenv] tools for Ruby and Python. I'll walk through all of this
functionality, and the situations where it's useful, in the rest of the post.

[rvm]: https://rvm.io/
[rbenv]: https://github.com/rbenv/rbenv
[pyenv]: https://github.com/yyuu/pyenv

Today rustup is a command line application, and I'm going to show you some
examples of what it can do, but it's also a [Rust library], and eventually these
features are expected to be presented through a graphical interface where
appropriate, particularly on Windows. Getting cross-compilation set up should
eventually be a matter of checking a box in the Rust installer.

As we'll see later, our ambitions go beyond managing just the Rust toolchain: to
have a true push-button experience for cross-compilation, it needs to set up the
C toolchain as well. That functionality is not shipping today, but it's
something we hope to incorporate over the next few months.

[channels]: http://blog.rust-lang.org/2014/10/30/Stability.html
[Rust library]: https://github.com/rust-lang-nursery/rustup.rs/

## Basic toolchain management

Let's start with something simple: installing multiple Rust toolchains. In this
example I create a new library, 'hello', then test it using rustc 1.8, then use
rustup to install and test that same crate on the 1.9 beta.

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

That's an easy way to verify your code works on the next Rust release. That's
good Rust citizenship!

We can use `rustup show` to show us the installed toolchains, and `rustup
update` to keep the up to date with Rust's [train releases].

[train releases]: http://blog.rust-lang.org/2014/10/30/Stability.html

<script type="text/javascript"
	src="https://asciinema.org/a/6tajyqzhhh90wuelptqdsdhn5.js"
	id="asciicast-6tajyqzhhh90wuelptqdsdhn5"
	async
	data-speed="2"
></script>

Finally, rustup can also change the default toolchain with `rustup default`
(we'll see later how to override the default for a particular project):


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

For example, on Windows [where Rust supports both the GNU and MSVC ABI][abi],
you might want to switch from the default stable toolchain on Windows, which
targets the 32-bit x86 architecture and the GNU ABI, to the a stable toolchain
that targets the 64-bit, MSVC ABI.

[abi]: https://www.rust-lang.org/downloads.html#win-foot

```
$ rustup default stable-x86_64-pc-windows-msvc
info: syncing channel updates for 'stable-x86_64-pc-windows-msvc'
info: downloading component 'rustc'
info: downloading component 'rust-std'
...

  stable-x86_64-pc-windows-msvc installed - rustc 1.8.0-stable (db2939409 2016-04-11)

```

Here the "stable" toolchain name is appended with an extra identifier indicating
the compiler's architecture, in this case `x86_64-pc-windows-msvc`. This
identifier is called a "target triple": "target" because it specifies a platform
for which the compiler generates (targets) machine code; and "triple" for
historical reasons (in many cases "triples" are actually quads these
days). Target triples are the basic way we refer to particular common platforms;
`rustc` by default knows about 56 of them, and `rustup` today can obtain
compilers for 14, and standard libraries for 30 [†].

[†]: #appendix-rustup-supported-platforms

## Example: Building static binaries on Linux

Now that we've got the basic pieces in place, let's use them for a real example:
building an ultraportable, static binary for Linux.

One of the unique features of Linux that has become increasingly appreciated is
its stable syscall interface. Because the Linux kernel puts exceptional effort
into maintaining a backward compatible kernel interface, it's possible to
distribute [ELF] binaries with no dynamic library dependencies that will run on
any version of Linux. Besides being one of the features that make Docker
possible, it also allows developers to build self-contained applications and
deploy them to any machine running Linux, regardless of whether it's Ubuntu or
Fedora or any other distribution, and regardless of exact mix of software
libraries they have installed.

[ELF]: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format

Today's Rust depends on libc, and in most host platforms that means glibc. For
technical reasons, glibc cannot be fully statically linked, making it unusable
for producing a truly standalone binary. Fortunately, an alternative exists:
[musl], a small, modern implementation of libc that can be statically linked.
Rust has been compatible with musl since TODO, but until recently developers
have needed to build their own compiler to benefit from it.

[musl]: http://www.musl-libc.org/

With that background, let's walk through compiling a statically-linked Linux
executable. For this example you'll want to be running Linux&mdash;that is, your
*host platform* will be Linux, and your *target platform* will also be Linux,
just a different flavor: musl.

I'm going to be running on Ubuntu 16.04 (using this TODO docker image). We'll be
building the basic hello world:

```
$ cargo new --bin hello && cd hello
$ cargo run
   Compiling hello v0.1.0 (file:///hello)
     Running `target/debug/hello`
Hello, world!
```

That's with the default `x86_64-unknown-linux-gnu` target. And you can
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

To start cross-compiling, you need to acquire a standard library for the target
platform. Previously, this was an error-prone, manual process&mdash;queue those
blog posts I mentioned earlier. But with rustup, it's just part of the usual
workflow:

```
$ rustup target add x86_64-unknown-linux-musl
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

So I'm running the 1.8 toolchain for 64-bit Linux on x86, as indicated by the
`x86_64-unknown-linux-gnu` target triple, and now I can also target
`x86_64-unknown-linux-musl`. Neat. Now we're surely ready to build a slick
statically-linked binary we can release into the cloud. Let's try:

```
$ cargo run --target=x86_64-unknown-linux-musl
   Compiling hello v0.1.0 (file:///hello)
     Running `target/x86_64-unknown-linux-musl/debug/hello`
Hello, world!
```

And that... just worked! No need to even install `musl-gcc` from the Ubuntu
package repository. Run `ldd` on it for proof that it's the real deal:

```
ldd target/x86_64-unknown-linux-musl/debug/hello
        not a dynamic executable
```

Now take that `hello` binary and copy it to any x86_64 machine running Linux and
it'll just work.

For more advanced use of musl consider [rust-musl-builder], a Docker
image set up for musl development, which helpfully includes common C
libraries precompiled for musl.

[rust-musl-builder]: https://github.com/emk/rust-musl-builder

## Example: Running Rust on Android

One more example. This time building for Android, from Linux, i.e.,
`arm-linux-androideabi` from `x86_64-unknown-linux-gnu`.  This can also be done
from OS X or Windows, though on Windows the process is less smooth [right now].

[right now]: https://gist.github.com/brson/a2b97ba8b3fbb7939143f584b6366751

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
...
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
... (lots of noise elided)
error: aborting due to previous error
error: Could not compile `hello`.
```

The problem is that We don't have a linker that supports Android yet, so let's
take a moment's digression to talk about building for Android.

To develop for Android we need two things:

* [The Android SDK](https://developer.android.com/sdk/index.html).
* [The Android NDK](https://developer.android.com/ndk/guides/setup.html#install).

Experienced Android developers will know what all these pieces are. The SDK
contains the Android Java libraries and the Android emulator that we're going to
run our program on. The Android *NDK* is the most important component for basic
Rust development: it contains the linker `rustc` needs to create Android
binaries. Now, technically to just *build* Rust code that runs on Android the
only thing we *need* is the NDK, but for practical development we'll want the
SDK too.

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
```

Notice that "--platform=android-18". The "18" is the Android [API
level](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html#ApiLevels).
Today, Rust's `arm-linux-androideabi` target is built against Android
API level 18, and should theoretically be forwards-compatible with
subsequent Android releases.

The final thing for us to do is tell Cargo where to find the android linker,
which is in the standalone NDK toolchain we just created. To do that we put the
following in `.cargo/config`:

```toml
[target.arm-linux-androideabi]
linker = "/root/android-18-toolchain"
```

And run our build:

```
$ cargo build --target=arm-linux-androideabi
   Compiling hello v0.1.0 (file:///root/hello)
```

Success! Of course just getting something to build is not the end of the
story. You've also got to package your code up as an Android APK. For that you
can use [cargo-apk]. I'm not going to show you that now because this post is
already exceedingly long.

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

Rust is a software platform with the potential to run on anything with a
CPU. Today I showed you a little bit of what Rust can already do, with the
rustup tool. Today Rust runs on most of the platforms you use daily. Tomorrow it
will run everywhere.

So what should you expect next?

In the coming months we're going to continue removing barriers to Rust
cross-compilation. Today rustup provides access to the standard library, but as
we've seen in this post, there's more to cross-compilation than rustc +
std. It's acquiring and configuring the linker and C toolchain that is the most
vexing&mdash;each combination of host and target platform requires something
slightly different. We want to make this easier, and will be adding "NDK
support" to rustup. What this means will again depend on the exact scenario, but
we're going to start working from the most demanded uses cases, like Android,
and try to automate as much of the detection, installation and configuration of
the non-Rust toolchain components as we can. On Android for instance, the hope
is to automate everything for a basic initial setup except for accepting the
licenses.

In addition to that there are multiple efforts to improve Rust cross-compilation
tooling, including [xargo], which can be used to build the standard library for
targets unsupported by rustup or custom targets, and [cargo-apk], which builds
Android packages from Cargo packages.

[cargo-apk]: https://users.rust-lang.org/t/announcing-cargo-apk/5501
[xargo]: https://github.com/japaric/xargo

Finally, the most exciting platform on the horizon for Rust is not a traditional
target for systems languages: the web. With [emscripten] today it's quite easy
to run *C++* code on the web by converting LLVM IR to JavaScript (or the asm.js
subset of JavaScript). And the upcoming [WebAssembly] (wasm) standard will
cement the web platform as a first-class target for programming languages.

*Rust is uniquely-positioned to be the most powerful and usable wasm-targetting
language for the immediate future.* The same properties that make Rust so
portable to real hardware makes it nearly trivial to port Rust to wasm. The same
can't be said for languages with complex runtimes that include garbage
collectors.

Rust has [already been ported to emscripten][em] (at least twice), but the code
has not yet fully landed. This summer it's happening though: Rust +
Emscripten. Rust on the Web. Rust everywhere.

[em]: https://internals.rust-lang.org/t/need-help-with-emscripten-port/3154

## Epilogue

While many people are reporting success with rustup, it remains in beta, with
some [key outstanding bugs], and is not yet the officially recommended
installation method for Rust (though you should try it). We're going to keep
soliciting feedback, applying polish, and fixing bugs. Then we're going to
improve the rustup installation experience on Windows by
[embedding it into a GUI that behaves like a proper Windows installer][gui].

At that point we'll likely update the download instructions on www.rust-lang.org
to recommend rustup. I expect all the existing installation methods to remain
available, including the non-rustup Windows installers, but at that point our
focus will be on improving the installation experience through rustup. It's also
plausible that rustup itself will be packaged for package managers like Homebrew
and dpkg.

If you want to try rustup for yourself, visit [www.rustup.rs] and follow the
instructions. Then leave feedback on the [dedicated forum thread][irlo], or
[file bugs] on the issue tracker.

[irlo]: https://internals.rust-lang.org/t/beta-testing-rustup-rs/3316/112
[gui]: https://github.com/rust-lang-nursery/rustup.rs/issues/253
[key outstanding bugs]: https://github.com/rust-lang-nursery/rustup.rs/issues?q=is%3Aopen+is%3Aissue+label%3A%22initial+release%22
[file bugs]: https://github.com/rust-lang-nursery/rustup.rs/issues
[www.rustup.rs]: https://www.rustup.rs
[emscripten]: https://kripken.github.io/emscripten-site/

## Thanks

Rust would not be half as capable a system as it is without the help of many
individuals. Thanks to Diggory Blake for writing rustup, to Jorge Aparicio for
fixing lots of cross-compilation bugs and documenting the process, Tomaka for
pioneering Rust on Android, and Alex Crichton for creating the release
infrastructure for Rust's many platforms. And thanks to all the rustup
contributors:

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
