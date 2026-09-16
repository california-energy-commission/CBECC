# BEMCompiler

Year-suffixed compiler EXE (`BEMCompiler25.exe` in `CBECC/`). Compiles BEMBase text and ruleset text to the `.bin` files the UI and CLI load.

**Source of truth:** `src/BEMCompiler/BEMCompiler.cpp` in [CBECC-software/cbecc](https://github.com/CBECC-software/cbecc). Help output (verified): `BEMCompiler25.exe --help`.

Day-to-day in this repo: `CBECC/CompileRules-T24_2025.bat`, `CompileRules_SFam_2025.bat`, `CompileRules_ALL_2025.bat` (and 2022/2028 siblings). Author-facing flow: [RulesetDev/05-compile-and-runtime.md](../RulesetDev/05-compile-and-runtime.md).

CBECC-CLI `-CompileDataModel` / `-CompileRuleset` calls the same BEMProc APIs. The **batches** also copy Screens, ToolTips, RTF, and images; the CLI compile path does not.

## Flags

| Flag | Meaning |
|------|---------|
| `--sharedPath1` | Extra include directory (this repo: `../RulesetSrc/shared/`) |
| `--sharedPath2` | Second include directory |
| `--bemBaseTxt` | BEMBase source (`BEMBase.txt` or `BEMBase-SFam.txt`) |
| `--bemEnumsTxt` | Enumerations text |
| `--bemBaseBin` | Output (or input, when only compiling rules) BEMBase `.bin` |
| `--rulesTxt` | Year manifest (`T24N_2025.txt` / `Rules-2025.txt`) |
| `--rulesBin` | Output ruleset `.bin` |
| `--rulesLog` | Compile log |
| `--compileDM` | Compile data model |
| `--compileRules` | Compile ruleset (needs a BEMBase `.bin`) |
| `--noResultGUI` | No result message box |
| `--noSuccessGUI` | No success message box |
| `-h` / `--help` | Qt command-line help |
| `-v` / `--version` | Version |

The batches do not currently pass `--noResultGUI`. Automation should add it (and `--noSuccessGUI`) to avoid modal dialogs.

## 2025 NRMF (from `CompileRules-T24_2025.bat`)

Working directory: `CBECC/`.

```bat
BEMCompiler25.exe --sharedPath1="../RulesetSrc/shared/" --bemBaseTxt="../RulesetSrc/BEMBase.txt" --bemEnumsTxt="../RulesetSrc/T24NRMF/T24N_2025 BEMEnums.txt" --bemBaseBin="Data/Rulesets/T24_2025/T24_2025 BEMBase.bin" --rulesTxt="../RulesetSrc/T24NRMF/T24N_2025.txt" --rulesBin="Data/Rulesets/T24_2025.bin" --rulesLog="_T24-2025 Rules Log.out" --compileDM --compileRules
```

On success the batch copies `T24N_2025 Screens.txt`, `T24N ToolTips.txt`, RTF, images, and `shared/Screens_Res_2025.txt` into `Data/Rulesets/T24_2025/`.

## 2025 SFam (from `CompileRules_SFam_2025.bat`)

```bat
BEMCompiler25.exe --sharedPath1="../RulesetSrc/shared/" --bemBaseTxt="../RulesetSrc/BEMBase-SFam.txt" --bemEnumsTxt="../RulesetSrc/T24SFam/CAR25 BEMEnums.txt" --bemBaseBin="Data/Rulesets/CA Res 2025/CAR25 BEMBase.bin" --rulesTxt="../RulesetSrc/T24SFam/Rules-2025.txt" --rulesBin="Data/Rulesets/CA Res 2025.bin" --rulesLog="_Rules-SFam-2025 Log.out" --compileDM --compileRules
```

Then copies `CAR25 Screens.txt`, `T24R ToolTips.txt` to `CAR25 ToolTips.txt`, RTF, and images into `Data/Rulesets/CA Res 2025/`.
