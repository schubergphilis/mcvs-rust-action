# GitHub workflows

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
| token                                       |         | GitHub token, used to upload release assets                                         |

<!-- markdownlint-enable MD013 -->

Note: if an **x** is registered in the Default column, refer to the
[action.yml](../action.yml) for the corresponding value.

## Releases

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
