# container-nesting

Rootless nested podman / buildah / skopeo layer for OpenCharly images.

The `container-nesting` candy adds everything needed to run **rootless**
podman/buildah/skopeo **inside a rootless outer container** — at the default
uid 1000, with **zero added capabilities**, no `--privileged`, no
`seccomp=unconfined`, and no `label=disable`. It is a direct port of
`quay.io/podman/stable`'s canonical rootless-in-rootless configuration into the
charly candy system, so any box can compose it.

The load-bearing mechanism is `security_opt: [unmask=/proc/*]`: it tells podman
not to emit the `maskedPaths` entries for `/proc` on the outer container, so the
kernel's `mount_too_revealing()` check has nothing to reject when the inner
container mounts a fresh procfs. Capability hammers (`cap_add: ALL`,
`--privileged`) are unnecessary; this is the least-privilege fix. The candy also
writes `containers.conf`/`storage.conf`/`policy.json` to **both** the
system-wide and user locations, sets two hidden env contracts, and lays out
subuid/subgid ranges that fit inside the outer keep-id window.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `container-nesting` |
| Capabilities | **none** (`cap_add` empty) |
| Security opt | `unmask=/proc/*` |
| Devices | `/dev/fuse`, `/dev/net/tun` |
| Binaries | `podman`, `buildah`, `skopeo`, `fuse-overlayfs`, `newuidmap`/`newgidmap` (file caps) |
| Environment | `CHARLY_BUILD_ENGINE=podman`, `CHARLY_RUN_ENGINE=podman`, `_CONTAINERS_USERNS_CONFIGURED=""`, `BUILDAH_ISOLATION=chroot` |
| Volume | `storage` → `/var/lib/containers/storage` (root images only) |
| Service / port | none |

Per-distro packages:

- `fedora` — `buildah`, `fuse-overlayfs`, `shadow-utils`, `skopeo`, `tailscale`,
  `libsecret`.
- `arch` — `buildah`, `crun`, `fuse-overlayfs`, `libsecret`, `podman`, `shadow`,
  `skopeo`, `tailscale` (`podman` and `crun` are declared explicitly — Arch has
  no transitive pull).
- `debian` / `ubuntu` — `podman`, `buildah`, `skopeo`, `fuse-overlayfs`, `crun`,
  `uidmap`, `passwd`, `libsecret-1-0`, plus `libcap2-bin` (the `setcap`
  provider).

## How to use it

Compose the layer by pinning this repo in a box's `candy:` list:

```yaml
openclaw-desktop:
  base: cachyos.cachyos
  candy:
    - selkies-desktop
    - openclaw-full
    - container-nesting      # donates unmask + devices + config + env
```

Rootless boxes add nothing at the box level. Root boxes that want the historical
full-hammer posture (`charly-fedora`, `charly-arch`, `githubrunner`) assert
`cap_add: [ALL]` + `security_opt: [label=disable, seccomp=unconfined]` at the box
level; the box `security:` block only **unions** onto the candy's set, never
strips it.

Verify on a live deployment:

```bash
podman run --rm quay.io/libpod/alpine:latest true
# → exit 0, no "mount proc to proc: Operation not permitted"
```

## Layout

- `charly.yml` — the `container-nesting:` candy entity: the package arms, the
  security block, the env contracts, the subuid/config `plan:` steps, the
  `check:` assertions, and the embedded `skill:` entity.
- `CHANGELOG/` — per-CalVer release notes.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-distros:container-nesting` — the authoritative
  `mount_too_revealing()` RCA, the config/env contracts, and the security posture
- Pairs with: `/charly-tools:charly`, `/charly-infrastructure:virtualization`,
  `/charly-coder:sshd`
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
