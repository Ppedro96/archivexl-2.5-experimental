# ArchiveXL 2.5 — experimental source patch

Experimental source patch for ArchiveXL 1.27.5 on Cyberpunk 2077 2.5,
exe 3.0.88.46235 / Steam build 26133166. **Unofficial, experimental, for testing.**

This is a **source patch release**, with build instructions; it does not contain a
ready-to-install DLL. It is intended for developers and experienced mod testers.
ArchiveXL remains the work of [psiberx](https://github.com/psiberx/cp2077-archive-xl).

**Experimental.2 fixes external archive discovery:** the combined SDK patch now
uses the native 2.50 `ArchiveScope::Mod` value of 6. Experimental.1 used 4 and
silently skipped external `.xl` configs. Discovery and localization/journal
processing of Deceptious Quest Core passed locally. See [Archive-discovery.md](Archive-discovery.md)
for the diagnosis and the limits of that test.

The earlier local4 build passed startup, API/resource checks, repeated reload,
one existing save load confirmed by the player, and normal shutdown. Local5 passed
external discovery, localization/journal processing, main-menu startup and normal
shutdown; no new save-load check is claimed for its combined framework setup.
These checks do not certify full gameplay or dependent-mod compatibility. See [Validation.md](Validation.md).
Back up your saves and current mod files before testing a build.

## Contents and credits

- archivexl-2.5-local.patch: local ArchiveXL port, against
  d56db6e0e63e9562be92a17e58ecbbf3d824cd56.
- red4ext-sdk-2.5-combined.patch: SDK changes against
  f9ad5334362a8a0815baf58f47615697485feb52. Combines official update
  0f5eba42a4cf3a1161ad3de0b4ed6f1901ec169e and PR 228 layouts at
  b139fcbd51b1c391399662f9fbfc00c5977f7231. These are upstream/community
  contributions, not original work of this port. It also includes the local
  ITransactionSystem virtual-slot corrections documented in Save-loading-hang.md
  and the ArchiveScope::Mod correction documented in Archive-discovery.md.
- Technical-notes.md: offsets, rationale, test iterations and limitations.
- Save-loading-hang.md: local diagnosis and native evidence for the transaction SDK fix.
- Archive-discovery.md: scope correction required to discover external archive configs.
- Original MIT licenses retained for psiberx/ArchiveXL and RED4ext.SDK.

No game executable, proprietary assets, saves, user logs or credentials are included.
This independent test publication has not been accepted or endorsed by upstream.

## Reproduce on Windows x64

Use Git, Microsoft C++ Build Tools 2022 (tested MSVC 14.44.35207), Windows SDK
10.0.26100.0, and xmake 3.1.2. In a fresh checkout:

```powershell
git clone https://github.com/psiberx/cp2077-archive-xl.git
git -C cp2077-archive-xl checkout d56db6e0e63e9562be92a17e58ecbbf3d824cd56
git -C cp2077-archive-xl submodule update --init --recursive
# Replace C:/port-bundle with this folder's location.
git -C cp2077-archive-xl apply --check C:/port-bundle/archivexl-2.5-local.patch
git -C cp2077-archive-xl apply C:/port-bundle/archivexl-2.5-local.patch
git -C cp2077-archive-xl/vendor/RED4ext.SDK apply --check C:/port-bundle/red4ext-sdk-2.5-combined.patch
git -C cp2077-archive-xl/vendor/RED4ext.SDK apply C:/port-bundle/red4ext-sdk-2.5-combined.patch
Set-Location cp2077-archive-xl
xmake f -p windows -a x64 -m release --toolchain=msvc --vs=2022 --vs_sdkver=10.0.26100.0 -y
xmake -y -j 2
```

The package patch pins mimalloc 2.2.4 because TiltedCore uses mi_malloc_size.
Build output: build/windows/x64/release/ArchiveXL.dll and ArchiveXL.pdb.
Install using the original 1.27.5 bundle/scripts, replacing DLL and PDB with matching build outputs.
Test only with the targeted 2.50 runtime and compatible RED4ext/SDK layouts.
The local runtime used RED4ext 1.30.0, redscript 0.5.32, CET 1.38.0, plus locally
ported Codeware 1.20.6 PR63 and TweakXL 1.11.5 PR22; stock incompatible builds are not equivalent.
The experimental.2 test also applied `ArchiveScope::Mod = 6` to Codeware's SDK
because its ArchiveExists API uses this enum. That Codeware build is not included here.

Both patches were checked for reverse application against the working source trees
and forward application against clean exports of their pinned baseline commits.
A fresh-clone build from this bundle has not been performed yet; the build above succeeded
in the prepared working tree. Generated binaries may differ because of build timestamps and paths.

## Review focus / outstanding

- WorldWidgetLimitPatch intentionally disabled: old address and pattern absent.
  Do not claim the component limit is removed.
- Sector/cooked transform operations need representative gameplay tests.
- Test actual quest injection, garments, custom appearances, journal merging and mappins.
- The reload callback is for RED4ext DLL mode; ASI mode has not been ported.
- Menu/resource smoke tests do not certify all dependent mods.

## Evidence behind reload fix

RED4ext 1.30's StateSystem::Run erases callbacks returning true. Return false to
retain a Running callback, even though the SDK GameState comment says the return
value is ignored for Running:
https://github.com/WopsS/RED4ext/blob/v1.30.0/src/dll/Systems/StateSystem.cpp

Upstream SDK layouts: https://github.com/WopsS/RED4ext.SDK/pull/228
Related ABI updates: https://github.com/psiberx/cp2077-codeware/pull/63
and https://github.com/psiberx/cp2077-tweak-xl/pull/22

## Feedback

Report reproducible results in this repository, including game build, framework
versions, enabled mods, and the smallest reproduction. Redact usernames and local
paths from logs. Do not attach saves, crash dumps, credentials, or game binaries.

The release is a starting point for community review, not a replacement for an
upstream-supported update. WorldWidgetLimitPatch is disabled and ASI mode is unsupported.
