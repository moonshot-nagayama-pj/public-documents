# Development environment

This document assumes prior knowledge of Unix-like operating systems, Python, and Rust. If anything is unclear, please ask.

## Essential build tools

On MacOS, be sure that you have either installed Xcode or the smaller [Xcode CLI package](https://developer.apple.com/documentation/xcode/installing-the-command-line-tools).

On Ubuntu, be sure that you have installed the `build-essential` package.

## Git

[We have a guide on how to configure and use Git](git-workflow.md). Please read it.

## EditorConfig

We use [EditorConfig](https://editorconfig.org/) to manage formatting and style. Please make sure that your IDE or editor supports it. Specifically, if you use VSCode, **be sure to install the VSCode plugin for EditorConfig**.

## Modern version of Bash

Some of our projects use Bash scripts for automation.

By default, MacOS has a very old version of Bash. This is because the Bash project changed their license from GPL 2.0 to GPL 3.0 in 2009. Apple's lawyers refuse to allow MacOS to include GPL 3.0 software. Bash has added new features and changed some behaviors since 2009. In order to ensure that our scripts run correctly, it is important that we all run modern versions of Bash.

If you are using MacOS, you can install a modern version of Bash using [Homebrew](https://brew.sh/) or [MacPorts](https://ports.macports.org/port/bash/). Please make sure that it is on your `$PATH`.

If you are using Linux, you do not need to do anything.

## Rust

Please install Rust by following [the instructions on the official homepage](https://www.rust-lang.org/).

## uv

[`uv`](https://docs.astral.sh/uv/) is a package and project manager for Python. Many of our Rust projects also use `uv` to manage Python tools we use during validation checks; make sure to install `uv` even if you are not working on Python projects.

## clang

[clang](https://clang.llvm.org/) should be installed on your system. We use its formatting tool to format [Protocol Buffers](https://protobuf.dev/) files. It may also be necessary for other gRPC-related build tasks (in which case you may need to install

In order to ensure consistent formatting, we require everyone to use clang version 20. In general, clang should be installed through your system package manager (MacPorts, Homebrew, `apt`, etc.). Do not download it directly from the clang homepage.

* In Ubuntu: `sudo apt install clang-20 clang-format-20`.
* In Homebrew: `sudo brew install llvm@20`

## ShellCheck, shfmt

[ShellCheck](https://www.shellcheck.net/) is a static analysis tool for shell scripts.

[shfmt](https://github.com/mvdan/sh/), part of the `sh` suite of tools, is a formatter for shell scripts. Note that this is available from a Ubuntu or Homebrew package called `shfmt`, not `sh`.

Both of these will run as part of our static analysis suite; it is important that you be able to run them locally.

## direnv

[direnv](https://direnv.net/) is not required, but is highly recommended when working with projects on the command-line. In particular, we often use it to help manage Python virtual environments.

Generally, we do not use the built-in `python` layout, but instead [extend direnv to use `uv`](https://github.com/direnv/direnv/wiki/Python#uv).

## Other requirements

Some libraries have additional requirements; please be sure to check project `README.md` files for details before getting started.
