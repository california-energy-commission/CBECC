# Compile and runtime

How ruleset source becomes binaries the UI and CLI load, and when named lists / transform blocks run.

Engine binaries and flags: [Engine/bemcompiler.md](../Engine/bemcompiler.md) and [Engine/cli.md](../Engine/cli.md). This page is the **author** view.

## Two compile paths

Daily work in this repo uses **BEMCompiler**, not CBECC-CLI.

| Path | Binary | How we invoke it |
|------|--------|------------------|
| **Batches (normal)** | `CBECC/BEMCompiler25.exe` | `CBECC/CompileRules-T24_2025.bat`, `CompileRules_SFam_2025.bat`, `CompileRules_ALL_2025.bat` (and 2022/2028 siblings) |
| **CLI compile** | `CBECC/CBECC-CLI25.exe` | `-CompileDataModel` then `-CompileRuleset` (same engine APIs, different front end) |

Both call `BEMPX_CompileDataModel` / `BEMPX_CompileRuleset` in BEMProc. The batches also **copy** Screens, ToolTips, RTF, and images that are not inside the `.bin`. CLI compile does **not** do those copies — after a CLI-only compile, run the `:copyfiles` section of the batch or copy the files yourself. See [01-ruleset-structure.md](01-ruleset-structure.md).

### Batch example (2025 NRMF)

From `CBECC/`:

```bat
BEMCompiler25.exe --sharedPath1="../RulesetSrc/shared/" --bemBaseTxt="../RulesetSrc/BEMBase.txt" --bemEnumsTxt="../RulesetSrc/T24NRMF/T24N_2025 BEMEnums.txt" --bemBaseBin="Data/Rulesets/T24_2025/T24_2025 BEMBase.bin" --rulesTxt="../RulesetSrc/T24NRMF/T24N_2025.txt" --rulesBin="Data/Rulesets/T24_2025.bin" --rulesLog="_T24-2025 Rules Log.out" --compileDM --compileRules
```

SFam uses `BEMBase-SFam.txt`, `CAR25 BEMEnums.txt`, `Rules-2025.txt`, and writes `Data/Rulesets/CA Res 2025.bin`.

Logs: `_T24-2025 Rules Log.out` and `_Rules-SFam-2025 Log.out` under `CBECC/`. Fix every compile error before considering a rule change done.

`--noResultGUI` / `--noSuccessGUI` suppress compiler message boxes (useful in automation). The batches do not pass those flags today.

## Deployed artifacts (2025)

| Artifact | Path under `CBECC/` |
|----------|---------------------|
| NRMF BEMBase | `Data/Rulesets/T24_2025/T24_2025 BEMBase.bin` |
| NRMF ruleset | `Data/Rulesets/T24_2025.bin` |
| NRMF Screens / ToolTips / RTF | `Data/Rulesets/T24_2025/` |
| SFam BEMBase | `Data/Rulesets/CA Res 2025/CAR25 BEMBase.bin` |
| SFam ruleset | `Data/Rulesets/CA Res 2025.bin` |
| SFam Screens / ToolTips / RTF | `Data/Rulesets/CA Res 2025/` |

The UI and CLI load these binaries plus weather under `Weather/2025/`. They do **not** read `RulesetSrc/` at analysis time.

## When lists run (NRMF)

Compliance Manager (`src/BEMCmpMgr/BEMCmpMgrCom.cpp`) drives analysis steps. DataModel transform blocks on each `RULE` run in those steps. `AnalysisThruStep` (OptionsCSV) can stop early; see [Engine/options-csv.md](../Engine/options-csv.md).

| Step | What runs |
|-----:|-----------|
| 1 | Analysis init: `DEFAULT`, pre-analysis checks, `CHECKSIM`, `CHECKCODE` |
| 2 | `SIZING_PROPOSED` / `SIZING_BASELINE` expressions (model copies `zp` / `zb`) |
| 3–4 | Generate and simulate sizing OSM/IDF |
| 5 | `ANNUAL_PROPOSED` / `ANNUAL_BASELINE` (`ap` / `ab`) |
| 6–7 | Annual sim, results, UMLH |
| 8 | Compliance report generation (`NRCCPRF` and report-gen) |

Named `RULELIST`s in `RULELISTFILES` are **not** those transform names. CompMgr and other rules call them with `EvalRulelist` (CSE input, CUAC, batch defaulting, CF1R, file hashes, …). Look for the list name in `BEMCmpMgrCom.cpp` or `EvalRulelist( "…" )` in `.rule` files.

## When lists run (SFam)

There is no `TRANSFORMATIONS` block. `src/BEMCmpMgr/BEMCmpMgrRes.cpp` evaluates named lists such as `ProposedInput` (and `ProposedInput_*` splinters), model-check lists, proposed-model prep, CSE simulation lists, budget conversion, results, and CF1R. Flag meanings: [04-datamodel-vs-procedural.md](04-datamodel-vs-procedural.md).

## Engine file map (cite, do not edit here)

| Engine file | Role |
|-------------|------|
| `src/BEMProc/expRuleFile.cpp` | Parses manifests, `RULE` / `RULELIST` files, tables |
| `src/BEMProc/expFormula.c` / `.h` | Expression parser, `functable[]`, evaluation |
| `src/BEMCompiler/BEMCompiler.cpp` | Compiler CLI |
| `src/CBECC-CLI/CBECC-CLI.cpp` | Analysis / compile / batch CLI |
| `src/BEMCmpMgr/BEMCmpMgrCom.cpp` | NRMF analysis steps, OptionsCSV |
| `src/BEMCmpMgr/BEMCmpMgrRes.cpp` | SFam analysis steps, OptionsCSV |

Repository: [CBECC-software/cbecc](https://github.com/CBECC-software/cbecc).
