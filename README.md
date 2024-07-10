This repository contains a reproducer for the following issue:

- The `tough` crate uses a `hyper` version < 1.0. This is a transitive dependency
  of an old version of `reqwest` that is being used.
- The `sigstore-rs` crate uses a `hyper` version >= 1.0. This is a transitive
  dependency of the `oci-distribution` crate, which brings latest version of `reqwest`.
- The main rust project has a top-level dependency on both `tough` and `sigstore-rs` and on
  latest version of `reqwest`.

Running the program will cause a runtime error of `tough`, which is used by `sigstore-rs` to checkout the
TUF repository of the Sigstore project.

A "nice" printout of the error is:

```
Caused by:
    0: Transport 'other' error fetching 'https://tuf-repo-cdn.sigstore.dev/6.root.json': error sending request for url (https://tuf-repo-cdn.sigstore.dev/6.root.json): error trying to connect: invalid URL, scheme is not http
    1: error sending request for url (https://tuf-repo-cdn.sigstore.dev/6.root.json): error trying to connect: invalid URL, scheme is not http
    2: error trying to connect: invalid URL, scheme is not http
    3: invalid URL, scheme is not http
```

This is the full, and "ugly", error:

```
Error: TufError(Transport { url: Url { scheme: "https", cannot_be_a_base: false, username: "", password: None, host: Some(Domain("tuf-repo-cdn.sigstore.dev")), port: None, path: "/6.root.json", query: None, fragment: None }, source: TransportError { kind: Other, url: "https://tuf-repo-cdn.sigstore.dev/6.root.json", source: Some(reqwest::Error { kind: Request, url: Url { scheme: "https", cannot_be_a_base: false, username: "", password: None, host: Some(Domain("tuf-repo-cdn.sigstore.dev")), port: None, path: "/6.root.json", query: None, fragment: None }, source: hyper::Error(Connect, "invalid URL, scheme is not http") }) }, backtrace: Backtrace [{ fn: "snafu::backtrace_impl::<impl snafu::GenerateImplicitData for std::backtrace::Backtrace>::generate", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/snafu-0.8.4/src/backtrace_impl_std.rs", line: 5 }, { fn: "snafu::GenerateImplicitData::generate_with_source", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/snafu-0.8.4/src/lib.rs", line: 1281 }, { fn: "<tough::error::TransportSnafu<__T0> as snafu::IntoError<tough::error::Error>>::into_error", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tough-0.17.1/src/error.rs", line: 20 }, { fn: "<core::result::Result<T,E> as snafu::ResultExt<T,E>>::context", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/snafu-0.8.4/src/lib.rs", line: 812 }, { fn: "tough::load_root::{{closure}}", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tough-0.17.1/src/lib.rs", line: 727 }, { fn: "tough::Repository::load::{{closure}}", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tough-0.17.1/src/lib.rs", line: 345 }, { fn: "tough::RepositoryLoader::load::{{closure}}", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tough-0.17.1/src/lib.rs", line: 205 }, { fn: "sigstore::trust::sigstore::SigstoreTrustRoot::new::{{closure}}", file: "/home/flavio/.cargo/git/checkouts/sigstore-rs-874f7064c0c10336/fd2968e/src/trust/sigstore/mod.rs", line: 74 }, { fn: "tough_hyper_bug::main::{{closure}}", file: "./src/main.rs", line: 11 }, { fn: "tokio::runtime::park::CachedParkThread::block_on::{{closure}}", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tokio-1.38.0/src/runtime/park.rs", line: 281 }, { fn: "tokio::runtime::coop::with_budget", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tokio-1.38.0/src/runtime/coop.rs", line: 107 }, { fn: "tokio::runtime::coop::budget", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tokio-1.38.0/src/runtime/coop.rs", line: 73 }, { fn: "tokio::runtime::park::CachedParkThread::block_on", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tokio-1.38.0/src/runtime/park.rs", line: 281 }, { fn: "tokio::runtime::context::blocking::BlockingRegionGuard::block_on", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tokio-1.38.0/src/runtime/context/blocking.rs", line: 66 }, { fn: "tokio::runtime::scheduler::multi_thread::MultiThread::block_on::{{closure}}", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tokio-1.38.0/src/runtime/scheduler/multi_thread/mod.rs", line: 87 }, { fn: "tokio::runtime::context::runtime::enter_runtime", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tokio-1.38.0/src/runtime/context/runtime.rs", line: 65 }, { fn: "tokio::runtime::scheduler::multi_thread::MultiThread::block_on", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tokio-1.38.0/src/runtime/scheduler/multi_thread/mod.rs", line: 86 }, { fn: "tokio::runtime::runtime::Runtime::block_on", file: "/home/flavio/.cargo/registry/src/index.crates.io-6f17d22bba15001f/tokio-1.38.0/src/runtime/runtime.rs", line: 349 }, { fn: "tough_hyper_bug::main", file: "./src/main.rs", line: 13 }, { fn: "core::ops::function::FnOnce::call_once", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/core/src/ops/function.rs", line: 250 }, { fn: "std::sys_common::backtrace::__rust_begin_short_backtrace", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/sys_common/backtrace.rs", line: 155 }, { fn: "std::rt::lang_start::{{closure}}", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/rt.rs", line: 159 }, { fn: "core::ops::function::impls::<impl core::ops::function::FnOnce<A> for &F>::call_once", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/core/src/ops/function.rs", line: 284 }, { fn: "std::panicking::try::do_call", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/panicking.rs", line: 559 }, { fn: "std::panicking::try", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/panicking.rs", line: 523 }, { fn: "std::panic::catch_unwind", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/panic.rs", line: 149 }, { fn: "std::rt::lang_start_internal::{{closure}}", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/rt.rs", line: 141 }, { fn: "std::panicking::try::do_call", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/panicking.rs", line: 559 }, { fn: "std::panicking::try", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/panicking.rs", line: 523 }, { fn: "std::panic::catch_unwind", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/panic.rs", line: 149 }, { fn: "std::rt::lang_start_internal", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/rt.rs", line: 141 }, { fn: "std::rt::lang_start", file: "/rustc/129f3b9964af4d4a709d1383930ade12dfe7c081/library/std/src/rt.rs", line: 158 }, { fn: "main" }, { fn: "__libc_start_main" }, { fn: "_start", file: "/home/abuild/rpmbuild/BUILD/glibc-2.31/csu/../sysdeps/x86_64/start.S", line: 120 }] })
```

The error does NOT happen if `tough` consumes latest version of `reqwest`.
