# ArchiveXL 2.5 experimental port

Target: Cyberpunk 2077 2.5, exe 3.0.88.46235, Steam build 26133166.
ArchiveXL source: d56db6e0e63e9562be92a17e58ecbbf3d824cd56 (1.27.5).
Branch: codex/archivexl-2.5-port. Experimental community source patch; not an official release.
SDK baseline: f9ad5334362a8a0815baf58f47615697485feb52.
SDK changes: official 2.5 update 0f5eba42a4cf3a1161ad3de0b4ed6f1901ec169e and PR 228
layouts at b139fcbd51b1c391399662f9fbfc00c5977f7231.

## Implemented

- Replaced missing addresses for widget spawning, resource dependencies, TweakDB loading,
  appearance suffixes, mesh materials, archive loading, customization, factories and journals.
- ABI fixes: InkSpawningContext request +0x10; local widget spawning adds bool;
  TweakDB load adds CString reference; suffix fourth argument is TweakDBID reference;
  depot loader returns bool and has memory mode before error level (native calls use 0,2);
  ResourceRequest uses the SDK 0x30 layout.
- Customization lock/state +0x98/+0xA0; resource handles +0x48/+0x58. State groups
  +0x70/+0x80/+0x90 confirmed against native accessors.
- Sector setup: size 0x90, 19-bit node index at +0x70, distances +0x78/+0x7C,
  cooked entry index +0x80. Node arrays +0x30/+0x40. Added index/count checks before writes.
- Cooked instanced meshes: entries +0x70, stride 0x40, mesh +0x20, appearance +0x28;
  transforms +0x60, stride 0x30; ranges +0x80, uint16 first/count; version +0x90.
  Nonempty transforms array takes precedence and is used WHOLE. Otherwise select the
  cooked buffer range for setup.cookedEntryIndex. Reject unsupported versions and bad ranges.
- Destructible mesh transforms now use DynArray.
- Mappin/POI hooks now target common lookup bodies, not misleading old wrapper hashes.
- Reload queued by ArchiveXL.Reload and processed by RED4ext Running callback. Return false
  to retain callback: RED4ext 1.30 StateSystem::Run removes callbacks returning true even
  in Running, contrary to the SDK comment. Two separate reloads are included in smoke tests.
- Runtime restricted to 2.50. This port targets RED4ext DLL loading; ASI reload is not supported.
- Build pins tiltedcore.mimalloc v2.2.4 to retain mi_malloc_size.

## Native evidence for the substantial layout changes

RVA values below apply ONLY to the tested executable. Adjacent archive-*.txt files contain
local disassembly evidence; these are not bundled for public distribution.

- Setup initializer 0x409514 copies setup+0x80 to nodeInstance+0x88.
- GetTransforms 0x20BF910 (hash 271197749) establishes whole-array vs cooked-range behavior.
- Constructor 0x4EC040 and serializer 0x6CD098 establish cooked node fields and strides.
- Entry accessors: mesh 0x20BEF20, appearance 0x20BEEE0.
- Native 0x4AF45C -> 0x4AEFB4 -> 0x20A038 passes raw world transforms to rendering;
  0x930D3C consumes fixed position/quaternion/scale. No setup transform is composed there.
- Mappin 0x22119F0 / hash 2918455690 reads resource +0x58; caller 0x2D434F8 uses position +4/+8/+C.
- POI 0x22118A0 / hash 3603701972 reads +0x68; wrapper 0x9B615C uses entityID +8, position +0x10.
  Old mappin hash 3299551353 actually resolves to that POI wrapper. Address presence alone is insufficient.
- Public format corroboration: nobody71004/fivem2open77 commit
  225cc27b6ef0799535533b3d200f9b793285a151, fivem2open77/world/sector25.py.

## Explicit limitations

WorldWidgetLimitPatch is disabled with a startup warning. Its old address is absent and
its old byte pattern 83 FB 0A 73 16 does not occur in this executable. No replacement branch
was patched speculatively. Mods relying on removing this world-widget component limit remain unverified.

Sector mutations, custom quest content, clothing/appearance combinations and gameplay have
NOT been exercised by menu tests. Cooked mesh implementation is based on native evidence and
compile-time layout assertions but still needs representative runtime sector tests.
A successful startup does not certify the full mod collection.

## Test iterations

1. local1: resolver terminated startup on old widget-limit address 1693194103. Removed that
   query entirely; missing-address resolution is fatal, not a recoverable null return.
2. local2: menu and normal shutdown passed. Require returned true. Reload request accepted but
   not executed because the callback returned true. Smoke version lookup failed because CET
   does not expose _G; corrected the test to direct calls.
3. local3: two persistent reloads completed, but loading the existing save stalled in
   attachment suffix resolution. The engine watchdog timed out after 120 seconds.
   A local minidump identified a stale transaction-system virtual slot in the SDK.
4. local4 (build port7): individually corrected SDK slots, including FindSlotData
   0x450 and MatchVisualTag 0x4E0. Version APIs, Require, two actual reloads, resource
   existence and asynchronous loads of the original mesh and alias passed. The player
   confirmed that the game and save opened. Normal shutdown was then confirmed:
   RED4ext unloaded ArchiveXL and the other two plugins and completed shutdown
   without a watchdog timeout in that session. Full gameplay is not certified.
   See Save-loading-hang.md for the native evidence and nearby unchanged slots.

Build logs archivexl-build-port1 through port4 preserve successful compilation. The final
address audit has 129 constants and zero missing (archive-port-address-audit.json).

Installed DLL local4 SHA256:
0B3B2EDCDB050AD9B8F7902F1F241C6712E707BB107D07D867540976D239E1D2

## Reproduction and review

community-port contains two patches: ArchiveXL changes and the combined SDK update against
its pinned baseline, plus licenses and build instructions. Both patches passed reverse
application checks against the working trees and forward application against clean
exports of the pinned baseline commits. This publication is independent of upstream.

Upstream sources:
- https://github.com/psiberx/cp2077-archive-xl
- https://github.com/WopsS/RED4ext.SDK/pull/228
- https://github.com/psiberx/cp2077-codeware/pull/63
- https://github.com/psiberx/cp2077-tweak-xl/pull/22
- https://github.com/WopsS/RED4ext/blob/v1.30.0/src/dll/Systems/StateSystem.cpp
