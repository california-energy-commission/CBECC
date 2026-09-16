# OptionsCSV

Analysis knobs passed into Compliance Manager. Used by CBECC-CLI `-OptionsCSV`, the CBECC UI (built from INI), BEMTest, and CompMgr APIs.

**Source of truth:** `src/BEMCmpMgr/BEMCmpMgrCom.cpp` (NRMF) and `src/BEMCmpMgr/BEMCmpMgrRes.cpp` (SFam) in [CBECC-software/cbecc](https://github.com/CBECC-software/cbecc). Parsers: `ParseCSVRecord` / `GetCSVOption*` in `src/BEMCmpMgr/BEMCmpMgr_19.cpp`.

Report-generation and report-security keys are **not** listed here. They live in [internal/options-csv-private.md](internal/options-csv-private.md) (do not copy to the public wiki).

CLI usage and recipes: [cli.md](cli.md). UI INI mapping: [ini.md](ini.md).

## Format

```
Name,Value,Name,Value,...
```

- Names are case-insensitive.
- Integers: `atoi`. Flags are **on when the integer is > 0**.
- Strings: quote values that contain commas: `EnergyPlusPath,"C:/Program Files/EnergyPlus",`.
- Unknown names are **ignored** (no error).
- If a key is omitted, CompMgr uses the default in the tables below.

## Run-type suffixes (NRMF)

| Suffix | Meaning |
|--------|---------|
| `_zp` | Sizing proposed |
| `_zb` | Sizing baseline (T24N) |
| `_zb1`…`_zb4` | Baseline sizing by orientation (90.1 / ECBC) |
| `_ap` | Annual proposed |
| `_ab` | Annual baseline (T24N) |
| `_ab1`…`_ab4` | Annual baseline by orientation |
| `_All` | Apply to all relevant run types |

Skip baseline OS/E+ (still run proposed):

```text
BypassOpenStudio_zb,1,BypassOpenStudio_ab,1,
```

Do not set `BypassOpenStudio_all` if you still want proposed simulations.

## CLI / automation defaults

```text
Silent,1,IsBatchProcessing,1,PreAnalysisCheckPromptOption,0,CompReportWarningOption,0,AnalysisDialogTimeout,1,PromptUserUMLHWarning,0,
```

UI live analysis often uses prompt option `3` / `5`. Those can dialog; avoid them on the CLI.

## CLI-only keys (parsed in CBECC-CLI.cpp)

| Option | Default | Notes |
|--------|--------:|-------|
| `CUACReportID` | 0 | `>0` → CUAC path |
| `RunsSpanClimates` | 0 | **SFam batch only.** NRMF batch errors (CLI exit 6) |
| `AllOrientationsResultsCSV` | 1 | SFam: extra CSV rows when `Proj:AllOrientations` is on |

## `AnalysisThruStep` (NRMF)

| Step | Description |
|-----:|-------------|
| 1 | Init: DEFAULT / PREANALYSISCHECK / CHECKSIM / CHECKCODE |
| 2 | SIZING_PROPOSED / SIZING_BASELINE rules |
| 3 | Generate sizing OSM & IDF |
| 4 | Simulate sizing models |
| 5 | ANNUAL_PROPOSED / ANNUAL_BASELINE rules |
| 6 | Generate annual OSM & IDF |
| 7 | Annual sim, results, UMLH |
| 8 | Compliance report generation |

Default `100` ≈ all steps. `AnalysisThruStep,1` is a valid way to prove CLI parsing without EnergyPlus (verified).

## `PreAnalysisCheckPromptOption` (NRMF)

| Value | Behavior |
|------:|----------|
| 0 | No prompt; abort on errors only (CLI/batch) |
| 1 | No prompt; abort on errors or warnings |
| 2 | Abort on errors; prompt on warnings |
| 3 | Prompt on errors or warnings (UI live) |

## `CompReportWarningOption` (NRMF)

Use **`0`** on the CLI (no prompt; continue). Other values combine report-generator availability with report-security policy; they are documented only in the [internal addendum](internal/options-csv-private.md).

---

## NRMF (nonresidential / multifamily)

Consumed by `CMX_PerformAnalysisCB_NonRes`. Defaults are CompMgr defaults when the option is omitted.

### Flags and integers

| Option | Default | Purpose |
|--------|--------:|---------|
| `Verbose` | 0 | Verbose logging. UI INI often `LogRuleEvaluation`. |
| `StoreBEMDetails` | 0 | Write `.ibd-Detail*` dumps |
| `WriteMidAnalysisInputs` | follows `StoreBEMDetails` | Write `*.cibd##i` for each generated model. Independent of detail dumps if set explicitly. |
| `Silent` | 0 | Suppress message boxes |
| `AnalysisThruStep` | 100 | Stop after step *N* |
| `DontAbortOnErrorsThruStep` | 0 | Continue through step *N* despite errors |
| `BypassInputChecks` | 0 | Skip input checks |
| `BypassUMLHChecks` | 0 | Skip unmet load hour checks |
| `BypassPreAnalysisCheckRules` | 0 | Skip pre-analysis check rules |
| `BypassCheckSimRules` | 0 | Skip CHECKSIM |
| `BypassCheckCodeRules` | 0 | Skip CHECKCODE |
| `BypassValidFileChecks` | 0 | Skip valid-file / hash checks |
| `DurationStats` | 0 | Timing stats |
| `IgnoreFileReadErrors` | 0 | Continue after file read errors |
| `PurgeUnreferencedObjects` | 1 | Purge unreferenced objects |
| `ModelRpt_ALL` | 0 | All ruleset model report types |
| `ComplianceReportPDF` | 0 | Request PDF compliance report |
| `ComplianceReportXML` | 0 | Request XML compliance report |
| `ComplianceReportStd` | 0 | Standard-design compliance report |
| `BypassRecircDHW` | 0 | Skip recirculating DHW / CSE DHW |
| `SimulationStorage` | 1 | E+ intermediate retention |
| `AnalysisStorage` | 2 | Analysis intermediate retention |
| `ParallelSimulations` | 1 | Parallel sims when possible |
| `LogWritingMode` | 100 | Log flush mode (UI often 2) |
| `QuickAnalysis` | -1 | `-1` = leave project |
| `WriteRulePropsToResultsXML` | 0 | Extra properties in results XML |
| `WriteUMLHViolationsToFile` | 1 | UMLH text file |
| `LogRecircDHWSimulation` | 0 | Alias with `LogCSESimulation` |
| `LogCSESimulation` | 0 | CSE logging |
| `PromptUserUMLHWarning` | 0 | Forced off if `Silent` or `DontAbortOnErrorsThruStep` > 6 |
| `PerformDupObjNameCheck` | 1 | Duplicate object names |
| `PreAnalysisCheckPromptOption` | 0 | See table above. UI live often 3. |
| `CompReportWarningOption` | 0 | Use 0 on CLI |
| `AnalysisDialogTimeout` | 20 | Seconds. UI batch often 1. |
| `ReportStandardUMLHs` | 0 | Standard-model UMLH |
| `ReportAllUMLHZones` | 0 | UMLH for all zones |
| `SimulateCSEOnly` | 0 | CSE-only path where applicable |
| `IsBatchProcessing` | 0 | Stored on `Proj`; suppresses some prompts |
| `ReportGenNRCCPRFXML` | 1 | NRCC-PRF XML |
| `LogAnalysisProgress` | -1 | Progress logging |
| `EnableResearchMode` | 0 | Research mode if not in input |
| `AllowProposedPVBattery` | -1 | `-1` = use model |
| `AllowStandardPV` | -1 | `-1` = use model |
| `AllowStandardBattery` | -1 | `-1` = use model |
| `SimSpeedOption` | -1 | CSE speed (`-1` = model) |
| `DownloadVerbose` | -1 | Weather/rate download logging |
| `ExportHourlyResults_ap` / `_ab` | 0 | Hourly export per annual run |
| `ExportHourlyResults_All` | 0 | If `>0`, sets both `_ap` and `_ab` |
| `SimOutputVariablesToCSV_zp` / `_zb` / `_ap` / `_ab` | 0 | E+ output variables CSV |
| `SimOutputVariablesToCSV_All` | 0 | Enables all four |
| `NumFileOpenDefaultingRounds` | 3 | Defaulting rounds on load |
| `UseEPlusRunMgr` | 1 | EnergyPlus run manager |
| `CUACReportID` | 0 | `>0` CUAC |
| `LogCUACBillCalcDetails` | -1 | CUAC bill logging |
| `CustomMeterOption` | -1 | `>0` invalidates normal compliance reporting |
| `LogAnalysisMsgs` | 0 | Extra messages without full `Verbose` |
| `BypassOpenStudio_all` | 0 | Bypass OS/E+ for every run type |
| `BypassOpenStudio_zp` / `_zb` / `_ap` / `_ab` | 0 | Per T24N run |
| `BypassOpenStudio_zb1`…`zb4` / `_ab1`…`ab4` | 0 | Per orientation run |
| `OverrideAutosize_zp` / `_zb` / `_ap` / `_ab` | -1 | `-1` = none |
| `OverrideAutosize_zb1`… / `_ab1`… | -1 | Per orientation |
| `VerboseInputLogging` | 0 | Verbose defaulting-rule logging |

### Strings (NRMF)

| Option | Purpose |
|--------|---------|
| `ProxyServerAddress` | HTTP proxy host |
| `ProxyServerType` | e.g. `Http` |
| `NetComLibrary` | Legacy network library |
| `RunTitle` | Sets `Proj:RunTitle` |
| `IDFToSimulate` | Force this IDF (must exist) |
| `ModelkitPath` | Modelkit tools (HybridCooling) |
| `CUACElecTariffFile` / `CUACGasTariffFile` | CUAC tariffs |
| `DebugRuleEvalCSV` | Targeted rule-eval debug CSV |
| `EnergyPlusPath` | EnergyPlus directory |
| `CSEPath` | CSE directory |

Proxy **credentials** are internal-only.

### Dynamic model reports

Any `Proj:RuleReportType` enumeration name may appear as a flag. If value `>0`, or `ModelRpt_ALL` is set, that report is generated. Names are ruleset-specific.

### Not consumed in NonRes (do not rely on)

| Option | Notes |
|--------|-------|
| `MaxNumErrorsReportedPerType` | Parse commented out |
| `AllowAnalysisAbort` | Parse commented out; abort allowed is hard-coded `true` |
| `FileSaveOnlyValidInputs` | In the UI INI table; no matching Com `GetCSVOption*` |

---

## SFam (single-family)

Consumed by `CMX_PerformAnalysisCB_CECRes` when `-Compliance` sees a `.ribd*` extension.

`StoreBEMProcDetails` is the Res name. `StoreBEMDetails` is accepted as an alias (engine 2026). INI files still often say `StoreBEMDetails`.

| Option | Default | Purpose |
|--------|--------:|---------|
| `Verbose` | 0 | Verbose logging |
| `FullComplianceAnalysis` | 1 | Full proposed+standard when applicable |
| `InitHourlyResults` | 1 | Initialize hourly results |
| `StoreBEMProcDetails` | 0 | Detailed BEM dumps |
| `StoreBEMDetails` | 0 | Alias of the previous |
| `WriteMidAnalysisInputs` | follows `StoreBEMProcDetails` | Write `*.ribd##i` mid-analysis |
| `PerformRangeChecks` | 1 | Range checks |
| `PerformDupObjNameCheck` | 1 | Duplicate names |
| `PerformSimulations` | 1 | Run CSE. `0` is a parse-only shortcut; analysis returns a non-success code |
| `BypassCSE` | 0 | Skip CSE |
| `BypassDHW` | 0 | Skip DHW |
| `IgnoreFileReadErrors` | 0 | Continue after read errors |
| `ComplianceReportPDF` / `ComplianceReportXML` | 0 | Request reports |
| `Silent` | 0 | Suppress prompts |
| `BypassRuleLimits` | 0 | Bypass certain rule limits |
| `BypassValidFileChecks` | 0 | Skip valid-file checks |
| `SimSpeedOption` | -1 | CSE speed |
| `AllowAnalysisAbort` | 1 | Res still reads this |
| `StoreResultsToModelInput` | 0 | Store results back into the model |
| `StoreDetailedResultsToModelInput` | 0 | Detailed results into the model |
| `DHWCalcMethod` | -1 | DHW engine override |
| `EnableResearchMode` | 0 | Research mode |
| `EnableMixedFuelCompare` | 0 | Mixed-fuel compare |
| `SimulateCentralDHWBranches` | 1 | Central DHW branches |
| `AllowNegativeDesignRatings` | 0 | Negative design ratings |
| `EnableCO2DesignRatings` | 0 | CO2 design ratings |
| `EnableHPAutosize` | 0 | HP autosize |
| `EnableRHERS` | 0 | RHERS |
| `LogWritingMode` | 100 | Log write mode |
| `LogCSESimulation` | 0 | CSE logging (INI older name: `SimLoggingOption`) |
| `SimReportDetailsOption` | 1 | 0 none / 1 user reports / 2 entire `.rpt` |
| `SimErrorDetailsOption` | 1 | CSE errors in output |
| `WriteCF1RXML` | 1 | Write CF1R XML |
| `CSE_DHWonly` | 0 | CSE DHW-only |
| `ShuffleSFamDHW` | -1 | Shuffle SFam DHW |
| `IncludePeakCooling` | -1 | Peak cooling (2025+); `-1` = ruleset default |
| `DownloadVerbose` | -1 | Download verbose |
| `ClassifyEditableDefaultsAsUserData` | 0 | Treat editable defaults as user data |
| `LogAnalysisMsgs` | 0 | Extra analysis messages |
| `CUACReportID` | 0 | CUAC |
| `LogCUACBillCalcDetails` | -1 | CUAC bill details |
| `AllOrientationsResultsCSV` | 1 | All-orientation CSV rows |
| `ExportHourlyResults_All` | 0 | Hourly export all SFam run IDs |
| `ExportHourlyResults_u` / `_p` / `_p_N` / `_p_E` / `_p_S` / `_p_W` / `_s` | 0 | Hourly export by run |
| `FileSaveAllDefinedProperties` | 0 | Save all defined properties (some paths) |
| `RunsSpanClimates` | 0 | SFam **batch** only: all 16 CZs |

**Strings (SFam):** `AltWeatherPath`, `ProxyServerAddress`, `ProxyServerType`, `NetComLibrary`, `BatchAnalysisType`, `CUACElecTariffFile`, `CUACGasTariffFile`, `DebugRuleEvalCSV`, `BatchPath`, `BatchFile`.
