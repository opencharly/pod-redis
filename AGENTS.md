# AGENTS.md — pod-redis

Standalone candy repo for the `redis` candy — a Redis-compatible key-value server
(`redis-server` + `redis-cli`) supervised on `127.0.0.1:6379`. The entire candy
lives in `charly.yml` at the repo root. There is no source tree.

Canonical files:

- `charly.yml` — the `redis:` candy entity (description, `require`, `distro`,
  `env`, `port`, `service`, `plan`) and its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-infrastructure:redis` — the owning skill: the candy properties, the
  env contract, the distro package divergence (`redis` on Fedora resolves to
  `valkey-compat-redis`; `valkey` on Arch), and verification. Load before editing,
  building, deploying, or troubleshooting this candy.
- `/charly-infrastructure:valkey` — the separate Remi-repo Valkey 9 candy.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, service declarations).
- `/charly-check:check` — the check/R10 framework; this candy is the
  gold-standard `check:` pattern referenced there.
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The candy's own `check:` steps assert the `redis-server` / `redis-cli`
  binaries, the providing package (`valkey-compat-redis`, mapped to `valkey` on
  Arch), a live `PONG`, the reachable port, and the running service.

## Modify this repo

- Edit the `redis:` candy entity in `charly.yml`; the `skill:` entity in the same
  file is the owning skill's source — a candy change and its skill change land
  together.
- A `package:` check must query the real installed name: Fedora 43 installs
  `valkey-compat-redis` (dnf resolves `redis` via `Provides:`, but `rpm -q redis`
  returns "not installed"). Keep the `package_map` in step.
- The `~/.redis` data dir and the `--save 60 1` snapshot policy are the service's
  product; keep them in step with the checks.
- The `skill:` entity is the source for `/charly-infrastructure:redis`; never edit
  the generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
