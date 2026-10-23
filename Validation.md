# Validation scope — 23 October 2026

Target: Cyberpunk 2077 2.50, executable 3.0.88.46235, Steam build 26133166.
The runtime was RED4ext DLL mode with RED4ext 1.30.0, redscript 0.5.32, CET 1.38.0,
and locally ported Codeware 1.20.6 / TweakXL 1.11.5 based on the PRs cited in README.

## Earlier local4 checks retained by this patch

- MSVC release compilation and linking of the final port7/local4 build.
- Game startup and main menu.
- Version APIs, ArchiveXL.Require, and two completed extension reloads.
- Resource existence and asynchronous loading of a built-in ArchiveXL mesh and alias.
- Existing save loading: confirmed by the player after the transaction vtable correction.
- Normal game shutdown: plugin unload and RED4ext shutdown completed in that session.
- Patch syntax/whitespace checks, reverse applicability, and forward applicability to
  clean baseline exports. The source patches reproduce the reviewed working files.

Earlier local4 DLL SHA256:
`0B3B2EDCDB050AD9B8F7902F1F241C6712E707BB107D07D867540976D239E1D2`

## Experimental.2 / local5 checks

The local5 build changes only the SDK's ArchiveScope::Mod value from 4 to 6.

- MSVC release compilation and linking succeeded.
- Named external discovery of zDeceptiousQuestCore.archive.xl succeeded.
- Its dqc_onscreens localization entries merged, followed by completion of all translations.
- Its deceptiousquestcore.journal entries merged, followed by completion of all journal merges.
- ModSettings.archive.xl was also discovered and its localization processed.
- Main menu was visibly reached with local5 in the subsequent combined framework test.
- A read-only Codeware ArchiveExists query returned true for Deceptious Quest Core.
- Normal shutdown of that combined test completed: all six RED4ext plugins unloaded,
  followed by the loader's shutdown-complete message. No new save-load or gameplay
  claim is made for this combined configuration.

Local5 DLL SHA256:
`54517870E18A947FC01214A44BC7040E1976C77A0E4D2EC77824F98FD6A0BD98`

The combined runtime also used Codeware rebuilt with the same scope correction,
Deceptious Quest Core 3.3.1, locally ported ModSettings 0.2.21 and Audioware 1.9.9,
Input Loader 0.2.3, Native Settings 1.97, locally adapted CyberwareEX 1.5.6 with
Override Mode, Enhanced Capacity and Extra Weapon Slots. These companion mods/builds
are not included in this source release. Their inclusion in a menu test is not a
certification of their gameplay functions.

The local binary is not included. A fresh-clone build of this published bundle has
not been performed, and a new build may have a different hash because of timestamps,
paths and compiler environment.

## Not validated / known limits

- WorldWidgetLimitPatch is disabled; this build does not remove that component limit.
- Sector mutations and cooked meshes still need representative runtime tests.
- Custom quests, journal merging, mappins, garments, appearances, and complete mod
  combinations have not received comprehensive gameplay testing.
- ASI reload and other game versions are not supported by this port.
- Loading one save does not establish long-session stability or safety for every save.

Only source patches, technical notes, licenses and checksums are distributed.
The local minidump, disassembly dumps, screenshots, private paths and user logs are
excluded. The SDK patch retains upstream/community credit and adds the documented
transaction-system virtual-slot correction plus the archive-scope correction.
