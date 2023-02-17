
# Simple example exercises:

These examples are only suggestions - adapt to the interests and skills of the workshop participants.

- Exercise 0:
  - Write a Hello world program and bazelify it
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
- Exercise 6:
  - Generate a dependency graph of a target
    - Exclude external dependencies

## CI exercises

- Exercise 1:
  - Set up CI workflows for a project that cover the following aspects:
    - Multi-lingual unit testing
      - Use remote caching
      - Use Bazel-diff
      - Run all tests with language X with a different test config
    - Building artifacts with remote caching
      - Publish the build artifacts 
    - Build file format checking


TBC...
