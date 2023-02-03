
<!-- 

This file contains parts derived from [1].

[1] https://github.com/magma/magma/blob/e6d5a5a02e0a15b73a733deda664d348a5724d3d/docs/readmes/bazel/agw_with_bazel.md

    Licensed under BSD 3-Clause License

    For Magma software

    Copyright (c) 2020, The Magma Authors

    Redistribution and use in source and binary forms, with or without modification,
    are permitted provided that the following conditions are met:

    1. Redistributions of source code must retain the above copyright notice, this
    list of conditions and the following disclaimer.

    2. Redistributions in binary form must reproduce the above copyright notice,
    this list of conditions and the following disclaimer in the documentation and/or
    other materials provided with the distribution.

    3. Neither the name of the copyright holder nor the names of its contributors
    may be used to endorse or promote products derived from this software without
    specific prior written permission.

    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
    ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
    WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
    DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR
    ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
    (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
    LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
    ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
    (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
    SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

 -->

# bazel-workshop

## Prerequisites

- Install Bazel: https://bazel.build/install
- (Optional) Clone examples repository: https://github.com/bazelbuild/examples

## Resources to find general information

- Project website [bazel.build](https://bazel.build/).
- [Official Bazel reference](https://bazel.build/reference)
- [`bazel help`](https://bazel.build/docs/user-manual#help) command.
- Bazel [glossary](https://bazel.build/reference/glossary).
- [FAQ](https://bazel.build/about/faq).
- Source code [github.com/bazelbuild](https://github.com/bazelbuild).

## What is Bazel?

[Bazel](https://bazel.build/about) is an open-source build system that is intended for multi-language mono-repos. Bazel is focused on fast, scalable, parallel and reproducible builds. A core strength of Bazel is an extensive [caching framework](#caching) based on a strict and fine grained build graph.

Reasons to use Bazel:

- "reproducible" builds
- hermetic (sand-boxed) execution with strictly defined in- and outputs 
- granular targets and dependencies, no circular dependencies allowed, strict visibility control
- incremental builds
- (remote) caching
- parallelization and scalability, remote execution
- free open-source project, with active community
- multi-language, multi platform, extensible system

See also the [intro](https://bazel.build/about/intro) and [Bazel vision](https://bazel.build/about/vision) pages.

Possible limitations:

- running services and integration tests with extended system access requirements
- languages (yet) without rule sets

## Bazel basics

## Basic bazel setup

- [Bazel output directory](https://bazel.build/remote/output-directories)

Basic files:
- [`BUILD.bazel`](https://bazel.build/concepts/build-files)
    - "A directory within the workspace that contains a BUILD file is a package."
- `WORKSPACE.bazel`
  - "Identifies the directory and its contents as a Bazel workspace and lives at the root of the project's directory structure"
- [`.bazelrc`](https://bazel.build/docs/bazelrc)

Basic commands:
  - `bazel help`
  - See also:
    -  https://bazel.build/docs/build
    -  https://bazel.build/docs/user-manual
    -  https://bazel.build/reference/command-line-reference
  - Buildifier/formatter
    - See https://bazel.build/rules/build-style

### `bazel build`

- Bazel targets
- [build rules](https://bazel.build/concepts/build-files)
    - `_binary`
    - `_test`
    - `_library`
    - [`genrule`](https://bazel.build/reference/be/general)
- Sources
- (External) dependencies
- Visibility
- Executables
- `bazel clean`

### `bazel test`
 - Command line options
 - Flaky test mechanics

### `bazel run`


## Advanced bazel topics

### `bazel coverage`
- Unittest coverage data

### [`bazel query`](https://bazel.build/docs/query-how-to)
- Dependency graph
- [aquery](https://bazel.build/docs/aquery)

### (Remote) caching
- See https://bazel.build/docs/remote-caching
- Use e.g. [buildbuddy](https://www.buildbuddy.io/)


## Exercises:
- Exercise 1:
  - Setup a simple project with a genrule that concatenates two files
  - Build the genrule target
  - Find the build output
  - Clean the build output with bazel
- Exercise 2:
  - Setup a C++ project with a unittest
  - Build:
    - Build a single target
    - Build all targets
    - Find the binary build outputs
  - Test
    - Run the unittest
    - Run it again without test caching
- Exercise 3:
  - Setup a Project with at least one C++ and at least one Python target
  - Use at least one external dependency
  - Add a C++ and a Python unittest
  - Set the visibility of all targets to be as restrictive as possible
  - Add an assert statement to a random number to one of the tests to simulate a flaky test
  - Run all tests 100 times and detect flaky tests
  - Mark the flaky test targets as flaky
    - How does this influence the test behaviour?
- Exercise 4:
  - Add a `_binary` target
  - Run the `_binary` target with bazel
  - Add a `.bazelrc` configuration file
    - Change the appearance of the build output
    - Define a `config` group that has maximal command line output for `bazel test`
    - If a unit test fails it should be retried for 2 times
- Exercise 5:
  - Make all bazel commands use remote caching
    - Verify that the remote cache is used
    - How does the remote caching influence the build time?
  - Visualise the dependency graph of the project
  - Create a directory in the project that is ignored by bazel
  - Calculate the test coverage

TBC...
