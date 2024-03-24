<!--
Thank you for finding an Internal Compiler Error! 🧊  If possible, try to provide
a minimal verifiable example. You can read "Rust Bug Minimization Patterns" for
how to create smaller examples.
http://blog.pnkfx.org/blog/2019/11/18/rust-bug-minimization-patterns/
-->

It seems that a link to a private item in a proc_macro crate can lead to compiler panicks.

### Code

I have reduced my existing code as following:
The `base_crate` depends on the proc-macro crate `pm_crate`. The later has only a single macro which itself
has a link to a private item in the `pm_crate` in its docstring. This leads to a warning when generating documentation
for the `pm_crate` but the compiler panicks when running `cargo doc` or `cargo rustdoc` for the `base_crate`.
```
├── Cargo.toml
├── pm_crate
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── README.md
└── src
    └── lib.rs
```
Have a look at the code right [here](https://github.com/jonaspleyer/compiler-error).

### Meta
<!--
If you're using the stable version of the compiler, you should also check if the
bug also exists in the beta or nightly versions.
-->

`rustc --version --verbose`:
```
# Nightly
rustc 1.79.0-nightly (2f090c30d 2024-03-23)
binary: rustc
commit-hash: 2f090c30ddd6b3bbe5c81c087579a5166e7c7278
commit-date: 2024-03-23
host: x86_64-unknown-linux-gnu
release: 1.79.0-nightly
LLVM version: 18.1.2

# Beta
rustc 1.78.0-beta.2 (277d06bc9 2024-03-23)
binary: rustc
commit-hash: 277d06bc95c6c38a2337ccde798b2c709384bd84
commit-date: 2024-03-23
host: x86_64-unknown-linux-gnu
release: 1.78.0-beta.2
LLVM version: 18.1.2

# Stable
rustc 1.77.0 (aedd173a2 2024-03-17)
binary: rustc
commit-hash: aedd173a2c086e558c2b66d3743b344f977621a7
commit-date: 2024-03-17
host: x86_64-unknown-linux-gnu
release: 1.77.0
LLVM version: 17.0.6

```

### Error output

```
error: could not document `base_crate`

Caused by:
  process didn't exit successfully: `/home/USER/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/bin/rustdoc --crate-type lib --crate-name base_crate src/lib.rs -o /home/USERBUILDDIR/compiler_error/target/doc --error-format=json --json=diagnostic-rendered-ansi,artifacts,future-incompat --diagnostic-width=156 -C metadata=295f613bd8c64072 -L dependency=/home/USER/BUILDDIR/compiler_error/target/debug/deps --extern pm_crate=/home/USER/BUILDDIR/compiler_error/target/debug/deps/libpm_crate-5e398788ba50bfb5.so --crate-version 0.0.0` (exit status: 101)
```

<!--
Include a backtrace in the code block by setting `RUST_BACKTRACE=1` in your
environment. E.g. `RUST_BACKTRACE=1 cargo build`.
-->
<details><summary><strong></strong></summary>
<p>

```
thread 'rustc' panicked at compiler/rustc_metadata/src/rmeta/decoder.rs:1507:75:
called `Option::unwrap()` on a `None` value
stack backtrace:
   0:     0x79ee92f1d3e5 - std::backtrace_rs::backtrace::libunwind::trace::h3a9d45e1b8c03229
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/std/src/../../backtrace/src/backtrace/libunwind.rs:105:5
   1:     0x79ee92f1d3e5 - std::backtrace_rs::backtrace::trace_unsynchronized::he9adce911761e763
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/std/src/../../backtrace/src/backtrace/mod.rs:66:5
   2:     0x79ee92f1d3e5 - std::backtrace::Backtrace::create::h0d89c3978930e1d9
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/std/src/backtrace.rs:331:13
   3:     0x79ee92f1d335 - std::backtrace::Backtrace::force_capture::h1db55c8cb0c499d8
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/std/src/backtrace.rs:312:9
   4:     0x79ee8f9cb1b4 - std[c6137b842b039b9c]::panicking::update_hook::<alloc[6507a21133592989]::boxed::Box<rustc_driver_impl[c8791899bfc93667]::install_ice_hook::{closure#0}>>::{closure#0}
   5:     0x79ee92f381c8 - <alloc::boxed::Box<F,A> as core::ops::function::Fn<Args>>::call::h590bf7f3bbdb3074
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/alloc/src/boxed.rs:2029:9
   6:     0x79ee92f381c8 - std::panicking::rust_panic_with_hook::h7019e963bee9c774
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/std/src/panicking.rs:789:13
   7:     0x79ee92f37e9d - std::panicking::begin_panic_handler::{{closure}}::h1c13df3b8f2c52fb
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/std/src/panicking.rs:649:13
   8:     0x79ee92f35499 - std::sys_common::backtrace::__rust_end_short_backtrace::h5a50cddcfd176d93
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/std/src/sys_common/backtrace.rs:171:18
   9:     0x79ee92f37c07 - rust_begin_unwind
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/std/src/panicking.rs:645:5
  10:     0x79ee92f82516 - core::panicking::panic_fmt::h573473781e8856ca
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/core/src/panicking.rs:72:14
  11:     0x79ee92f825bf - core::panicking::panic::hc1cfefed262285d0
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/core/src/panicking.rs:141:5
  12:     0x79ee92f82299 - core::option::unwrap_failed::h93b1377a7959be3c
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/core/src/option.rs:1984:5
  13:     0x79ee91b6010d - <rustc_metadata[93a521d2dd69104d]::creader::CrateMetadataRef>::def_key
  14:     0x79ee91b5fa0c - <rustc_metadata[93a521d2dd69104d]::creader::CStore as rustc_session[cf42eb386bd1146c]::cstore::CrateStore>::def_path
  15:     0x79ee91b5f8d6 - <rustc_middle[50693f396bf8ba]::ty::context::TyCtxt>::def_path
  16:     0x56010f7e27d0 - rustdoc[d0f82014da61b517]::clean::inline::record_extern_fqn
  17:     0x56010f7f84f3 - rustdoc[d0f82014da61b517]::clean::utils::register_res
  18:     0x56010f9431f5 - <rustdoc[d0f82014da61b517]::passes::collect_intra_doc_links::LinkCollector>::resolve_links
  19:     0x56010f921160 - <rustdoc[d0f82014da61b517]::passes::collect_intra_doc_links::LinkCollector as rustdoc[d0f82014da61b517]::visit::DocVisitor>::visit_inner_recur
  20:     0x56010f8f1934 - rustdoc[d0f82014da61b517]::passes::collect_intra_doc_links::collect_intra_doc_links
  21:     0x56010f8288bd - rustdoc[d0f82014da61b517]::core::run_global_ctxt
  22:     0x56010f72949f - <rustc_middle[50693f396bf8ba]::ty::context::GlobalCtxt>::enter::<rustdoc[d0f82014da61b517]::main_args::{closure#1}::{closure#0}::{closure#0}, core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>>::{closure#0}
  23:     0x56010f733263 - rustc_interface[e1f0298d19bc4b71]::interface::run_compiler::<core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>, rustdoc[d0f82014da61b517]::main_args::{closure#1}>::{closure#0}
  24:     0x56010f6ffc41 - std[c6137b842b039b9c]::sys_common::backtrace::__rust_begin_short_backtrace::<rustc_interface[e1f0298d19bc4b71]::util::run_in_thread_with_globals<rustc_interface[e1f0298d19bc4b71]::util::run_in_thread_pool_with_globals<rustc_interface[e1f0298d19bc4b71]::interface::run_compiler<core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>, rustdoc[d0f82014da61b517]::main_args::{closure#1}>::{closure#0}, core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>>::{closure#0}, core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>>::{closure#0}::{closure#0}, core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>>
  25:     0x56010f73c976 - <<std[c6137b842b039b9c]::thread::Builder>::spawn_unchecked_<rustc_interface[e1f0298d19bc4b71]::util::run_in_thread_with_globals<rustc_interface[e1f0298d19bc4b71]::util::run_in_thread_pool_with_globals<rustc_interface[e1f0298d19bc4b71]::interface::run_compiler<core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>, rustdoc[d0f82014da61b517]::main_args::{closure#1}>::{closure#0}, core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>>::{closure#0}, core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>>::{closure#0}::{closure#0}, core[538b8a97dfb8cd96]::result::Result<(), rustc_span[5c8c1d2776d81648]::ErrorGuaranteed>>::{closure#1} as core[538b8a97dfb8cd96]::ops::function::FnOnce<()>>::call_once::{shim:vtable#0}
  26:     0x79ee92f41989 - <alloc::boxed::Box<F,A> as core::ops::function::FnOnce<Args>>::call_once::h3c29ce2e08aff0f2
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/alloc/src/boxed.rs:2015:9
  27:     0x79ee92f41989 - <alloc::boxed::Box<F,A> as core::ops::function::FnOnce<Args>>::call_once::hfd7de007db754644
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/alloc/src/boxed.rs:2015:9
  28:     0x79ee92f41989 - std::sys::pal::unix::thread::Thread::new::thread_start::hd114416927dd9b45
                               at /rustc/2f090c30ddd6b3bbe5c81c087579a5166e7c7278/library/std/src/sys/pal/unix/thread.rs:108:17
  29:     0x79ee8c77955a - <unknown>
  30:     0x79ee8c7f6a3c - <unknown>
  31:                0x0 - <unknown>


rustc version: 1.79.0-nightly (2f090c30d 2024-03-23)
platform: x86_64-unknown-linux-gnu

query stack during panic:
end of query stack
```

</p>
</details>

