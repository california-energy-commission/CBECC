# BEMBase and support files

The data model (what objects and properties exist) is **not** defined only in `.rule` files. BEMBase, enums, screens, and (for SFam) datatype/range/symbol tables are first-class source.

The record formats are documented in the **headers of the files themselves**. This page tells you which file to open and how the pieces fit. Engine parser: `src/BEMProc` in [CBECC-software/cbecc](https://github.com/CBECC-software/cbecc).

## BEMBase (class and property schema)

| File | Used by |
|------|---------|
| [`RulesetSrc/BEMBase.txt`](../../RulesetSrc/BEMBase.txt) | NRMF (`T24N_*`) |
| [`RulesetSrc/BEMBase-SFam.txt`](../../RulesetSrc/BEMBase-SFam.txt) | SFam (`Rules-YYYY`) |
| [`RulesetSrc/shared/BEMBase_*.txt`](../../RulesetSrc/shared/) | Included fragments (CSE, Res DHW, CUAC, CF1R, …) |

Format version integer (e.g. `1007`) sits near the top after the header. Record types:

| Header | Meaning |
|--------|---------|
| `-1` | End of file |
| `0` | New **class** (short name, long name, default name template, max definable, parent class names, max children, copy/XML flags, help ID) |
| `1` | New **property** on the current class (type, array count, simulation-write flag, user spec, units, referenced object classes, long name) |
| `2` | Include another BEMBase file |

Property types you will see: `BEMP_Int`, `BEMP_Flt`, `BEMP_Str`, `BEMP_Sym` (enumeration), `BEMP_Obj` (reference to another object).

User specification (`US`) on a BEMBase property is the schema default. DataModel `INPUTCLASS` in a `RULE` can refine it per ruleset. Values:

- `Comp` / Compulsory — must be entered when the object is created
- `Req` / Required
- `CReq` / CondRequired
- `Opt` / Optional
- `Def` / Default
- `Pres` / Prescribed (not user-enterable except research mode)
- `NInp` / NotInput (calculated or code-prescribed)

Parent/child is defined on the class `0` record (`P1`–`P20`). Name templates can use tokens such as `<pn>` (parent name) and `<i>` (1-based index of this class). That is the BEM parent-child model.

`RULE NEW` in a DataModel file can **add** a property that is not in BEMBase. Prefer BEMBase for properties that belong in the shared schema; use `RULE NEW` for ruleset-only or reporting fields.

## Enumerations (BEMEnums)

| File | Used by |
|------|---------|
| `T24NRMF/T24N_YYYY BEMEnums.txt` | NRMF year |
| `T24SFam/CAR25 BEMEnums.txt` (and `CAR22`) | SFam |
| `shared/BEMEnums_*.txt` | Shared Res / CSE / CUAC lists |

DataModel `OPTION` blocks on a `RULE` can override or supplement enums for that property. If both exist, the rule `OPTION` wins for that property.

SFam also has `Symbols.txt` (see below) for selection lists that are not always in BEMEnums.

## Screens and ToolTips (UI only)

| Kind | NRMF | SFam |
|------|------|------|
| Screens | `T24N_YYYY Screens.txt` | `CAR25 Screens.txt` |
| ToolTips | `T24N ToolTips.txt` | `T24R ToolTips.txt` |
| Extra Res screens copied into NRMF deploy | `shared/Screens_Res_2025.txt` | — |

These drive the tabbed UI (`BEMProcUI`). They are **copied** next to the compiled BEMBase; they are not inside the ruleset `.bin`. Changing a screen file does not require a full ruleset recompile, but the copy step in `CompileRules-*.bat` must run for the UI to see it.

RTF files under `T24NRMF/RTF/`, `T24SFam/RTF/`, and `shared/RTF/` are help topics referenced from screens.

## SFam-only tables referenced from the manifest

These are **procedural** support files, listed in `Rules-2025.txt`:

| Manifest keyword | File | Role |
|------------------|------|------|
| `DATATYPES` | `Datatype.txt` (and `Datatype-22.txt`) | Per-property compliance data type and UI flags (Compulsory / Required / … / display columns) |
| `RANGES` | `Ranges.txt` | Min/max and conditional ERROR/WARNING/MESSAGE checks |
| `SYMBOLS` | `Symbols.txt` | Conditional enumeration lists |
| `MAXCHILD` | `MaxChild.txt` | Max children by type (mostly unused today) |
| `RESETS` | `Resets.txt` | UI resets: when the user edits property A, clear property B |
| `UNIQUEASSIGNMENTS` | `UniqueAssignments.txt` | When assignments of a component type must be unique (`Proj:UniqueAssignFlag`) |
| `LIBRARY` | `Library.txt`, `Library_Schedules-T24N.rule` | Library objects |

Each of those files starts with its own format header. NRMF encodes the same ideas inside DataModel `RULE` blocks (`INPUTCLASS`, `MINIMUM`/`MAXIMUM`, `RESETS`, `OPTION`) plus BEMBase.

## Libraries

Library files define reusable objects (constructions, schedules, HVAC performance curves, holidays, PV/battery). NRMF lists them under `LIBRARYFILES` in `T24N_YYYY.txt`. SFam uses `LIBRARY` in the procedural manifest.

Rules pull library components with `RuleLibrary(...)`. Unused HVAC library objects left in a model can fail analysis; batch/sensitivity work often creates objects from the library then deletes unused ones.

## Lookup tables

- **NRMF:** `TABLEFILES` in the manifest; CSV files under `T24NRMF/Tables/` and `shared/Tables/`.
- **SFam:** `TABLELIST` with a table name, file, search-parameter count, and data-column count. Newer tables use `0, 0` for the “new table format.”
- Workbooks that generate many of those CSVs live under `Documentation/T24N/` and `Documentation/T24Res/`. Edit the workbook, export CSV into `RulesetSrc`, then recompile.

DataModel table lookup in a rule looks like:

```
BaseVerticalFenPerformance:UFactor("ProductType", bz:FenProdType, "OccupancyClass", bz:FenestrationOccupancy)
```

See [Syntax Examples/Table Lookup Syntax.rule](Syntax%20Examples/Table%20Lookup%20Syntax.rule) and [rule language](03-rule-language.md).

## Input data model dumps

Each CBECC release can write human-readable “Input Data Model” text under the install `Data` directory (described in the Compliance Manager Word doc). That dump is generated from BEMBase + ruleset INPUTCLASS, not hand-edited in this repo.
