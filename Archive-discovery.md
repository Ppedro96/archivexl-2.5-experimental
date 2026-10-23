# External archive discovery on Cyberpunk 2077 2.50

Experimental.1 could start, reload its own bundle and load an existing save, but
external archive extension configs in `archive/pc/mod` were not discovered. This
is a separate defect from the transaction-system save-loading issue.

## Cause

`ExtensionLoader::Configure` walks `ResourceDepot::groups` and reads configs only
for groups matching `ArchiveScope::Mod`. The SDK still assigned value 4 to Mod.
The native 2.50 runtime assigns value 6. ArchiveXL loads its own bundle through a
separate path, so testing that bundle alone did not exercise the failing filter.

## Native evidence and minimal correction

For executable 3.0.88.46235 / Steam build 26133166:

- RVA 0x1259F8C passes 6 to the directory-path builder at 0x1259FAC.
- That builder maps 6 to `mod`, 1 to `content`, 2 to `ep1`, and 4/5 to `patch`.
- RVA 0x125946C creates or looks up the mod archive group with scope 6 before
  loading its archives.
- Group size 0x38, basePath offset 0x10 and scope offset 0x30 remain consistent
  with native group construction and lookup.

The combined SDK patch changes only `ArchiveScope::Mod` from 4 to 6 in
ResourceDepot.hpp. This also corrects the scope ArchiveXL assigns to its own
extra archive group. Other scope values and unrelated ResourceDepot fields were
left unchanged; this is not a claim that the entire depot layout is updated.

Codeware's `ResourceDepot.ArchiveExists` uses the same enum. The local test also
rebuilt Codeware with this one-line SDK change so its archive-presence queries
agree with the game. This repository does not distribute that Codeware build.

## Observed result

The local5 build discovered `zDeceptiousQuestCore.archive.xl`; the startup log
reported its localization merge, the `deceptiousquestcore.journal` merge and
completion of all journal merges. This proves discovery and processing of that
external config, not quest completion, all archive features, or compatibility
of arbitrary mod combinations. See Validation.md for the exact runtime scope.
