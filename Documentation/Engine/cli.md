# CBECC-CLI

Engine surface for headless compile and analysis. **Source of truth:** [CBECC-software/cbecc](https://github.com/CBECC-software/cbecc) `src/CBECC-CLI/CBECC-CLI.cpp`. Binaries in this repo: `CBECC/CBECC-CLI25.exe` (and year-suffixed siblings).

This is **not** ruleset authoring ([RulesetDev](../RulesetDev/README.md)) and **not** the Python wrappers under `TestingCBECC`.

The CLI does **not** read a CBECC `.ini` file. Paths are flags. Analysis knobs are [`-OptionsCSV`](options-csv.md). Translating a UI INI: [ini.md](ini.md).

## Substitution block

Replace these once; examples below use the same names.

```text
ROOT      = C:/src/NOR-Codes-Stds/CBECC-Dev
CBECC     = C:/src/NOR-Codes-Stds/CBECC-Dev/CBECC
WEATHER   = C:/src/NOR-Codes-Stds/CBECC-Dev/Weather/2025
PROJECTS  = C:/src/NOR-Codes-Stds/CBECC-Dev/Projects/2025
RUN       = <your output directory>
```

A vendor install can use `C:/apps/CBECC25/` the same way: point `CBECC` at the install root that contains `CBECC-CLI25.exe` and `Data/Rulesets/`.

Run the EXE from `CBECC/` (or otherwise keep its DLLs on `PATH`). Arguments are case-insensitive. Unknown tokens print `(--unrecognized--)` and, if no primary mode was set, exit `5`.

Verified against `CBECC-CLI25.exe` in this repo (`-NotARealFlag` → `(--unrecognized--)`; recipes below).

## Modes (pick one)

| Flag | Mode | Engine |
|------|------|--------|
| `-CompileDataModel` | BEMBase text → `.bin` | `BEMPX_CompileDataModel` |
| `-CompileRuleset` | Ruleset text → `.bin` | `BEMPX_CompileRuleset` |
| `-Compliance` | Single-model analysis | NRMF or SFam (extension) |
| `-BatchRuns` | Batch NRMF | `CMX_PerformBatchAnalysis_CECNonRes` |
| `-SFamBatchRuns` | Batch SFam | `CMX_PerformBatchAnalysis_CECRes` |

**`-Compliance` sector:** if the model extension starts with `r` / `R` (`.ribd25`), SFam; otherwise NRMF (`.cibd25`). Multifamily is NRMF (`*.cibd*`), not SFam.

There is no `-CUAC` mode. CUAC is `CUACReportID` in OptionsCSV (or a CUAC property already in the model).

Day-to-day ruleset compile in this repo still uses [BEMCompiler batches](bemcompiler.md). CLI compile does not copy Screens / ToolTips / RTF.

## Arguments

| Argument | Value | Used by |
|----------|-------|---------|
| `-sharedPath` | Directory (repeatable) | Compile modes — extra include roots (`RulesetSrc/shared/`) |
| `-BEMBaseTxt` | BEMBase `.txt` | `-CompileDataModel` |
| `-BEMEnumsTxt` | Enums `.txt` | `-CompileDataModel` |
| `-BEMBaseBin` | BEMBase `.bin` | Compile ruleset; compliance; batch |
| `-RulesetTxt` | Manifest `.txt` | `-CompileRuleset` |
| `-RulesetBin` | Ruleset `.bin` | Compile ruleset; compliance; batch |
| `-LogFile` | Log path | Compile ruleset; batch log |
| `-WeatherPath` | Weather directory | Compliance; batch |
| `-ProcessingPath` | Working / output directory | Compliance; batch (optional) |
| `-ModelInput` | One project file, **or** an old-style batch CSV | `-Compliance`; batch scenario 3 |
| `-OptionsCSV` | `Name,Value,Name,Value,…` | Compliance; batch |
| `-ModelInputPath` | Directory of projects | Batch |
| `-BatchRunDefs` | Run-set definitions CSV | Batch (optional) |
| `-IncludeSubdirs` | `0` or `1` (default **`1`**) | Batch: recurse under `-ModelInputPath` |

No `-INI` / `-Config`. Proxy OptionsCSV for nested quoted strings is a CompMgr batch API argument; the CLI currently passes `NULL` for that slot — put non-nested proxy fields in `-OptionsCSV` if required (credentials: [internal addendum](internal/options-csv-private.md)).

### Batch scenarios

1. **Directory scan (usual):** `-ModelInputPath` set; `-ModelInput` and `-BatchRunDefs` empty. Processes `*.cibd*` (NRMF) or `*.ribd*` (SFam). `-IncludeSubdirs 1` (default) includes nested folders; `0` does not.
2. **Run-set CSV:** `-BatchRunDefs` plus `-ModelInputPath`. Example: `Projects/2025/batch-run-sets/Com 2025 CZ-Weather (16 runs).csv`.
3. **Old-style batch CSV:** `-ModelInput` is that CSV; other path flags unused. Rare.

Verified: `-IncludeSubdirs 0` on a folder with one `.cibd25` at the top and one in a subfolder → **1** successful NRMF run (`AnalysisThruStep,1`). `-IncludeSubdirs 1` on the same tree → **2** runs.

`RunsSpanClimates,1` on **NRMF** `-BatchRuns` fails while generating the batch file: *Batch processing option 'RunsSpanClimates' only compatible with SFam CBECC.* (CLI exit `6`).

## Exit codes

| Code | Meaning |
|-----:|---------|
| 0 | Success |
| 1 | MFC initialization failed |
| 2 | `GetModuleHandle` failed |
| 3 | Data model compilation failed |
| 4 | Ruleset compilation failed |
| 5 | Unrecognized primary function |
| 6 | Batch: could not generate the batch input/directives file |
| >1000 | Compliance: `1000 +` return from the analysis API |
| >2000 | Batch: `2000 +` return from the batch API |

SFam analysis return `22` with `PerformSimulations,0` / `BypassCSE,1` was observed (CLI `1022`). That OptionsCSV is only for a **parse check**. A real compliance run must allow simulations.

## Recommended OptionsCSV (headless)

```text
Silent,1,IsBatchProcessing,1,PreAnalysisCheckPromptOption,0,CompReportWarningOption,0,AnalysisDialogTimeout,1,PromptUserUMLHWarning,0,
```

Do **not** copy UI live defaults (`PreAnalysisCheckPromptOption,3` / `CompReportWarningOption,5`) into CLI; they prompt. Do not copy UI-only INI keys (`DeveloperMenu`, `AllowBlankSlate`) into OptionsCSV — unknown names are ignored, but they are not analysis knobs.

CLI-parsed OptionsCSV keys (in addition to CompMgr): `CUACReportID`, `RunsSpanClimates` (SFam batch only), `AllOrientationsResultsCSV` (SFam post-processing).

Full catalogs: [options-csv.md](options-csv.md). Report-gen / security keys: [internal/](internal/README.md).

## Compile

```text
CBECC-CLI25 -CompileDataModel ^
  -sharedPath "%ROOT%/RulesetSrc/shared/" ^
  -BEMBaseTxt "%ROOT%/RulesetSrc/BEMBase.txt" ^
  -BEMEnumsTxt "%ROOT%/RulesetSrc/T24NRMF/T24N_2025 BEMEnums.txt" ^
  -BEMBaseBin "%CBECC%/Data/Rulesets/T24_2025/T24_2025 BEMBase.bin"

CBECC-CLI25 -CompileRuleset ^
  -sharedPath "%ROOT%/RulesetSrc/shared/" ^
  -BEMBaseBin "%CBECC%/Data/Rulesets/T24_2025/T24_2025 BEMBase.bin" ^
  -RulesetTxt "%ROOT%/RulesetSrc/T24NRMF/T24N_2025.txt" ^
  -RulesetBin "%CBECC%/Data/Rulesets/T24_2025.bin" ^
  -LogFile "%CBECC%/_T24-2025 Rules Log.out"
```

Prefer `CompileRules-T24_2025.bat` so Screens/RTF copies run. See [bemcompiler.md](bemcompiler.md).

## Recipe 1 — NRMF single (`.cibd25`)

Verified: this command returned **0** with `AnalysisThruStep,1` (stops after analysis init; no E+ sim). For a full run, omit `AnalysisThruStep` or set it to `8` / `100`.

```text
CBECC-CLI25 -Compliance ^
  -BEMBaseBin "%CBECC%/Data/Rulesets/T24_2025/T24_2025 BEMBase.bin" ^
  -RulesetBin "%CBECC%/Data/Rulesets/T24_2025.bin" ^
  -WeatherPath "%WEATHER%/" ^
  -ProcessingPath "%RUN%/OffSml-Office_SZVAV - run/" ^
  -ModelInput "%PROJECTS%/non-residential/examples/OffSml-Office_SZVAV.cibd25" ^
  -OptionsCSV "Silent,1,IsBatchProcessing,1,PreAnalysisCheckPromptOption,0,CompReportWarningOption,0,AnalysisDialogTimeout,1,LogAnalysisMsgs,1,AnalysisThruStep,1,"
```

## Recipe 2 — SFam single (`.ribd25`)

Arguments and the SFam analysis path were verified on `1storyExampleIAQ.ribd25`. Use this OptionsCSV for a **real** run (simulations on). Detail dumps: `StoreBEMProcDetails` (SFam also accepts `StoreBEMDetails` as an alias).

```text
CBECC-CLI25 -Compliance ^
  -BEMBaseBin "%CBECC%/Data/Rulesets/CA Res 2025/CAR25 BEMBase.bin" ^
  -RulesetBin "%CBECC%/Data/Rulesets/CA Res 2025.bin" ^
  -WeatherPath "%WEATHER%/" ^
  -ProcessingPath "%RUN%/1storyExampleIAQ - run/" ^
  -ModelInput "%PROJECTS%/single-family/examples/1storyExampleIAQ.ribd25" ^
  -OptionsCSV "Silent,1,IsBatchProcessing,1,LogAnalysisMsgs,1,StoreBEMProcDetails,1,ExportHourlyResults_All,1,"
```

## Recipe 3 — NRMF batch

```text
CBECC-CLI25 -BatchRuns ^
  -BEMBaseBin "%CBECC%/Data/Rulesets/T24_2025/T24_2025 BEMBase.bin" ^
  -RulesetBin "%CBECC%/Data/Rulesets/T24_2025.bin" ^
  -WeatherPath "%WEATHER%/" ^
  -ModelInputPath "%PROJECTS%/non-residential/examples/" ^
  -ProcessingPath "%RUN%/nrmf-batch-out/" ^
  -LogFile "%RUN%/nrmf-batch-out/cli-batch.log" ^
  -IncludeSubdirs 0 ^
  -OptionsCSV "Silent,1,IsBatchProcessing,1,PreAnalysisCheckPromptOption,0,CompReportWarningOption,0,AnalysisThruStep,1,"
```

Optional: `-BatchRunDefs "%PROJECTS%/batch-run-sets/Com 2025 CZ-Weather (16 runs).csv"`.

## Recipe 4 — SFam batch (`RunsSpanClimates`)

Without spanning climates (parse/batch-file generation verified). `RunsSpanClimates,1` runs each project in all 16 CZs — expensive; equivalent to the SFam UI “Process each project for ALL Climates” box.

```text
CBECC-CLI25 -SFamBatchRuns ^
  -BEMBaseBin "%CBECC%/Data/Rulesets/CA Res 2025/CAR25 BEMBase.bin" ^
  -RulesetBin "%CBECC%/Data/Rulesets/CA Res 2025.bin" ^
  -WeatherPath "%WEATHER%/" ^
  -ModelInputPath "%PROJECTS%/single-family/examples/" ^
  -ProcessingPath "%RUN%/sfam-batch-out/" ^
  -LogFile "%RUN%/sfam-batch-out/cli-batch.log" ^
  -IncludeSubdirs 1 ^
  -OptionsCSV "Silent,1,IsBatchProcessing,1,LogAnalysisMsgs,1,StoreBEMProcDetails,1,"
```

Add `RunsSpanClimates,1,` only when you intend 16 climates per file.

## What the CLI does not do

- Load `CBECC-25.ini` / `CBECC-Res25.ini`
- Build OptionsCSV from the UI table (`PopulateAnalysisOptionsString`)
- Set `EnergyPlusPath` / `CSEPath` from INI `[paths]` (pass them in OptionsCSV or use CompMgr defaults next to the install)
- Copy Screens/RTF after compile (batches do)

Vendors who link the DLL instead of the EXE: [compliance-manager-api.md](compliance-manager-api.md).
