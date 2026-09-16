# Ruleset structure

How [`RulesetSrc/`](../../RulesetSrc/) is laid out, how a year-specific ruleset is assembled from a manifest, and what compile copies versus what stays source.

See also [BEMBase and support files](02-bembase-and-support-files.md) and [Compile and runtime](05-compile-and-runtime.md).

## Top-level layout

```
RulesetSrc/
  BEMBase.txt              NRMF (and shared-include) data model
  BEMBase-SFam.txt         SFam data model
  T24NRMF/                 Nonresidential + multifamily (DataModel)
  T24SFam/                 Single-family (procedural)
  shared/                  Fragments included by both (BEMBase pieces, tables, RTF, some .rule files)
```

There is no `RulesetDev/Rulesets/` path in this GitHub repo. Older SVN and CLI comments still use that name; here everything lives under `RulesetSrc/`.

## NRMF vs SFam vs shared

| Tree | Sector | Manifest(s) | BEMBase | Enums | Screens |
|------|--------|-------------|---------|-------|---------|
| `T24NRMF/` | Nonres + multifamily | `T24N_2019.txt`, `T24N_2022.txt`, `T24N_2025.txt`, `T24N_2028.txt` | `RulesetSrc/BEMBase.txt` | `T24N_YYYY BEMEnums.txt` | `T24N_YYYY Screens.txt` |
| `T24SFam/` | Single-family | `Rules-2022.txt`, `Rules-2025.txt`, `Rules-2028.txt` | `RulesetSrc/BEMBase-SFam.txt` | `CAR22/CAR25 BEMEnums.txt` | `CAR22/CAR25 Screens.txt` |
| `shared/` | Both | Included via `--sharedPath1` / `-sharedPath` | `BEMBase_*.txt` fragments | `BEMEnums_*.txt` | `Screens_Res*.txt` |

Multifamily compliance uses the **NRMF** ruleset (`T24N_*`), not `T24SFam`. Shared residential objects (DHW, CSE, CUAC tables, RTF help) are pulled in from `shared/` at compile time.

## Year-specific manifests

### NRMF DataModel (`T24N_2025.txt`)

The compiler starts at this file. Typical sections, in order:

1. `FORMATVERSION` then `RULESET` — name, labels (`T24N_2025`), `FORMAT DataModel`, paths to compiled BEMBase / Screens / ToolTips
2. `TRANSFORMATIONS` — analysis copies of the model (`USER u`, `SIZING_PROPOSED zp`, `SIZING_BASELINE zb`, `ANNUAL_PROPOSED ap`, `ANNUAL_BASELINE ab`) plus `EXCLUDE` lines
3. `TABLEFILES` — CSV/TXT lookup tables (paths relative to the ruleset folder and `shared/`)
4. `LIBRARYFILES` — library `.rule` / `.txt` objects available to rules and the UI
5. `RULEFILES` — DataModel `.rule` files compiled into the binary
6. `RULESUBSET` blocks — extra tables hashed separately (hourly multipliers, CUAC, `RuleSubsetFileHashes`)

`T24N_2022.txt` / `T24N_2028.txt` are the same idea for other code years. Year-specific ACM numbers often live in `Ruleset-Version-YYYY.rule` plus year-suffixed table names.

### SFam procedural (`Rules-2025.txt`)

This file is itself a long compiler-format header (read it). Sections:

1. `RULESET_STRUCT_VERSION` (currently 5)
2. `RULESETID` / `RULESETVERSION`
3. `BEMBASEFILE` / `SCREENSFILE` / `TOOLTIPSFILE`
4. `TABLELIST` … `END` — named tables with search-column counts
5. `DATATYPES`, `RANGES`, `SYMBOLS`, `MAXCHILD`, `RESETS`, `UNIQUEASSIGNMENTS`, `LIBRARY`
6. `RULEFILE` lines — procedural `.rule` files
7. `RULESUBSET` — same hashing idea as NRMF (`25SFam_HourlyMultTables`, `SFam25_CUACTables`, …)

After the include list, the manifest can also contain `RULELIST` blocks (for example `Default_CodeVersion`).

## Naming conventions

- **NRMF rule files:** ACM-ish CamelCase, often `Area-Topic.rule` or `Area-Topic-T24N.rule` (year suffix when 2019/2022/2025 logic diverges). Example: `HVACSecondary-CoilCooling-DX.rule`.
- **SFam rule files:** `Rules_<phase>_<system>.rule` (`Rules_Default_HVAC.rule`, `Rules_CSE_Simulation_DHW.rule`).
- **Libraries:** `Library_*.rule` or `Library.txt`.
- **Do not** put DataModel `RULE`/`ENDRULE` blocks into a procedural `RULELIST` file, or curly-brace procedural assignments into a DataModel `RULE` body without understanding transforms. See [DataModel vs procedural](04-datamodel-vs-procedural.md).

## `RULESUBSET`

A subset is a named bundle of extra table files. The compiler hashes them (`RuleSubsetFileHashes` / `SFamRuleSubsetFileHashes`) so analysis can detect a mismatched hourly-multiplier pack. When you add a CSV to a subset, update the subset block **and** the hash table CSV the subset references.

## Compile vs deploy artifacts

Compile (see [05-compile-and-runtime.md](05-compile-and-runtime.md)) produces binaries under `CBECC/Data/Rulesets/`. The batch files then **copy** files the UI needs that are not inside the `.bin`:

| Source (example, 2025 NRMF) | Deployed |
|-----------------------------|----------|
| `T24N_2025 Screens.txt` | `Data/Rulesets/T24_2025/T24_2025 Screens.txt` |
| `T24N ToolTips.txt` | `…/T24_2025 ToolTips.txt` |
| `T24NRMF/RTF/*` and `shared/RTF/*` | `…/RTF/` |
| `shared/Screens_Res_2025.txt`, `shared/*.jpg`, `shared/*.png` | ruleset folder |

The compiled ruleset binary (`T24_2025.bin`, `CA Res 2025.bin`) embeds compiled tables, libraries, and rules. Screens, ToolTips, RTF, and images stay as sidecar files.

| Compiled output (2025) | Path under `CBECC/` |
|------------------------|---------------------|
| NRMF BEMBase | `Data/Rulesets/T24_2025/T24_2025 BEMBase.bin` |
| NRMF ruleset | `Data/Rulesets/T24_2025.bin` |
| SFam BEMBase | `Data/Rulesets/CA Res 2025/CAR25 BEMBase.bin` |
| SFam ruleset | `Data/Rulesets/CA Res 2025.bin` |

## Shared path

`BEMCompiler25.exe --sharedPath1="../RulesetSrc/shared/"` (and CBECC-CLI `-sharedPath`) is an extra include root. Manifest paths such as `"Tables\HPWHData_NEEA.csv"` resolve in the ruleset directory first, then in `shared/`. BEMBase record type `2` includes another BEMBase fragment by filename from those search paths.

## Adding a new `.rule` file

1. Write the file in the correct tree (`T24NRMF` vs `T24SFam` vs `shared`).
2. Add it to the year manifest (`RULEFILES` / `RULEFILE`). If it is shared, add it to every year that should ship it.
3. Recompile with `CBECC/CompileRules-T24_2025.bat` or `CompileRules_SFam_2025.bat` (or `CompileRules_ALL_2025.bat`).
4. Confirm the compile log (`_T24-2025 Rules Log.out` or `_Rules-SFam-2025 Log.out`) has no errors.
