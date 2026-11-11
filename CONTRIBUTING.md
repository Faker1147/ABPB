# Contributing to ABPB

Please open ABPB-specific issues and pull requests in this repository. Upstream AiBo issues belong in [keinzwei/AiBo](https://github.com/keinzwei/AiBo); YaPB issues belong in [yapb/yapb](https://github.com/yapb/yapb).

## Report a problem

Include the ABPB version, operating system, engine/GameDLL, Metamod and AMX Mod X versions, map, plugins, relevant configuration and reproduction steps. A short video and console output help with navigation/aim problems. Remove passwords and personal information from logs before posting. For custom graphs, identify their source and license.

Useful console output:

```text
meta list
abpb version
abpb_human_status
abpb_final_status
```

## Make a change

Keep patches focused and preserve affected components' attribution/license notices. Describe the triggering case, changed behavior and validation. Keep generated binaries, caches and personal learning files out of source commits.

Follow the [build guide](docs/human-copycat/BUILD.md) and run relevant native tests. DLL boundary changes also need a Windows x86 build and `tools/check_dll.py`. Distinguish fake-engine tests from actual Windows/game testing.

Persistent data formats and native graph compatibility need compatibility handling and meaningful regression coverage when changed. Contributions should remain compatible with the applicable component/fork licenses. There is no promise about response times, release dates or acceptance.
