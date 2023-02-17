:warning: These notes are still a work in progress.

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

Some benefits of Bazel:

  - multi-language, multi platform, extensible system
    - ideal for multilingual mono-repos
    - can support multiple architectures, configurations etc.
  - free open-source project, with active community
  - clear file and target structure
  - reproducible builds
  - hermetic (sand-boxed) execution with strictly defined in- and outputs
  - build [profiling](https://bazel.build/rules/performance#performance-profiling) and metrics
    - easy to get information, e.g. about performance bottle necks for further optimization 
  - powerful [query language](https://bazel.build/query/quickstart) ([full query guide](https://bazel.build/query/guide))
    - easy to analyze service architecture, create dependency graphs etc.
  - granular targets and dependencies, no circular dependencies allowed, strict visibility control
    - helps to avoid regression and keep services separated
    - know which unit tests need to run for any change
    - only build and test what is really needed
  - incremental builds
    - faster iteration and development
  - [(remote) caching](https://bazel.build/remote/caching)
    - faster builds and CI
  - parallelization and scalability, remote execution, [distributed builds](https://bazel.build/basics/distributed-builds)


See also the [intro](https://bazel.build/about/intro) and [Bazel vision](https://bazel.build/about/vision) pages.

Possible limitations:

- running services and integration tests with extended system access requirements
- languages (yet) without rule sets

## Bazel basics

### Files


The basic structure of the Bazel files in a repository could, for example, look like this:

```bash
.
├── WORKSPACE.bazel
├── .bazelrc
├── BUILD.bazel
├── bazel
│   ├── bazelrcs
│   │   ├── vm.bazelrc
│   │   └── ...
│   ├── external
│   │   ├── BUILD.bazel
│   │   ├── sentry_native.BUILD
│   │   ├── requirements.in
│   │   ├── requirements.txt
│   │   └── ...
│   ├── scripts
│   │   ├── bazel_diff.sh
│   │   ├── run_buildifier.sh
│   │   └── ...
│   ├── BUILD.bazel
│   ├── cpp_repositories.bzl
│   ├── go_repositories.bzl
│   ├── python_repositories.bzl
│   ├── python_swagger.bzl
│   └── ...
├── lte
│   ├── gateway
│   │   └── python
│   │       ├── magma
│   │       │   ├── mobilityd
│   │       │   │   ├── tests
│   │       │   │   │   └── BUILD.bazel
│   │       │   │   └── BUILD.bazel
│   │       │   └── policydb
│   │       │       ├── servicers
│   │       │       │   └── BUILD.bazel
│   │       │       ├── tests
│   │       │       │   └── BUILD.bazel
│   │       │       └── BUILD.bazel
│   │       └── BUILD.bazel
│   ├── protos
│   │   ├── oai
│   │   │   └── BUILD.bazel
│   │   └── BUILD.bazel
│   └── swagger
│       └── BUILD.bazel
.
```

- [`WORKSPACE.bazel`](https://bazel.build/concepts/build-ref#workspace): Defines a Bazel project. Configuration of [Bazel rules](https://bazel.build/extending/rules) to be used in the project.
- [`.bazelrc`](https://bazel.build/run/bazelrc): General configuration of flags for commands.
- [`BUILD.bazel`](https://bazel.build/concepts/build-files): A file, e.g. `lte/gateway/python/magma/mobilityd/BUILD.bazel`, in which sources are defined in binaries, libraries and tests. A `BUILD.bazel` file defines a [Bazel package](https://bazel.build/concepts/build-ref#packages).
- `bazel/`: Centralized folder for custom Bazel definitions and configuration of external sources.
- `bazel/scripts/`: Centralized folder for Bash wrapper scripts for Bazel, e.g. for running the [Starlark formatter](#starlark-formatter), [Bazel-diff](#bazel-diff) or the [LTE integration tests](#lte-integration-tests).
- `bazel/bazelrcs/name.bazelrc`: Configuration of flags for commands in different environments.
- `bazel/name.bzl`
    - Custom Bazel code - these are custom [Bazel rules](https://bazel.build/extending/rules), configuration of the tool chain and external dependencies.
    - In `cpp`, `go` and `python_repositories.bzl`, external repositories are specified. These can be local repositories (folder in the environment) or git repositories.
- `bazel/external/name.BUILD`: `.BUILD` files for external repositories that are not bazelified.

### Starlark

#### Buildifier Starlark formatter

To ensure the standardized formatting of all `BUILD.bazel` and `.bzl` files the Starlark formatter [buildifier](https://github.com/bazelbuild/buildtools/tree/master/buildifier) can be used.

### Caching

Bazel has a very extensive and granular system of caching intermediate build outputs, artifacts and even test results. Builds that have many cache hits are much faster than builds without cache hits. The number of cache hits is reported at the end of each build, e.g. as `INFO: 4449 processes: 703 disk cache hit, 2377 internal, 1367 processwrapper-sandbox, 2 worker.`.

Whenever cacheable results are produced, a local cache entry is generated in the disk cache. Some of the cache locations can be found by running `bazel info` and looking e.g. at the `repository_cache` entry.

### Targets

### Rules


rules https://github.com/orgs/bazelbuild/repositories?language=&page=1&q=rules&sort=&type=all
    cc https://github.com/bazelbuild/rules_cc
    go https://github.com/bazelbuild/rules_go
    python https://github.com/bazelbuild/rules_python
    java https://github.com/bazelbuild/rules_java
    javascript/typescript
          https://bazelbuild.github.io/rules_nodejs/TypeScript.html
          https://github.com/bazelbuild/rules_nodejs
          https://github.com/bazelbuild/rules_nodejs/tree/3.x/third_party/github.com/bazelbuild/rules_typescript
          https://github.com/bazelbuild/rules_typescript
    ...
    generic rules, e.g. genrule https://bazel.build/reference/be/general#genrule
    ..library
    ..test
    ..binary

### Commands

Basic commands:
  - `bazel help`
  - See also:
    -  https://bazel.build/docs/build
    -  https://bazel.build/docs/user-manual
    -  https://bazel.build/reference/command-line-reference
  - Buildifier/formatter
    - See https://bazel.build/rules/build-style

    help
    info
    build
        phases of building
            loading
            analysis
            execution

        test
          - Flaky test mechanics
        run
        query
        coverage
        options

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
   
### Dependencies

### Inputs and outputs

A note on build environments and containers...

[Bazel output directory](https://bazel.build/remote/output-directories)

## Advanced topics

### Querying
[`bazel query`](https://bazel.build/docs/query-how-to)
-  Dependency graph
- aquery
  - [aquery](https://bazel.build/docs/aquery)
- cquery

### bazel-diff


### Remote caching
- See https://bazel.build/docs/remote-caching
- bazel-remote
- build-buddy
  - Use e.g. [buildbuddy](https://www.buildbuddy.io/)

### CI with Bazel

### Bazel Gazelle
  - Natively supports Go and Protobuf


### Remote execution (difficult - is there a free service?)

### Custom rules

Writing your own rules/extending Bazel
  - See e.g. https://bazel.build/rules/rules-tutorial
  - Danger zone
    - This is where most bugs are coming from
    - Careful to keep it efficient and safe

#### Starlark tests

  - https://github.com/bazelbuild/rules_testing


## Simple example exercises:
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
