# CBECC-Dev documentation

This folder holds three kinds of material. Use the tree that matches the job.

## Rules authoring (#1021)

How Title-24 rulesets in this repository are organized and how to write them.

Start at **[RulesetDev/README.md](RulesetDev/README.md)**.

## Engine, CLI, and INI (#675)

How shipped CBECC binaries behave: CBECC-CLI, BEMCompiler, OptionsCSV, and UI INI files.

Engine source of truth is [CBECC-software/cbecc](https://github.com/CBECC-software/cbecc). Those pages are written here so rules authors and vendors can use them without cloning the engine.

Start at **[Engine/README.md](Engine/README.md)**.

## End-user product manuals

- [quick-start-guide/quick-start-guide.md](quick-start-guide/quick-start-guide.md)
- `CBECC-25_UserManual_NRMF.pdf`
- `CBECC-25_UserManual_SFam.pdf`

These describe the CBECC user interface, not ruleset authoring.

## Supporting workbooks (not narrative docs)

- `T24N/`, `T24Res/` — ACM / table workbooks that feed ruleset CSVs
- `S901G/`, `SDD/` — older ASHRAE 90.1-G and SDD workbooks
- `WeatherData/` — weather files used by documentation or tests, not prose

## Historical

Word/PDF/SVN-era files that are still useful as source material but are **not** current authoring docs: **[Archive/README.md](Archive/README.md)**.
