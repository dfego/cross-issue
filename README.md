# README

This repository demonstrates an issue with cross-rs and/or cross-toolchains.

For reference, to set this repository up:

```bash
git clone https://github.com/cross-rs/cross
cd cross
git submodule set-branch --branch powerpc-musl docker/cross-toolchains
git submodule update --init --remote
cd ..
```

Then, once it's up:

```bash
cd cross
cargo build-docker-image powerpc-unknown-linux-musl-cross --tag local
cd ..
```

Issue #1, at the end of the above, I get the warning, which may or may not be relevant:

```
 1 warning found (use docker --debug to expand):
 - UndefinedVar: Usage of undefined variable '$PKG_CONFIG_PATH' (line 25)
```

At this point, build the container represented by the `Dockerfile` in this repository:

    docker build . -t cross-issue

Then, try running cross in it:

    docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/project -w /project --rm cross-issue \
      cargo +nightly build --target powerpc-unknown-linux-musl
        Compiling hello v0.1.0 (/project)
    error[E0463]: can't find crate for `std`
      |
      = note: the `powerpc-unknown-linux-musl` target may not be installed
      = help: consider downloading the target with `rustup target add powerpc-unknown-linux-musl`
      = help: consider building the standard library from source with `cargo build -Zbuild-std`

    For more information about this error, try `rustc --explain E0463`.
    error: could not compile `hello` (bin "hello") due to 1 previous error

At this point, given the `Cross.toml` present, I'd expect these errors not to make sense.

So I try passing `-Z build-std`:

    docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/project -w /project --rm cross-issue \
      cargo +nightly build --target powerpc-unknown-linux-musl -Z build-std
        Updating crates.io index
    Downloading crates ...
      Downloaded addr2line v0.22.0
      Downloaded allocator-api2 v0.2.18
      Downloaded getopts v0.2.21
      Downloaded hashbrown v0.15.0
      Downloaded compiler_builtins v0.1.138
      Downloaded unicode-width v0.1.14
      Downloaded gimli v0.29.0
      Downloaded object v0.36.5
      Downloaded libc v0.2.161
      Compiling compiler_builtins v0.1.138
      Compiling core v0.0.0 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core)
      Compiling libc v0.2.161
      Compiling std v0.0.0 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/std)
      Compiling rustc-std-workspace-core v1.99.0 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/rustc-std-workspace-core)
      Compiling alloc v0.0.0 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/alloc)
      Compiling cfg-if v1.0.0
      Compiling adler v1.0.2
      Compiling memchr v2.7.4
      Compiling rustc-demangle v0.1.24
      Compiling unwind v0.0.0 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/unwind)
      Compiling rustc-std-workspace-alloc v1.99.0 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/rustc-std-workspace-alloc)
      Compiling panic_unwind v0.0.0 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/panic_unwind)
      Compiling panic_abort v0.0.0 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/panic_abort)
      Compiling gimli v0.29.0
      Compiling miniz_oxide v0.7.4
      Compiling object v0.36.5
      Compiling std_detect v0.1.5 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/stdarch/crates/std_detect)
      Compiling hashbrown v0.15.0
      Compiling addr2line v0.22.0
      Compiling proc_macro v0.0.0 (/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/proc_macro)
      Compiling hello v0.1.0 (/project)
    error: linking with `cc` failed: exit status: 1
      |
      = note: LC_ALL="C" PATH="/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/bin:/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/bin/self-contained:/usr/local/cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" VSLANG="1033" "cc" "-m32" "crt1.o" "crti.o" "crtbegin.o" "/tmp/rustce9oyCn/symbols.o" "/project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.0dfimt4ei552u7vnvwnvwobi5.rcgu.o" "/project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.0rdw8xgh18v3aa4k563hhmswx.rcgu.o" "/project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.1rbl66onk0o5rck8kzzc13yoa.rcgu.o" "/project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.74740rl5s2xdy3y5joiujsohr.rcgu.o" "/project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.9nqtpqlrtv46goc6yqe15l09y.rcgu.o" "/project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.d22g2oc91hytdqoy7mssnn6hu.rcgu.o" "/project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.dfp559a48ofcf28za79947wh1.rcgu.o" "-Wl,--as-needed" "-Wl,-Bstatic" "/project/target/powerpc-unknown-linux-musl/debug/deps/libstd-be0783cb71818f91.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libpanic_unwind-2a2e3b76a78d4fcb.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libobject-6e8f6788e0b0a6d2.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libmemchr-d44697e8265475d7.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libaddr2line-bd0dba1b0896ee01.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libgimli-91e48d252523d189.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/librustc_demangle-dde09a8e9de60728.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libstd_detect-f8e1640f3e7526da.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libhashbrown-76386d5e5cd1ebb0.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/librustc_std_workspace_alloc-8d8cf5e10a7c0b30.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libminiz_oxide-2a562dd8445e6e27.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libadler-4e3b7a18831d1150.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libunwind-b7106ba54cd63b26.rlib" "-lunwind" "/project/target/powerpc-unknown-linux-musl/debug/deps/libcfg_if-2a5d8c05161c8163.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/liblibc-74797c36d86162d4.rlib" "-lc" "/project/target/powerpc-unknown-linux-musl/debug/deps/liballoc-2cd963000e8d97a1.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/librustc_std_workspace_core-53b843f8829846ef.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libcore-cef143d4b6991813.rlib" "/project/target/powerpc-unknown-linux-musl/debug/deps/libcompiler_builtins-def84b2bbb262997.rlib" "-Wl,-Bdynamic" "-Wl,--eh-frame-hdr" "-Wl,-z,noexecstack" "-nostartfiles" "-L" "/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/powerpc-unknown-linux-musl/lib/self-contained" "-L" "/usr/local/rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/powerpc-unknown-linux-musl/lib" "-o" "/project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec" "-Wl,--gc-sections" "-static" "-no-pie" "-Wl,-z,relro,-z,now" "-nodefaultlibs" "crtend.o" "crtn.o"
      = note: /usr/bin/ld: cannot find crt1.o: No such file or directory
              /usr/bin/ld: cannot find crti.o: No such file or directory
              /usr/bin/ld: cannot find crtbegin.o: No such file or directory
              /usr/bin/ld: /project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.0dfimt4ei552u7vnvwnvwobi5.rcgu.o: relocations in generic ELF (EM: 20)
              /usr/bin/ld: /project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.0dfimt4ei552u7vnvwnvwobi5.rcgu.o: relocations in generic ELF (EM: 20)
              /usr/bin/ld: /project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.0dfimt4ei552u7vnvwnvwobi5.rcgu.o: relocations in generic ELF (EM: 20)
              /usr/bin/ld: /project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.0dfimt4ei552u7vnvwnvwobi5.rcgu.o: relocations in generic ELF (EM: 20)
              /usr/bin/ld: /project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.0dfimt4ei552u7vnvwnvwobi5.rcgu.o: relocations in generic ELF (EM: 20)
              /usr/bin/ld: /project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.0dfimt4ei552u7vnvwnvwobi5.rcgu.o: relocations in generic ELF (EM: 20)
              /usr/bin/ld: /project/target/powerpc-unknown-linux-musl/debug/deps/hello-5dc4de284283c7ec.0dfimt4ei552u7vnvwnvwobi5.rcgu.o: error adding symbols: file in wrong format
              collect2: error: ld returned 1 exit status
              

    error: could not compile `hello` (bin "hello") due to 1 previous error