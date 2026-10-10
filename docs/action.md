# GitHub Action

The [GitHub Action](https://github.com/features/actions) in this repository
consists of the following steps:

- Install the Rust toolchain that is defined in the project
  `rust-toolchain.toml`, or stable if absent.
- Verify that `Cargo.lock` is up to date (every cargo command runs with
  `--locked`).
- Code security scanning of `Cargo.lock` with
  [osv-scanner](https://github.com/google/osv-scanner) and suppression of
  certain vulnerabilities for a limited time, see
  [osv-scanner.md](osv-scanner.md). Optionally
  [Grype](https://github.com/anchore/grype).
- Linting with `rustfmt` and `clippy` (warnings are errors).
- Unit tests.
- Integration tests, i.e. the tests in the `tests` directory.
- Code coverage with
  [cargo-llvm-cov](https://github.com/taiki-e/cargo-llvm-cov).
- Optionally building a binary and attaching it to a GitHub release.

Note: there is an [internal action](../.github/workflows/package-version-updater.yml)
that will update package versions that cannot be updated by Dependabot.
