# CBECC engine surface (CLI, compiler, INI)

These pages document **how shipped CBECC binaries behave**. They live in this rules repository so authors and vendors can use them without cloning the engine.

**Source of truth for behavior:** [CBECC-software/cbecc](https://github.com/CBECC-software/cbecc) (`src/CBECC-CLI`, `src/BEMCompiler`, `src/BEMCmpMgr`, `src/BEMProc`, `src/CBECCUI`).

This is **not** ruleset authoring. For writing `.rule` files see [`../RulesetDev/`](../RulesetDev/README.md).

## Modules (year-suffixed binaries)

Each module compiles to year-suffixed names (`22`, `25`, `28` = energy code year). In this repo they sit under `CBECC/`.

| Module | Role |
|--------|------|
| **BEMProc** | Parses and compiles data model and ruleset source; applies rules at analysis time |
| **BEMCmpMgr** | Compliance analysis steps, report-generator contact, results export |
| **OS_Wrap** | OpenStudio / EnergyPlus wrap (NRMF) |
| **BEMCompiler** | CLI utility that compiles BEMBase + ruleset text to `.bin` |
| **CBECC-CLI** | Headless compile, single-model analysis, and batch for NRMF and SFam |
| **BEMProcUI** | Screen-definition DLL for the tabbed UI |
| **CBECCUI** | Main Windows UI (`CBECC-25.exe`) |
| **BEMTest** | Internal harness; not a supported product |

SFam analysis is implemented in the same CBECC product (from 2025.2.0). Ruleset **source and binaries** remain separate: `T24_YYYY` (NRMF) vs `CA Res YYYY` (SFam).

## Pages

- [CBECC-CLI](cli.md) — all modes and arguments; single vs batch for NRMF and SFam
- [OptionsCSV](options-csv.md) — analysis knobs passed to CompMgr
- [INI files](ini.md) — UI configuration vs OptionsCSV
- [BEMCompiler](bemcompiler.md) — `CompileRules-*.bat` and compiler flags
- [Compliance Manager API](compliance-manager-api.md) — vendor DLL exports (not the CLI)

Internal-only OptionsCSV notes (report-gen / security logging) are in [`internal/`](internal/README.md) and are **not** for the public wiki.

## Related

- Ruleset compile/runtime from an author perspective: [RulesetDev/05-compile-and-runtime.md](../RulesetDev/05-compile-and-runtime.md)
