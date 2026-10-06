# MCVS Rust Action

[![GitHub release](https://img.shields.io/github/v/release/schubergphilis/mcvs-rust-action)](https://github.com/schubergphilis/mcvs-rust-action/releases)
[![License](https://img.shields.io/github/license/schubergphilis/mcvs-rust-action)](LICENSE)

The Mission Critical Vulnerability Scanner (MCVS) Rust Action repository is a
collection of standardized tools to ensure a certain level of quality of a
project with Rust code. It is the Rust counterpart of the
[mcvs-golang-action](https://github.com/schubergphilis/mcvs-golang-action).

## Github Action

The [GitHub Action](https://github.com/features/actions) in this repository
consists of the following steps:

- Install the Rust toolchain that is defined in the project
  `rust-toolchain.toml`, or stable if absent.
- Verify that `Cargo.lock` is up to date (every cargo command runs with
  `--locked`).
- Code security scanning of `Cargo.lock` with
  [osv-scanner](https://github.com/google/osv-scanner) and suppression of
  certain vulnerabilities for a limited time, see
  [docs/osv-scanner.md](docs/osv-scanner.md). Optionally
  [Grype](https://github.com/anchore/grype).
- Linting with `rustfmt` and `clippy` (warnings are errors).
- Unit tests.
- Integration tests, i.e. the tests in the `tests` directory.
- Code coverage with
  [cargo-llvm-cov](https://github.com/taiki-e/cargo-llvm-cov).
- Optionally building a binary and attaching it to a GitHub release.

Note: there is an [internal action](.github/workflows/package-version-updater.yml)
that will update package versions that cannot be updated by Dependabot.

## Taskfile

This repository offers a `./build/task.yml` with standard
[Task](https://taskfile.dev/) tasks, so that the same checks can be run
locally as in the pipeline.

### Usage

Create a `Taskfile.yml` with the following content:

```yml
---
version: 3

vars:
  REMOTE_URL: https://raw.githubusercontent.com
  REMOTE_URL_REF: v0.1.0
  REMOTE_URL_REPO: schubergphilis/mcvs-rust-action

includes:
  remote: >-
    {{.REMOTE_URL}}/{{.REMOTE_URL_REPO}}/{{.REMOTE_URL_REF}}/build/task.yml
```

and run e.g.:

```zsh
task remote:lint
task remote:test
task remote:test-integration
task remote:coverage
task remote:osv-scanner
```

Use `task --list-all` to get a list of all available tasks.

### Configuration

The following variables can be overridden:

| Variable               | Description                                                                                              |
| :--------------------- | :------------------------------------------------------------------------------------------------------- |
| `CODE_COVERAGE_STRICT` | Enables or disables strict enforcement of setting the minimum coverage to the maximum observed coverage. |

```yml
---
includes:
  remote:
    taskfile: >-
      {{.REMOTE_URL}}/{{.REMOTE_URL_REPO}}/{{.REMOTE_URL_REF}}/build/task.yml
    vars:
      CODE_COVERAGE_STRICT: "false"
```

## GitHub

Create a `.github/workflows/rust.yml` file. Note that the job has to be named
`mcvs-rust-action`, as `task remote:coverage` reads the expected coverage from
it when run locally.

```yml
---
name: rust
"on": pull_request
permissions:
  contents: read
  packages: read
jobs:
  mcvs-rust-action:
    strategy:
      matrix:
        args:
          - testing-type: coverage
          - testing-type: integration
          - testing-type: lint
          - testing-type: security-cargo
          - testing-type: security-grype
          - testing-type: unit
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v5
      - uses: schubergphilis/mcvs-rust-action@v0.1.0
        with:
          code-coverage-expected: 80
          testing-type: ${{ matrix.args.testing-type }}
          token: ${{ secrets.GITHUB_TOKEN }}
```

<!-- markdownlint-disable MD013 -->

| Option                                      | Default | Description                                                                         |
| :------------------------------------------ | :------ | :---------------------------------------------------------------------------------- |
| code-coverage-expected                      | x       | Expected line coverage percentage, one decimal                                      |
| github-token-for-downloading-private-crates |         | Token to fetch crates from private GitHub repositories                              |
| grype-version                               |         | Version of Grype                                                                    |
| release-application-name                    |         | Name of the binary in `Cargo.toml` to build and release                             |
| release-target                              | x       | Target triple, e.g. `x86_64-unknown-linux-gnu`                                      |
| task-install                                | x       | Whether to install Task (`yes` or `no`)                                             |
| task-version                                | x       | Version of Task                                                                     |
| testing-type                                |         | `coverage`, `integration`, `lint`, `security-cargo`, `security-grype` or `unit`     |
| token                                       |         | GitHub token, used by Grype and to upload release assets                            |

<!-- markdownlint-enable MD013 -->

Note: if an **x** is registered in the Default column, refer to the
[action.yml](action.yml) for the corresponding value.

### Releases

```yml
---
name: rust-releases
"on": push
permissions:
  contents: write
  packages: read
jobs:
  mcvs-rust-action:
    strategy:
      matrix:
        args:
          - os: ubuntu-24.04
            release-target: x86_64-unknown-linux-gnu
          - os: ubuntu-24.04-arm
            release-target: aarch64-unknown-linux-gnu
          - os: macos-15
            release-target: aarch64-apple-darwin
    runs-on: ${{ matrix.args.os }}
    steps:
      - uses: actions/checkout@v5
      - uses: schubergphilis/mcvs-rust-action@v0.1.0
        with:
          release-application-name: some-app
          release-target: ${{ matrix.args.release-target }}
          token: ${{ secrets.GITHUB_TOKEN }}
```

The binary is built with a plain `cargo build`, so run each target on a runner
of the matching OS and architecture. The asset is uploaded when a tag is
pushed.
