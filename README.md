# ![](imgs/default64.png)Firefox for windows 7![](imgs/windows.png)
[Firefox](https://firefox.com/) is a fast, reliable and private web browser from the non-profit [Mozilla organization](https://mozilla.org/).

Firefox 115 is the last version to support Windows 7. This repository includes patches that enable Firefox to run on Windows 7 beyond the official end-of-support date, allowing users to continue using the latest features and improvements from Firefox.

**Firefox is a Trademark of The Mozilla Foundation.**

## Platform Support

Windows 7.

![](imgs/snapshot.png)

To replace the browser icon with the Fox icon, see: https://github.com/e3kskoy7wqk/Firefox-for-windows-7/discussions/110

## Building

1. Bootstrap a copy of the Firefox source code

   ```bat
   git clone https://github.com/mozilla-firefox/firefox.git
   ```

   This repository contains patches for specific versions of Firefox, stored in directories named `<DATE>.<commit_id>`. Here, `commit_id` is the commit ID of Firefox, and `DATE` is purely for File Explorer.

   ```bat
   cd firefox
   git checkout <commit_id>
   ./mach bootstrap
   ```

2. Building [Rust](https://github.com/rust-lang/rust) on Windows MSVC

   To ensure that `rustc` always generates programs compatible with Windows 7, Rust needs to be patched. This repository contains files named `Rust<rust_version>win7.diff` for this purpose, where `rust_version` is the version of Rust used to build Firefox, as specified [here](https://github.com/mozilla-firefox/firefox/blob/main/docs/writing-rust-code/update-policy.md). For detailed instructions on how to proceed, please refer to https://github.com/e3kskoy7wqk/Firefox-for-windows-7/issues/75.

   - Clone the source with `git`

     ```bat
     git clone https://github.com/rust-lang/rust.git
     cd rust
     git checkout <rust_version>
     ```

   - Apply patch

     ```bat
     git apply --verbose --ignore-space-change Rust<rust_version>win7.diff
     ```

   - If you want to support Windows Vista, you also need to apply Rust<rust_version>vista.diff:

     ```bat
     git apply --verbose --ignore-space-change Rust<rust_version>vista.diff
     ```

   - Build

     ```bat
     python x.py build --target i686-pc-windows-msvc,x86_64-pc-windows-msvc
     python x.py build library --target i686-pc-windows-msvc,x86_64-pc-windows-msvc
     ```

3. Compiling [Cargo](https://github.com/rust-lang/cargo) from Source

   - Clone the source with `git`

     ```bat
     git clone https://github.com/rust-lang/cargo.git
     cd cargo
     ```

   - With `cargo` already installed by ./mach bootstrap, you can simply run:

     ```bat
     cargo build --release
     ```

4. Apply patches

   Navigate to the `<DATE>.<commit_id>` directory, modify the `REPO_PATH` variable in `apply_patches.bat` to the path of the Firefox source, and then double-click it to apply the patches.

5. Build Firefox

   - Add our custom-built `rustc.exe` and `cargo.exe` to your PATH, delete or rename `%USERPROFILE%\.rustup` and `%USERPROFILE%\.cargo`, and remove any other instances of `rustc.exe` and `cargo.exe` from your PATH.

   - mozconfig for i686-pc-windows-msvc:

     ```bash
     ac_add_options MOZ_TELEMETRY_REPORTING=1
     ac_add_options --with-redist=<CRT_LOCATION>
     export WIN_UCRT_REDIST_DIR=<UCRT_LOCATION>
     ac_add_options --disable-crashreporter
     ac_add_options --allow-addon-sideload
     ac_add_options --with-unsigned-addon-scopes=app,system
     ac_add_options --disable-sandbox
     ac_add_options --target=i686-pc-windows-msvc
     ac_add_options --enable-optimize
     ac_add_options --disable-tests
     ac_add_options MOZ_PGO=1
     ac_add_options --enable-clang-plugin
     export MOZ_LTO=cross
     ac_add_options --enable-js-shell
     ac_add_options --enable-rust-simd
     ```
   
   - mozconfig for x86_64-pc-windows-msvc:
   
     ```bash
     ac_add_options MOZ_TELEMETRY_REPORTING=1
     ac_add_options --with-redist=<CRT_LOCATION>
     export WIN_UCRT_REDIST_DIR=<UCRT_LOCATION>
     ac_add_options --disable-crashreporter
     ac_add_options --allow-addon-sideload
     ac_add_options --with-unsigned-addon-scopes=app,system
     ac_add_options --target=x86_64-pc-windows-msvc
     ac_add_options --enable-optimize
     ac_add_options --disable-tests
     ac_add_options MOZ_PGO=1
     ac_add_options --enable-clang-plugin
     export MOZ_LTO=cross
     ac_add_options --enable-js-shell
     ac_add_options --enable-rust-simd
     ```
   
   - ```bash
     ./mach build
     ./mach package
     ./mach build installers-$AB_CD
     ```
