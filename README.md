# ABPB — Artificial Intelligence Bot for Counter-Strike 1.6

**An independent, identity-renamed build of [AiBo 4.3.3](https://github.com/keinzwei/AiBo) (YaPB fork), using its own `abpb` namespace so it can run next to YaPB, SYPB and other bot systems.**

ABPB keeps AiBo's Human Copycat, Human Aim, local learning, navigation, exploration and procedural dialogue, while every command, cvar, folder, DLL, Metamod name and AMXX native uses the separate `ABPB` / `abpb` identity.

- Base: AiBo `v4.3.3`, upstream commit `760180e`
- Plugin: `addons/ABPB/bin/ABPB.dll` (32-bit Metamod plugin)
- Config: `addons/ABPB/conf/ABPB.cfg` (181 `abpb_*` cvars)
- Commands: `abpb` (short alias `abp`)
- AMXX module library: `abpb`
- License: GPL-2.0-or-later

[中文说明](README-ABPB-CN.md) · [Quick install (中文)](INSTALL-ABPB-CN.txt) · [Build guide](docs/human-copycat/BUILD.md) · [Credits](CREDITS.md) · [Provenance notice](NOTICE-ABPB.md)

## Download

Download **`ABPB-4.3.3-Windows-x86.zip`** from [Releases](https://github.com/Faker1147/ABPB/releases). The archive contains the DLL, configuration, language files, a Dust2 navigation graph, optional AMXX module/include/sample and Chinese documentation.

SHA-256 of the current package:

```text
A0D74829F9B473B5A1F342A3C76786779EDDB1F5D826A19A6488EE746D239D14
```

## Install

1. Close Counter-Strike 1.6 / HLDS.
2. Merge the archive's `addons` folder into your `cstrike` folder:
   ```text
   cstrike/addons/ABPB/bin/ABPB.dll
   cstrike/addons/ABPB/conf/ABPB.cfg
   ```
3. Make sure Metamod is installed, then add this line to
   `cstrike/addons/metamod/plugins.ini`:
   ```text
   win32 addons/ABPB/bin/ABPB.dll
   ```
4. Start the server and check the console:
   ```text
   meta list
   abpb version
   abpb add
   ```

ABPB is a **Metamod plugin**, not a GameDLL. Do not put it in `liblist.gam` and do not rename it to `mp.dll`. Only 32-bit (x86) GoldSrc is supported.

To use the Simplified Chinese interface, set this in `cstrike/addons/ABPB/conf/ABPB.cfg`:

```text
abpb_language "chs"
```

Then run `exec addons/ABPB/conf/ABPB.cfg` or restart the server.

## Coexistence with YaPB / SYPB

| Item | ABPB | YaPB | SYPB |
| --- | --- | --- | --- |
| Folder | `addons/ABPB` | `addons/yapb` | `addons/sypb` |
| Command | `abpb` / `abp` | `yb` / `yapb` | `sypb` |
| Cvar prefix | `abpb_` | `yb_` | `sypb_` |
| Bot quota | `abpb_quota` | `yb_quota` | `sypb_quota` |

The files, commands, cvars and plugin names do not collide, so ABPB can be **installed** next to other bot systems. However, only **one bot system should actively manage bots at a time**. Two managers both hook the same GoldSrc engine functions, create fake clients and compete for the same edict/player slots; with both auto-filling, a 32-slot server can be driven past its limits and the server may exit with `no free edicts` or a similar fatal error.

ABPB includes a safety check:

- At map start it looks for other common bot cvars (`yb_*`, `sypb_*`, `fapb_*`, `ebot_*`, `podbot_*`, `pb_*`, `zbot_*`, `zb_*`, `realbot_*`).
- If one is found and `abpb_compat_auto` is `1` (default), ABPB sets `abpb_quota 0` and `abpb_autovacate 0` and writes a warning to the server log/console.
- Set `abpb_compat_auto 0` only if you deliberately want to manage several quotas yourself.
- The default admin setinfo key is `_abpw`, not YaPB's `_ybpw`; authenticate with `setinfo _abpw <password>`.

For a mixed test, load only one bot manager, or set every other system's quota to 0 and add bots manually from one system at a time. `abpb removebots` only removes ABPB bots.

## Commands

Run these in the server console:

| Command | Purpose |
| --- | --- |
| `abpb version` | Show the ABPB version |
| `abpb add [difficulty] [personality] [team] [model] [name]` | Add one bot |
| `abpb fill [team] [count] [difficulty] [personality]` | Fill bots automatically |
| `abpb removebots [instant] [team]` | Remove ABPB bots |
| `abpb list` | List ABPB bots |
| `abpb menu` | Open the main bot menu |
| `abpb weapons [mode]` | Set the weapon mode |
| `abpb cvars` | List all cvars |
| `abpb graphmenu` | Open the navigation graph editor |
| `abpb_human_status` | Human Copycat status |
| `abpb_human_aim_status` | Human Aim status |
| `abpb_learning_status` | Local learning status |
| `abpb_explore_status` | Geometry exploration status |
| `abpb_chat_status` | Procedural dialogue status |
| `abpb_final_status` | Combined 4.3.3 status |
| `abpb_human_save` / `abpb_human_reload` | Save / reload Human Copycat data |

## Settings

Edit `cstrike/addons/ABPB/conf/ABPB.cfg`; every option has a comment above it.

| Setting | Default | Purpose |
| --- | ---: | --- |
| `abpb_quota` | `31` | Target bot count |
| `abpb_compat_auto` | `1` | Detect other bot systems and set ABPB quota/autovacate to 0 |
| `abpb_quota_mode` | `normal` | `normal`, `fill` or `match` |
| `abpb_language` | `en` | Interface language; use `chs` or `cht` for Chinese |
| `abpb_human_copycat` | `1` | Enable Human Copycat |
| `abpb_human_aim_level` | `100` | Global Pro aim ceiling; `-1` selects adaptive mode |
| `abpb_human_aim_skill` | `100` | Base aim skill |
| `abpb_human_reaction_skill` | `100` | Base reaction skill |
| `abpb_human_pro_profiles` | `1` | 42 player-inspired profiles |
| `abpb_human_hltv_skill` | `1` | HLTV-ranking ability weighting |
| `abpb_human_ct_patrol` | `1` | Dust2 CT search patrols |
| `abpb_learning` | `1` | Local learning system |
| `abpb_chat` | `1` | Bot chat |
| `abpb_human_chat_language` | `profile` | `profile`, or force `en` / `zh` / `ru` … |

Player names and rankings are simulation references, without endorsement or measured equivalence to real players. The ranking-to-ability conversion is AiBo's own design.

## Features

| Area | Implementation |
| --- | --- |
| Navigation | Native `.GRAPH` navigation, observed human routes and geometry exploration with floor, clearance, connection and progress checks |
| Human Aim | Reaction delay, bounded turning, aim variation, recoil handling, tapping, bursts, spray control, counter-strafe, preaim and scoped AWP behavior |
| Human Copycat | Observes eligible human movement and aim, stores map-local demonstrations and reproduces bounded behavior |
| 42 profiles | AWP/rifle preferences and simulated ability weights inspired by historical HLTV rankings |
| Local learning | Tabular reinforcement learning, demonstration priors, bot-vs-bot updates, persistent experience, opponent adaptation and team formation learning |
| Exploration and dialogue | Geometry exploration plus local procedural conversations in the profile languages, without a cloud service, GPU or downloaded language model |
| Persistence | Validated map-local files for demonstrations, routes, learning, exploration, strategy memory and aim calibration |

## AMXX module (optional)

Only needed if your own AMXX plugin calls ABPB natives.

1. Copy `addons/amxmodx/modules/abpb_amxx.dll`.
2. Add `abpb` to `addons/amxmodx/configs/modules.ini`, or use `#pragma loadlib abpb`.
3. Copy `addons/amxmodx/scripting/include/abpb.inc` into your include folder.
4. Recompile against `abpb.inc`; see `abpb_sample.sma`.

All natives start with `abpb_`, for example `abpb_get_api_version`, `abpb_get_bot_count`, `abpb_add_bot` and `abpb_is_user_bot`. The module library name is `abpb`, which does not collide with `yapb` or `sypb`.

## Build

Requirements: CMake 3.22+, a C++20 compiler, Python 3, and Ninja or Make. Dependencies are vendored under `ext/`.

Native fake-engine tests:

```sh
cmake -S . -B build/test -G Ninja \
  -DCMAKE_BUILD_TYPE=Debug -DABPB_NATIVE=ON -DABPB_64BIT=ON \
  -DABPB_BUILD_TESTS=ON -DABPB_WITH_TLS=OFF \
  -DABPB_WITH_AMXX=OFF -DABPB_LTO=OFF
cmake --build build/test --parallel 4
ctest --test-dir build/test --output-on-failure --timeout 90 --parallel 2
```

Windows x86 with MSVC:

```bat
cmake -S . -B build/windows -A Win32 -DABPB_WITH_TLS=ON -DABPB_WITH_AMXX=OFF -DABPB_LTO=OFF
cmake --build build/windows --config Release --parallel 4
python tools/check_dll.py --arch i386 build/windows/Release/ABPB.dll
```

The [build guide](docs/human-copycat/BUILD.md) covers MinGW/Zig cross-builds and the optional AMXX module.

## Validation and limitations

- The shipped Windows x86 binaries were built with Zig 0.17.1 (Clang, `x86-windows-gnu`) plus CMake/Ninja and link statically against libc++/libc++abi.
- `ABPB.dll` and `abpb_amxx.dll` pass the 32-bit DLL contract check and were test-loaded with a 32-bit `LoadLibrary` harness.
- Host tests: **292/293 pass**. The only failure is `aim_baseline_compatibility`, caused by small Windows libm/FP differences from the Linux-recorded baseline (max 0.702°, not a rename or runtime defect).
- No real Counter-Strike 1.6 server was available in the build environment, so live gameplay still needs testing on your server.
- The DLL imports the UCRT (`api-ms-win-crt-*`). Windows 10/11 include it; Windows 7 needs KB2999226 or the VC++ 2015–2022 x86 runtime.
- TLS is disabled in this build (no `cacert.pem`); local graphs and learning are unaffected.

Legacy AiBo documentation is kept for feature details; its old `aibo_*` command names and installer paths are historical. The current names are always `abpb_*` and `addons/ABPB`.

## Credits and license

AiBo was created and maintained by **K9_einzwei** ([keinzwei](https://github.com/keinzwei)). AiBo builds on **YaPB, jeefo and the YaPB contributors**, with **PODBot / Count Floyd** ancestry and the **RealBot Human Copycat** extension. ABPB is an identity-renamed fork prepared by the ABPB contributors; all upstream authorship and graph/runtime credits are retained.

The combined fork is **GPL-2.0-or-later**. Read [LICENSE](LICENSE), [NOTICE-ABPB.md](NOTICE-ABPB.md), [CREDITS.md](CREDITS.md) and the notices under `ext/` and `cfg/addons/ABPB/licenses/`.

Counter-Strike and Half-Life belong to their respective rights holders. This package does not include the game.
