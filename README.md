# Doodle Workspace

Container repo with git submodules for all Doodle project repos.

Doodle is a small, dynamically typed, kid-first teaching language — a
modern Logo successor designed to be a real language kids need not outgrow.

## Submodules

| Repo | Description |
|------|-------------|
| [discussions](https://github.com/doodle-lang/discussions) | Design history, specs (language, engine), and plans |
| [doodle-rust](https://github.com/doodle-lang/doodle-rust) | Engine, bindings, and CLI host, implemented in Rust |

## Setup

```sh
git clone --recurse-submodules https://github.com/doodle-lang/workspace.git
```

Or if already cloned:

```sh
git submodule update --init --recursive
```

## License

[MIT](LICENSE)
