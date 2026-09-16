# OptionsCSV — report generation and security (internal)

**Do not copy this page to the public CBECC wiki or vendor extracts.** It documents analysis keys that log or control **compliance report generation and report security**.

Public catalogs: [`../cli.md`](../cli.md), [`../options-csv.md`](../options-csv.md).

Engine: `src/BEMCmpMgr/BEMCmpMgrCom.cpp` / `BEMCmpMgrRes.cpp`.

## Keys to keep off the public page

| Option | Default | Why it stays internal |
|--------|--------:|------------------------|
| `SendRptSignature` | 1 | Whether the client sends the report-security signature with the report-gen request |
| `EnableRptGenStatusChecks` | 1 | Probe the report-generator service before analysis and before generate |
| `ReportGenVerbose` | 0 | Verbose logging of report-gen HTTP / website access |
| `LogReportRuleEvaluation` | 0 | Verbose logging of **report** rule evaluation (NRMF) |
| `ProxyServerCredentials` | (empty) | Username/password for the proxy (`username:password`) |
| `RptGenViaAnalysisResultsXML` | 0 | Secondary report gen via Analysis Results XML (SFam 2019+) |
| `RptGenConnectTimeout` | 10 | Seconds; report-gen connect |
| `RptGenReadWriteTimeout` | 480 | Seconds; report-gen read/write (`CECRptGenDefaultReadWriteTimeoutSecs`) |

Public docs may say that `ComplianceReportPDF` / `ComplianceReportXML` request reports, and that CLI should use `CompReportWarningOption,0`. They must not document how to disable signatures or how to talk to the report-gen service in detail.

## `CompReportWarningOption` (full)

NRMF CompMgr. CLI/batch default is **0**.

| Value | Behavior |
|------:|----------|
| 0 | No prompt; continue (engine/batch/CLI). |
| 1 | No prompt; abort if report gen unavailable; continue if **security is disabled**. |
| 2 | No prompt; abort if unavailable **or** security disabled. |
| 3 | Prompt if unavailable; continue without prompt if only security disabled. |
| 4 | Abort (no prompt) if unavailable; prompt if security disabled. |
| 5 | Prompt if unavailable **or** security disabled (UI live default). |

Do not recommend values other than `0` for unattended CLI.

## Related INI (UI)

`EnableRptGenStatusChecks`, `ReportGenVerbose`, proxy credentials, and `ComplianceReportPrompt` appear in UI INI files. They are not required to run analysis without reports. `EnableRptIncFile` is UI-oriented.

SFam rules also store report-gen host / schema application names on `Proj:` (for example `RptGenServer`, `SecKeyRLName`, `RptGenCheckURL`) inside `Default_CodeVersion` — those are **ruleset** values, not OptionsCSV.

## Proxy

`ProxyServerAddress` and `ProxyServerType` are listed on the public OptionsCSV page. **`ProxyServerCredentials` is not.** CompMgr batch APIs accept a separate proxy OptionsCSV argument; CBECC-CLI currently passes `NULL` for that argument.
