# Compliance Manager API (vendor DLL)

Vendors who **do not** use CBECC-CLI can call the Compliance Manager DLL (`BEMCmpMgr25.dll` and year-suffixed siblings).

This page does **not** duplicate that API. The maintained write-up is:

- [`Documentation/CEC Compliance Manager software docu.docx`](../CEC%20Compliance%20Manager%20software%20docu.docx) (updated ~Nov 2025)
- The same file also lives in `CBECC-software/doc/`

Exports of interest (names evolve; trust the Word doc): `CMX_PerformAnalysisCB_NonRes`, `CMX_PerformAnalysisCB_CECRes`, `CMX_PerformBatchAnalysis_CECNonRes`, `CMX_PerformBatchAnalysis_CECRes`, plus results-CSV helpers.

Those APIs take the same **OptionsCSV** string as the CLI ([options-csv.md](options-csv.md)). They do not read the UI INI.

For command-line use, prefer [cli.md](cli.md).
