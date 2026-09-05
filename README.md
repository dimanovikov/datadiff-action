# datadiff-action

GitHub Action for [datadiff](https://github.com/dimanovikov/datadiff) —
semantic diff for structured data files (JSON, YAML, CSV, TOML, XML) as a
CI gate.

Unlike a text diff, datadiff compares the *data tree*: key order and
formatting are ignored, arrays of objects are matched by a key field, and
changes are reported as data paths (`spec.replicas: 3 → 5`).

## Usage

Fail the workflow when a config file changed relative to the base branch:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0
- name: Diff config against base branch
  uses: dimanovikov/datadiff-action@v1
  with:
    old: ${{ github.workspace }}/config.yaml        # from base, see below
    new: config.yaml
```

A more complete example: check out the base copy, then gate only on risky
changes with `fail-on`:

```yaml
steps:
  - uses: actions/checkout@v4
    with:
      fetch-depth: 0
      path: pr
  - uses: actions/checkout@v4
    with:
      ref: ${{ github.base_ref }}
      path: base
  - uses: dimanovikov/datadiff-action@v1
    with:
      old: base/deploy/app.yaml
      new: pr/deploy/app.yaml
      key: name
      fail-on: spec.replicas,*.image
```

Exit-code behavior (same contract as the CLI):

- step succeeds — no changes, or changes matched no `fail-on` pattern;
- step fails — changes found (with `fail-on`: a change path matched a pattern);
- step fails with an error — file/format problem (exit code 2).

## Inputs

| Input | Description | Default |
|---|---|---|
| `old` | Path to the old file | (required) |
| `new` | Path to the new file | (required) |
| `key` | Key field to match objects inside arrays (e.g. `id`) | — |
| `fail-on` | Comma-separated path patterns; fail only on matching changes | — |
| `format` | Force the input format (`json\|yaml\|csv\|toml\|xml`) | autodetect |
| `version` | datadiff version to install (`v0.2.0`, …) or `latest` | `latest` |

Supported runners: `ubuntu-latest` (x64/ARM), `macos-latest` (Apple
Silicon), `windows-latest`.

## License

Dual-licensed under either of MIT or Apache-2.0 at your option.
