# osv-scanner

The `mcvs-rust-action` uses [osv-scanner](https://github.com/google/osv-scanner)
to scan `Cargo.lock` for vulnerable crates, using the OSV database, which
includes the [RustSec](https://rustsec.org/) advisories.

## Ignoring Vulnerabilities

Add an `osv-scanner.toml` file to the project root to ignore vulnerabilities
that cannot be fixed right away, see
[osv-scanner.toml.example](../osv-scanner.toml.example):

```toml
[[IgnoredVulns]]
id = "RUSTSEC-2024-0001"
ignoreUntil = 2026-11-06
reason = "Waiting for upstream fix: https://github.com/example/crate/issues/1"
```

- Always set `ignoreUntil`, at most one month ahead, so that the pipeline fails
  again if the issue has not been resolved.
- Each ignored vulnerability should have a clear `reason`.
