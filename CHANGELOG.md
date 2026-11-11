# Changelog

## 4.3.3-abpb.1

- Compatibility guard: detect YaPB/SYPB/FAPB/E-Bot/PODBot/ZBot/RealBot cvars at map start and set `abpb_quota`/`abpb_autovacate` to 0, preventing two bot managers from filling the same slots.
- Do not intercept client commands or console prints belonging to other bot systems' fake clients.
- Documentation now states that side-by-side installation is supported, but only one bot system should actively manage bots.

## 4.3.3

- CT searches through Dust2 Tunnel, T-Spawn and Pit, with objective/contact priorities and rotating participation.
- Forty-two player-inspired weapon roles and documented historical ranking-based weights.
- Human Aim defaults at level 100; adaptive mode at level -1.
- Hybrid GRAPH/human-route/geometry navigation, six local learning mechanisms, persistent memory and dialogue in 16 languages.
- Windows installer with folder checks, plugin configuration merging, numbered backups and rollback handling.
- Custom graphs preserved; supplied `autobuy.txt` and `listenserver.cfg` retained.
- Active-release source, vendored dependencies, installer code, build recipes and tests.

The original package records 293 passing code tests and 11 passing installer scenarios on Linux. See [validation](docs/VALIDATION.md) for scope and limits. Earlier release history is not reconstructed here.
