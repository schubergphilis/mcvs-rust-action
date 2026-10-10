# Taskfile

This repository offers a [`build/task.yml`](../build/task.yml) with standard
[Task](https://taskfile.dev/) tasks, so that the same checks can be run
locally as in the pipeline.

## Usage

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

## Configuration

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
