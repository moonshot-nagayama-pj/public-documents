# Development environment

This document assumes prior knowledge of Unix-like operating systems and Python. If anything is unclear, please ask.

## Git

[We have a guide on how to configure and use Git](git-workflow.md). Please read it.

## EditorConfig

We use [EditorConfig](https://editorconfig.org/) to manage formatting and style. Please make sure that your IDE or editor supports it.

## direnv

[direnv](https://direnv.net/) is not required, but is highly recommended when working with projects on the command-line. In particular, we often use it to help manage Python virtual environments.

Generally, we do not use the built-in `python` layout, but instead extend direnv to use `uv`. See below.

## Modern version of Bash

Some of our projects use Bash scripts for automation.

By default, MacOS has a very old version of Bash. This is because the Bash project changed their license from GPL 2.0 to GPL 3.0 in 2009. Apple's lawyers refuse to allow MacOS to include GPL 3.0 software. Bash has added new features and changed some behaviors since 2009. In order to ensure that our scripts run correctly, it is important that we all run modern versions of Bash.

If you are using MacOS, you can install a modern version of Bash using [Homebrew](https://brew.sh/) or [MacPorts](https://ports.macports.org/port/bash/). Please make sure that it is on your `$PATH`.

If you are using Linux, you do not need to do anything.

## Rust

Please install Rust by following [the instructions on the official homepage](https://www.rust-lang.org/).

## uv

[`uv`](https://docs.astral.sh/uv/) is a package and project manager for Python. If you are working with any of our Python projects, install `uv`.

You should [extend direnv to work with uv projects](https://github.com/direnv/direnv/wiki/Python#uv).

## clang

[clang](https://clang.llvm.org/) should be installed on your system. We use its formatting tool to format [Protocol Buffers](https://protobuf.dev/) files.

In general, clang should be installed through your system package manager (MacPorts, Homebrew, `apt`, etc.). Do not download it directly from the clang homepage.

## ShellCheck, shfmt

[ShellCheck](https://www.shellcheck.net/) is a static analysis tool for shell scripts.

[shfmt](https://github.com/mvdan/sh/), part of the sh package, is a formatter for shell scripts.

Both of these will run as part of our static analysis suite; it is important that you be able to run them locally.
