# pod-redis

The `redis` candy of the OpenCharly candy library, as a standalone repo (the
candy de-submodule cutover, kind-prefixed naming). It ships a Redis-compatible
key-value server supervised on `127.0.0.1:6379`.

## What it provides

Installs the Redis server and CLI — the `redis` package on Fedora (resolved to
`valkey-compat-redis` on Fedora 43) and `valkey` (which provides
`/usr/bin/redis-server` and `/usr/bin/redis-cli`) on Arch/CachyOS. A supervised
`redis-server` binds `127.0.0.1:6379` with a `~/.redis` data dir and a 60s/1-key
RDB snapshot policy, so the running service answers `redis-cli ping` with `PONG`
and accepts TCP on its published host port.

| Property | Value |
|---|---|
| Port | `6379` |
| Service | `redis` (`/usr/bin/redis-server --bind 127.0.0.1 --port 6379 --dir ~/.redis --save 60 1`, `restart: always`, priority 20) |
| Env | `REDIS_URL=redis://127.0.0.1:6379` |
| Packages | `valkey` (arch), `redis` → `valkey-compat-redis` (fedora) |
| Consumed by | `pod-immich`, `pod-immich-ml` |

## How to use it

```yaml
my-image:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/pod-redis:<tag>'
```

```bash
charly box build my-image
charly config my-image
charly start my-image
redis-cli -h 127.0.0.1 -p 6379 ping   # PONG
```

The `charly config` step injects `REDIS_URL` for service discovery: same-container
consumers receive `redis://localhost:6379`, cross-container consumers receive
`redis://charly-redis:6379`.

## Layout

- `charly.yml` — the `redis:` candy entity (description, `require`, `distro`,
  `env`, `port`, `service`, `plan`) plus its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-infrastructure:redis` — the candy properties, the
  env contract, the distro package divergence, and verification.
- `/charly-infrastructure:valkey` — the Remi-repo Valkey 9 candy (a separate,
  differently-versioned package).
- `/charly-infrastructure:postgresql` — often paired in service stacks.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
