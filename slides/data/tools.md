## Tooling

---

> Rust can work with pretty much all C/C++ agnostic tools

`kcov`, `gdb`, `lldb`, `perf`...

[Rust developer tools - status and strategy](https://gist.github.com/nrc/a3bbf6dd1b14ce57f18c)

---

## [Travis CI](https://docs.travis-ci.com/user/languages/rust/)

```yaml
language: rust
rust:
  - stable
  - beta
  - nightly
matrix:
  allow_failures:
    - rust: nightly
```