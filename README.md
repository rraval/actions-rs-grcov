# Forked `grcov` Action

An updated version of [`@actions-rs/grcov`](https://github.com/actions-rs/grcov) which seems
[to be abandoned](https://github.com/actions-rs/grcov/pull/90#issuecomment-726937034).

This repo is in a state of "do the bare minimum to get the features working".
That said, the following enhancements seem to be working correctly:

- https://github.com/actions-rs/grcov/pull/81: Faster installation of the
  `grcov` binary from GitHub releases instead of building from source.

- https://github.com/actions-rs/grcov/pull/90: Support for the `--excl-*`
  options for excluding specific lines by regex.

The canonical workflow:

```yaml
on: [push, pull_request]

name: Code Coverage

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions-rs/toolchain@v1
        with:
          profile: minimal
          toolchain: nightly
          override: true
      - uses: actions-rs/cargo@v1
        with:
          command: test
          args: --all-features --no-fail-fast
        env:
          CARGO_INCREMENTAL: '0'
          RUSTFLAGS: '-Zprofile -Ccodegen-units=1 -Cinline-threshold=0 -Clink-dead-code -Coverflow-checks=off -Cpanic=abort -Zpanic_abort_tests'
          RUSTDOCFLAGS: '-Zprofile -Ccodegen-units=1 -Cinline-threshold=0 -Clink-dead-code -Coverflow-checks=off -Cpanic=abort -Zpanic_abort_tests'
      - uses: rraval/actions-rs-grcov@master
        id: grcov
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - uses: coverallsapp/github-action@master
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          path-to-lcov: ${{ steps.grcov.outputs.report }}
```

Options are configurable via `.github/actions-rs/grcov.yml`, here's what @rraval prefers:

```yaml
ignore-not-existing: true
llvm: true
prefix-dir: /home/user/build/
ignore:
  - "/*"
  - "C:/*"
  - "../*"
excl-line: "unreachable!"
```
