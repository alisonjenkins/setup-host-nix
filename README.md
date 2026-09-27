# setup-host-nix

A composite GitHub Action that configures a **NixOS host-nix self-hosted runner**
to build via the host's `nix-daemon` instead of installing Nix in the job.

Use it on runners whose pods mount the host's `/run/current-system/sw`, `/nix`
and the daemon socket (read-only store + host daemon) — e.g. the
`home-nix-builder-amd64` ARC scale-sets. On such a runner
`DeterminateSystems/nix-installer-action` cannot work (the store is a read-only
host mount), so this action symlinks the host `nix` binaries onto `PATH` and
writes a tuned `nix.conf` instead.

## Usage

```yaml
jobs:
  build:
    runs-on: home-nix-builder-amd64
    # Never run fork-PR code on a self-hosted runner.
    if: ${{ github.event.pull_request.head.repo.fork != true }}
    steps:
      - uses: actions/checkout@v4
      - uses: alisonjenkins/setup-host-nix@v1.0.1
      - run: nix flake check --no-build
      - run: nix develop --command cargo test --workspace
```

## Inputs

| Input | Default | Notes |
| --- | --- | --- |
| `max-jobs` | `1` | Concurrent derivations. `max-jobs * cores` bounds compiler procs. |
| `cores` | `4` | Cores per job. |
| `extra-substituters` | nixcache.org + nix-community | Space-separated. |
| `extra-trusted-public-keys` | matching keys | Space-separated. |

## Requirements

Only works on a self-hosted runner with the NixOS host mounts. It errors clearly
if `/usr/local/nixos-sw/bin/nix` is absent. Not for GitHub-hosted runners — use
`DeterminateSystems/nix-installer-action` there.
