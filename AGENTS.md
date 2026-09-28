# AGENTS.md — layer-container-nesting

Standalone candy repo for the `container-nesting` layer — rootless nested
podman/buildah/skopeo inside a rootless outer container, with zero added
capabilities. The candy lives in `charly.yml` at the repo root: the per-distro
package arms, the `security:` block (`unmask=/proc/*`, devices), the two env
contracts, the subuid/config `plan:` steps, the `check:` assertions, and the
embedded `skill:` entity projected into the marketplace corpus as
`/charly-distros:container-nesting`.

Canonical files:

- `charly.yml` — the `container-nesting:` candy entity and the
  `container-nesting-skill:` skill entity.
- `CHANGELOG/` — per-CalVer release history; read it before changing baked
  checks.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-distros:container-nesting` — the owning skill. The authoritative
  `mount_too_revealing()` kernel RCA, the `containers.conf`/`storage.conf`/
  `policy.json` dual-location contract, the two env contracts, the subuid
  layout, and the security posture. Load before editing or troubleshooting the
  layer.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, per-distro `distro:` arms, package/repo
  sections, and service declarations). Load before editing any entity field or
  plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The candy's `plan:` `check:` steps are the functional evidence; they must stay
  valid on every distro arm they run on. Scope a distro-specific check in the
  command itself — the check runner does not honour runner-level
  `exclude-distro` fields.

## Modify this repo

- Edit the `container-nesting:` candy entity AND the `container-nesting-skill:`
  skill entity in `charly.yml` together. The skill is the projected usage source,
  so a package, distro-arm, or behaviour change not mirrored in the skill leaves
  the corpus stale.
- Keep the security posture at the surgical minimum: `unmask=/proc/*` plus the
  two devices, **no** `cap_add`. The kernel RCA in the owning skill explains why
  capability hammers are not the fix.
- Keep every config written to **both** the system-wide and user locations, and
  keep the two env contracts (`_CONTAINERS_USERNS_CONFIGURED=""`,
  `BUILDAH_ISOLATION=chroot`) — each is load-bearing.
- Keep the subuid/subgid ranges inside the outer keep-id window
  (`user:1:999` + `user:1001:64535`, `root:1:65535`).
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
