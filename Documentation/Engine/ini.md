# INI files (UI configuration)

The CBECC **Windows UI** stores paths and options in an INI file. **CBECC-CLI does not read INI.** To reproduce UI analysis on the CLI, copy analysis keys into [`-OptionsCSV`](options-csv.md) and pass paths as [CLI flags](cli.md).

**Source of truth for the UI tables:** `src/CBECCUI/MainFrm.cpp` (`PopulateAnalysisOptionsString`, `saCECNonResAnalOpts`) in [CBECC-software/cbecc](https://github.com/CBECC-software/cbecc).

Checked-in examples in this repo:

| File | Status |
|------|--------|
| [`CBECC/CBECC-25.ini`](../../CBECC/CBECC-25.ini) | Stub: `[paths] DataPath=Data\` |
| [`CBECC-Res-64/Data/CBECC-Res25.ini`](../../CBECC-Res-64/Data/CBECC-Res25.ini) | Commented SFam example (`[paths]` / `[files]` / `[options]` / tree colors) |

A full NRMF INI is written next to a user install of `CBECC-25.exe`, not as a complete file in Git.

## How the UI feeds analysis

`PopulateAnalysisOptionsString`:

1. Reads selected keys from `[options]` (or a batch alternate section).
2. Optionally reads `[proxy]` and `[paths]` (`EnergyPlusPath`, `CSEPath`).
3. Builds an OptionsCSV string.
4. Passes that string into the same CompMgr APIs the CLI uses.

INI is the UI configuration store. OptionsCSV is the analysis API. They overlap; they are not the same document.

The UI often copies a key into OptionsCSV **only when it differs from the table default**, so a live OptionsCSV string can be shorter than `[options]`.

## Sections

| Section | Role |
|---------|------|
| `[paths]` | Directories: data, projects, weather, EnergyPlus, CSE, rulesets |
| `[files]` | `BEMFile`, `RulesetFile` |
| `[options]` | Analysis + UI flags |
| `[proxy]` | Proxy type / address / credentials |
| `[limits]` | UI numeric limits (often empty) |
| `[DefaultNames]` | Default object names in the tree |
| `[AppendToTreeEntries]` | Extra property text on tree nodes |
| `[TextColors]` | RGB for user / default / library / sim result text |

## INI → CLI

| INI | CLI |
|-----|-----|
| `[files] BEMFile=` | `-BEMBaseBin` |
| `[files] RulesetFile=` | `-RulesetBin` (plus ruleset folder from `[paths]`) |
| `[paths] WeatherPath=` | `-WeatherPath` |
| `[paths] EnergyPlusPath=` / `CSEPath=` | OptionsCSV `EnergyPlusPath,"...",CSEPath,"...",` |
| `[options] StoreBEMDetails=1` | NRMF: `StoreBEMDetails,1,` — SFam: `StoreBEMProcDetails,1,` (alias `StoreBEMDetails` also works) |
| `[options] AnalysisThruStep=8` | `AnalysisThruStep,8,` (NRMF) |
| `[options] LogRuleEvaluation=1` | `Verbose,1,` |
| `[options] BypassOpenStudio_zb=1` | `BypassOpenStudio_zb,1,` |
| `[options] SimLoggingOption=1` | SFam: `LogCSESimulation,1,` |
| `[proxy] ProxyServerType=Http` | `ProxyServerType,Http,` |
| UI-only keys | Omit |

## Name mismatches

| Topic | Detail |
|-------|--------|
| Verbose logging | INI `LogRuleEvaluation` → OptionsCSV `Verbose` |
| SFam dumps | INI `StoreBEMDetails` → OptionsCSV `StoreBEMProcDetails` |
| SFam CSE log | INI `SimLoggingOption` → OptionsCSV `LogCSESimulation` |
| Batch include subdirs | INI `Batch_IncludeSubdirs` → CLI **flag** `-IncludeSubdirs` (not OptionsCSV) |
| Not INI → OptionsCSV | `DeveloperMenu`, `AllowBlankSlate`, tooltip flags, `EnableRulesetSwitching`, `ViewFootprintOption`, `Batch_*` extras, tree/color sections |
| Not OptionsCSV → INI | `Silent`, `IsBatchProcessing`, `CUACReportID`, `RunTitle`, `IDFToSimulate`, `RunsSpanClimates`, … |

## `[paths]` / `[files]` (typical)

| Key | Meaning |
|-----|---------|
| `DataPath` | Install `Data\` (NRMF stub) |
| `ProjectsPath` | Default Open dialog |
| `RulesetPath` | Directory of compiled rulesets |
| `WeatherPath` | EPW / DDY |
| `EnergyPlusPath` | EnergyPlus |
| `CSEPath` | CSE |
| `BEMFile` | Compiled BEMBase `.bin` |
| `RulesetFile` | Compiled ruleset `.bin` name |

## `[options]` — analysis (safe to map to CLI)

These names generally match OptionsCSV (except the mismatches above). See [options-csv.md](options-csv.md) for defaults and meaning.

NRMF-oriented: `StoreBEMDetails`, `AnalysisThruStep`, `BypassOpenStudio_*`, `ExportHourlyResults_*`, `SimulationStorage`, `AnalysisStorage`, `ParallelSimulations`, `ComplianceReportPDF`, `ComplianceReportXML`, `EnableResearchMode`, `LogWritingMode`, `VerboseInputLogging`, `DebugRuleEvalCSV`, `WriteMidAnalysisInputs`, `IncludePeakCooling` (SFam), `CUACReportID`, `ClassifyEditableDefaultsAsUserData`.

SFam-oriented (from `CBECC-Res25.ini` comments): `BypassCSE`, `BypassDHW`, `BypassRuleLimits`, `SimSpeedOption`, `DHWCalcMethod`, `EnableHPAutosize`, `EnableRHERS`, `CSE_DHWonly`, `StoreResultsToModelInput`, `AllOrientationsResultsCSV`, `ExportHourlyResults_u` / `_p` / `_s`, `SimReportDetailsOption`, `SimErrorDetailsOption`, `WriteCF1RXML`.

Prompt options in a **UI** INI are often `PreAnalysisCheckPromptOption=3` and `CompReportWarningOption=5`. For CLI, use `0`.

## `[options]` — UI-only (do not put on CLI)

| Key | Role |
|-----|------|
| `DeveloperMenu` | Extra UI menus |
| `AllowBlankSlate` | Allow empty new project |
| `EnableRulesetSwitching` | Ruleset menu |
| `CompatRulesetVerKey` | Filter rulesets listed in the UI |
| `EnableRptIncFile` | Report include file (UI) |
| `IncludeCompParamStrInToolTip` / `IncludeStatusStrInToolTip` | Tooltips |
| `ViewFootprintOption` | Geometry view |
| `Batch_AdditionalInputs` | Batch UI |
| `Batch_IncludeSubdirs` | Batch UI checkbox (CLI: `-IncludeSubdirs`) |
| `ComplianceReportPrompt` | Which PDF to open after analysis |
| Color / tree keys | Display only |

Report-generator **security** INI/OptionsCSV keys: [internal addendum](internal/options-csv-private.md).

## `[proxy]`

| Key | Notes |
|-----|-------|
| `UseProxyServerSettings` | UI toggle |
| `ProxyServerAddress` | `host:port` |
| `ProxyServerType` | `Http` (CBECC default), `Socks5`, `Default`, `No`, `HttpCaching`, `FtpCaching` |
| `ProxyServerCredentials` | Internal-only; do not publish |

## `[TextColors]`

RGB triplets such as `UndefinedR/G/B`, `ProgDefault*`, `RuleDefault*`, `RuleLibrary*`, `RuleDefined*`, `UserDefault*`, `UserLibrary*`, `UserDefined*`, `SimResult*`. UI only.
