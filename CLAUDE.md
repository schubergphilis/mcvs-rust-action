# CLAUDE.md

**mcvs-rust-action** is a GitHub composite action ([action.yml](action.yml))
and a remote Taskfile ([build/task.yml](build/task.yml)) that standardize
quality checks for Rust projects. It mirrors
[mcvs-golang-action](https://github.com/schubergphilis/mcvs-golang-action).

- `action.yml` installs the toolchain (from `rust-toolchain.toml`) and Task,
  then runs `task remote:<task>` based on `testing-type`: `unit`,
  `integration`, `coverage`, `lint`, `security-cargo`, `security-grype`.
  Setting `release-application-name` builds and uploads a release binary.
- `build/task.yml` pins tool versions (`CARGO_LLVM_COV_VERSION`,
  `OSV_SCANNER_VERSION`); `scripts/package-version-updater.sh` bumps them
  weekly. Task keys must be sorted and the file must not contain empty lines
  outside `- |` blocks (enforced by workflows).
- Inputs in `action.yml` must be sorted alphabetically.
- Every cargo command runs with `--locked`, which also verifies `Cargo.lock`.
- Pin every referenced action by full commit SHA with the version as comment.

Run the updater tests with `bats test/` (requires mikefarah yq v4 and jq).
