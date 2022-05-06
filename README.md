# bazel-workshop

## Bazel basics

Ideas:
- [Hermiticity](https://bazel.build/concepts/hermeticity)
    - Encapsulation
    - Sandboxing
- Reproducibility
- Non-circularity
- Caching/content digest
- Starlark
  
Websites:
- Docs: https://bazel.build/
- Github: https://github.com/bazelbuild

## Basic bazel setup

- Install bazel
- (Use examples from docs? `git clone https://github.com/bazelbuild/examples` )

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
  - Make all bazel commands use remote caching
    - Verify that the remote cache is used
    - How does the remote caching influence the build time?
  - Visualise the dependency graph of the project
  - Create a directory in the project that is ignored by bazel
  - Calculate the test coverage

TBC...
