# ArchiveXL 2.5: save-loading hang investigation

The local3 test reached the menu and processed two extension reloads. Loading the
existing save then stalled. RED4ext recorded the engine watchdog's 120-second
timeout. A small thread/stack minidump was captured before shutdown, locally only.

Main thread 42856 was in the attachment-data shared-lock acquisition at game RVA
0x22CA51E. Matching local3 symbols identify ArchiveXL's call chain as
GarmentExtension::OnResolveSuffixes -> AttachmentExtension::OnGetSuffixValue ->
IsVisualTagActive -> ITransactionSystem::FindSlotData.

The compiled SDK dispatched FindSlotData through virtual offset 0x448. In this
executable that is the lock-taking wrapper at 0x75A4E0. Native suffix resolution
dispatches through 0x450 (0x138AD28), which directly invokes the predicate iterator
0x75A554 without acquiring the attachment-data lock again. This explains the
reentrant lock wait while resolving clothing suffixes during save loading.

## Verified 2.5 transaction virtual slots

Table RVA 0x30DC058. The interface needs individual corrections, not a uniform shift.

| Method | Old SDK | 2.5 | Native RVA |
|---|---:|---:|---:|
| GetItemAppearance | 1D0 | 1D0 | see slot-ABI disassembly |
| ResetItemAppearance | 1D8 | 1D8 | see slot-ABI disassembly |
| IsSlotEmpty | 3C0 | 3C8 | 7AE4BC |
| IsSlotSpawning | 3C8 | 3D0 | 1082F48 |
| GetItemInSlot | 428 | 428 | 1BDE98 |
| GetSlotDataList | 438 | 438 | CF4970 |
| FindSlotData | 448 | 450 | 138AD28 |
| RegisterSlotListener | 490 | 498 | 300DA0 |
| UnregisterSlotListener | 498 | 4A0 | 300D64 |
| MatchVisualTagByItemID | 4D0 | 4D8 | 758A48 |
| MatchVisualTag | 4D8 | 4E0 | 2A9F444 |

Checks against native bodies distinguish similarly shaped functions:
- 430 searches by ItemID, not slot ID; GetItemInSlot must remain at 428.
- 440 iterates a callback, not an output DynArray; GetSlotDataList remains at 438.
- 490 removes a listener from a different array; slot listeners use 498/4A0.
- Slot-ID helper 1BE0F0 compares EDX and the upper byte of RDX directly. The
  TweakDBID parameter is still by value here. Do not change all TweakDBID parameters
  to references because one other suffix API changed.

Build port5 failed to link because a research PE mapping held the output DLL open;
closing that mapping resolved the build issue. Port6 compiled an intermediate
uniform-shift hypothesis, but was not installed after native checks disproved it.
Port7 uses individually verified slots. Runtime results are recorded separately.

Local evidence: archive-transaction-*.txt, archivexl-local3-loading-hang.dmp,
archivexl-local3-hang-threads.json and archivexl-local3-RED4ext.log. The dump and
user logs must not be included in a public patch bundle.
