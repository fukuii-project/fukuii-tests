<div align="center">
  <img src="https://raw.githubusercontent.com/fukuii-project/fukuii-brand/HEAD/logo/fukuii-hex-logo.png" alt="fukuii-tests" width="280"/>
</div>

# fukuii-tests

Test fixtures and conformance vectors for Fukuii — Ethereum Classic and multi-network, organized by
network and purpose.

Part of the [Fukuii project](https://github.com/fukuii-project).

## Contents

| Path | What |
|---|---|
| `archive/` | preserved copies of upstream material that is being deprecated or deleted — **frozen** |
| `upstream/` | live upstream repositories, pinned as submodules and not fetched by default |
| `proposals/` | tests for a single EIP or ECIP, independent of any network |
| `networks/` | tests scoped to a network or one of its upgrades |

`archive/` and `upstream/` are inherited; `proposals/` and `networks/` are this project's own
work. Nothing under `archive/` is edited — see [AGENTS.md](AGENTS.md) for why, and for how the
freeze is checked.

## License

Apache-2.0 for Fukuii's own files. Vendored upstream corpora retain their original licenses. See
[`LICENSE`](LICENSE) and [`NOTICE`](NOTICE).

© 2025–present The Fukuii Authors · Chippr Robotics LLC · White B0x Inc.
