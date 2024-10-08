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

![bazel](https://github.com/LKreutzer/bazel-workshop/blob/main/media/2024-08-16-readme.gif)

## Prerequisites

- Install Bazel: [bazel.build/install](https://bazel.build/install)
  - Recommended method via Bazelisk:
    - `cd /usr/sbin`
    - `wget --progress=dot:giga https://github.com/bazelbuild/bazelisk/releases/download/v1.10.0/bazelisk-linux-amd64`
    - `chmod +x bazelisk-linux-amd64`
    - `ln -s /usr/sbin/bazelisk-linux-amd64 /usr/sbin/bazel`
    - Use a `.bazelversion` file to set the Bazel version for each repository.
  - Verify that all participants use the same version!
  - Alternatively: use a build environment, e.g. Docker container
- (Optional) Clone examples repository: [github.com/bazelbuild/examples](https://github.com/bazelbuild/examples)

## What do you want to learn in this workshop?

- What do you already know about Bazel?
- More theory or more hands-on?
- Which advanced topics are you interested in? E.g.:
  - distributed builds (remote caching, remote execution)
  - query language
  - build file generator (Gazelle)
  - bazel-diff
  - custom Bazel rules (with unit tests)

## Resources to find general information

- Project website: [bazel.build](https://bazel.build/)
- [Official Bazel reference](https://bazel.build/reference)
- [`bazel help`](https://bazel.build/docs/user-manual#help) command
- Bazel [glossary](https://bazel.build/reference/glossary)
- [FAQ](https://bazel.build/about/faq)
- Official Bazel slack: [slack.bazel.build](https://slack.bazel.build/)
- Source code [github.com/bazelbuild](https://github.com/bazelbuild)

## What is Bazel?

[Bazel](https://bazel.build/about) is an open-source build system that is intended for multi-language mono-repos. Bazel is focused on fast, scalable, parallel and reproducible builds. A core strength of Bazel is an extensive [caching framework](#caching), based on a strict and fine grained build graph.

### Some benefits of Bazel:

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

### Possible limitations:

- steep learning curve for maintainers
- running services and integration tests, that need extended system access, goes against the philosophy of sand-boxing
- languages that are (yet) without rule sets

## Bazel basics

### Starlark

The [Starlark configuration language](https://github.com/bazelbuild/starlark) is a deterministic, hermetic and parallelizable Python dialect.

#### Buildifier Starlark formatter

To ensure the standardized formatting of all `BUILD.bazel` and `.bzl` files the Starlark formatter [buildifier](https://github.com/bazelbuild/buildtools/tree/master/buildifier) can be used.

### File structure

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
- [`BUILD.bazel`](https://bazel.build/concepts/build-files): A file, e.g. `lte/gateway/python/magma/mobilityd/BUILD.bazel`, in which sources are defined in [targets](https://bazel.build/concepts/build-ref#targets), e.g. binaries, libraries and tests. A `BUILD.bazel` file defines a [Bazel package](https://bazel.build/concepts/build-ref#packages).
- `bazel/`: Centralized folder for custom Bazel definitions and configuration of external sources. This folder could also be called something like `tools`, this depends on the project. The same is true for all sub-folders mentioned here.
- `bazel/scripts/`: Centralized folder for Bash wrapper scripts for Bazel, e.g. for running the [Starlark formatter](#starlark-formatter) or [Bazel-diff](#bazel-diff).
- `bazel/bazelrcs/name.bazelrc`: Configuration of flags for commands in different environments.
- `bazel/name.bzl`
    - Custom Bazel code - these are custom [Bazel rules](https://bazel.build/extending/rules), configuration of the tool chain and external dependencies.
    - In this example, `cpp`, `go` and `python_repositories.bzl`, specify external repositories. These can be e.g. local repositories (folders in the environment) or git repositories.
- `bazel/external/name.BUILD`: `.BUILD` files for external repositories that are not bazelified.

### Build files

- [General info](https://bazel.build/concepts/build-files)
- [Build file style guide](https://bazel.build/build/style-guide)

### Targets and rules

The content of the build files are [Targets](https://bazel.build/concepts/build-ref#targets). There are three basic [types of rules](https://bazel.build/concepts/build-files#types-of-build-rules): `*_library`, `*_binary` and `*_test`.

General infos on rules can be found in the [Bazel docs](https://bazel.build/extending/rules).

Supported languages and formats are imported from:

- [Generic rules (genrule)](https://bazel.build/reference/be/general#genrule)
- [List of some rule repositories](https://github.com/orgs/bazelbuild/repositories?language=&page=1&q=rules&sort=&type=all), this list is not comprehensive
- [C/C++](https://github.com/bazelbuild/rules_cc)
- [Go](https://github.com/bazelbuild/rules_go)
- [Python](https://github.com/bazelbuild/rules_python)
- [Java](https://github.com/bazelbuild/rules_java)
  - [jvm](https://github.com/bazelbuild/rules_jvm_external)
- Javascript/Typescript:
  - [rules_ts](https://github.com/aspect-build/rules_ts)
  - [nodejs (unmaintained)](https://github.com/bazelbuild/rules_nodejs)
  - [rules_typescript](https://bazelbuild.github.io/rules_nodejs/TypeScript.html), [see also this subfolder](https://github.com/bazelbuild/rules_nodejs/tree/3.x/third_party/github.com/bazelbuild/rules_typescript)
  - [Typescript (moved to the above link)](https://github.com/bazelbuild/rules_typescript)
- [Bash](https://github.com/tweag/rules_sh)
- [Docker](https://github.com/bazelbuild/rules_docker)
- [Rust](https://github.com/bazelbuild/rules_docker)
- [Apple platform](https://github.com/bazelbuild/rules_apple)
- [Haskell](https://github.com/tweag/rules_haskell)
- [Packages (deb, rpm, zip, ...)](https://github.com/bazelbuild/rules_pkg)
- ... and many more.

### Commands

The Bazel commands are structured around verbs like build, test or query.

See also:
- complete list of [available commands](https://bazel.build/run/build#available-commands).
- [build docs](https://bazel.build/docs/build)
- [Specifying targets on the command line](https://bazel.build/run/build)
- [user manual](https://bazel.build/docs/user-manual)
- [command line reference](https://bazel.build/reference/command-line-reference)

The `bazel help` and `bazel info` commands can also be helpful.


#### [Build phases](https://bazel.build/run/build#build-phases)

- Loading phase
- Analysis phase
- Execution phase

### Caching

Bazel has a very extensive and granular system of caching intermediate build outputs, artifacts and even test results. Builds that have many cache hits are much faster than builds without cache hits. The number of cache hits is reported at the end of each build, e.g. as `INFO: 4449 processes: 703 disk cache hit, 2377 internal, 1367 processwrapper-sandbox, 2 worker.`.

Whenever cache-able results are produced, a local cache entry is generated in the disk cache. Some of the cache locations can be found by running `bazel info` and looking e.g. at the `repository_cache` entry.


### Dependencies

- [External dependencies](https://bazel.build/external/overview)

### Inputs and outputs

A note on build environments and containers...

- [Sand-boxing](https://bazel.build/docs/sandboxing)

- [Bazel output directory](https://bazel.build/remote/output-directories)
- The [`bazel clean`](https://bazel.build/docs/user-manual#cleaning-build-outputs) command

## Advanced topics

### Querying

- [Dependency graph and loading phase](https://bazel.build/docs/user-manual#querying-dependency-graph)
  - [`bazel query`](https://bazel.build/docs/query-how-to)
- [Action graph](https://bazel.build/docs/user-manual#aquery)
  - [aquery](https://bazel.build/docs/aquery)
- [cquery and analysis phase](https://bazel.build/query/cquery)

### [Tinder/bazel-diff](https://github.com/Tinder/bazel-diff)

- Target diff tool
- Only build/test what is really needed
- Use a wrapper script, e.g. [like this](https://github.com/magma/magma/blob/e1a45b343131b3853cb10d4470f5628b03b2785e/bazel/scripts/bazel_diff.sh).

### Remote caching
- See https://bazel.build/docs/remote-caching
  - Use e.g. Google Cloud Platform
- [buchgr/bazel-remote](https://github.com/buchgr/bazel-remote)
- Managed remote caching services, e.g. [buildbuddy](https://www.buildbuddy.io/)

### CI with Bazel

- Workflow setup
- Combining advanced topics like remote caching, bazel-diff, ...

### [Bazel Gazelle](https://github.com/bazelbuild/bazel-gazelle)

- Build file generator
- Natively supports Go and Protobuf

### Remote execution

- (difficult: is there a free service to try it?)

### Custom rules

Writing your own rules/extending Bazel
  - See e.g.
    - https://bazel.build/rules/rules-tutorial
    - https://bazel.build/extending/rules
  - Danger zone!
    - This is where most bugs are coming from
    - Careful to keep it efficient and safe

#### Starlark tests

  - https://github.com/bazelbuild/rules_testing
