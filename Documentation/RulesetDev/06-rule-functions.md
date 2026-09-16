# Expression functions

Registered in engine `static FuncTable functable[]` in [`src/BEMProc/expFormula.c`](https://github.com/CBECC-software/cbecc/blob/master/src/BEMProc/expFormula.c). Behavior is the `BEMPFunc` / `UnaryFunc` / `BinaryFunc` switch in the same file.

This repository does not contain that source. Names here were taken from the CBECC-software `master` copy of `functable[]` (including `OrderAssignments` and `SetObjectFlagBySum`, which the VS Code highlighter does not yet list).

Use **long names** in new rules. `#` aliases work but are harder to read. `VAR_ARGS` means the compiler accepts a range of argument counts; see the notes below rather than guessing extra arguments.

Shared grammar: [03-rule-language.md](03-rule-language.md).

## Math and strings

| Function | Args | What it does |
|----------|------|----------------|
| `log` / `log10` / `exp` / `pow` / `sqrt` / `abs` | 1 or 2 (`pow`) | Natural log, log10, e^x, power, square root, absolute value |
| `sin` `asin` `cos` `acos` `tan` `atan` | 1 | Trig; angles in **radians** |
| `int` | 1 | Truncate toward zero |
| `mod(a,b)` | 2 | Remainder |
| `round(x, digits)` | 2 | Round `x` to `digits` decimal places |
| `min(a,b)` / `max(a,b)` | 2 | Nest for more than two values |
| `strupper` / `strlower` | 1 | Case fold a string |
| `strlen` | 1 | Character length |
| `find(hay, needle)` / `findnocase` | 2 | 0-based index, or negative if missing (see `BatchRuns.rule` tests for `":"`) |
| `substr(s, first, length)` | 3 | `first` is 0-based |
| `ftoa` / `atof` / `StrToFlt` | 1 | Float ↔ string; `StrToFlt` honors locale |
| `FltToStr` | var | Format a number as text |
| `Format` / `FormatNL` | var | sprintf-style; `FormatNL` allows embedded newlines in the format string |
| `FindInString` / `ReplaceInString` | 2 | Search / replace inside a string |

## Navigation

The current object is the object whose `RULE` or `RULELIST` assignment is running.

| Function | Typical args | What it does |
|----------|----------------|----------------|
| `Local( prop [, index] )` | property name | Read on the current object |
| `Parent` / `Parent2` / `Parent3` | property name | Ancestor objects |
| `Global( Class:Prop [, index] )` | often `Proj:…` | Any class; not necessarily an ancestor |
| `LocalRef` / `ParentRef` / `Parent2Ref` / `Parent3Ref` / `GlobalRef` | reference property, then target property | Follow an object pointer, then read |
| `ChildRef( class, index )` | 2 | 1-based child of that class |
| `ChildIndex` / `ComponentIndex` | var | Index of the current or named object |
| `ComponentType` / `ParentComponentType` / `Parent2ComponentType` / `Parent3ComponentType` | 0 or 1 | Class name string |
| `CompName` / `CompExists` | 2 | Name or existence of a class instance |
| `ComponentCount( class )` | 1 | How many objects of that class exist |
| `ChildCount( class )` | 1 | How many **children** of that class the current object has |

## Aggregation

| Function | What it does |
|----------|----------------|
| `SumChildren( prop, … )` | Sum `prop` on children |
| `SumChildrenIf( … )` | Same, with a condition (conditional operators allowed) |
| `SumAll( prop )` | Sum `prop` on all objects of that property’s class |
| `SumRevRef` / `SumRevRefEx` | Sum on objects that **reference** the current object |
| `SumAcrossIf` / `MaxAcrossIf` / `MinAcrossIf` | Conditional scan across objects |
| `MaxChild` / `MinChild` / `MaxAll` / `MinAll` | Max/min on children or all instances |
| `MaxRevRef` / `MinRevRef` | Max/min via reverse references |
| `MaxChildComp` / `MinChildComp` / `MaxAllComp` / `MaxRevRefComp` | Return the **object** that holds the max/min, not the value |
| `NumUniqueChildVals` | Count distinct child values |
| `ScheduleSum` | Sum schedule values (SFam / CSE-related) |

`SumChildrenIf`, `SumAcrossIf`, `MaxAcrossIf`, `MinAcrossIf`, `IfValidAnd`, `ListRevRefIf`, and `UniqueListRevRefIf` are the functions that allow comparison operators in extra arguments (`CurrentFunctionAllowsConditionalOperators` in `expFormula.c`).

## Validity and status

Property **status** is a small integer (user / library / default / undefined, …). `LocalStatus( prop ) < 1` is the usual “not a real user/library value” test in current rules.

| Function | What it does |
|----------|----------------|
| `LocalValid` / `ParentValid` / `Parent2Valid` / `Parent3Valid` / `GlobalValid` | 1 if that property currently has a valid value |
| `IfValidAnd( expr, … )` | Conjunction that only uses arguments that are valid; used as a guard before arithmetic |
| `ValidOr( value, fallback )` | `value` if valid, else `fallback` |
| `LocalIsDefault` / `ParentIsDefault` | 1 if the stored value is a default, not user input |
| `LocalSymbolInvalid` | 1 if the enumeration selection is not a legal symbol |
| `LocalCompAssigned` / `ParentCompAssigned` / `GlobalCompAssigned` | 1 if the object-reference property points at an object |
| `EnsureSymbolExists` / `EnsureChildAssigned` | Repair/ensure helpers used in older lists |
| `EnumValue` / `SymValue` | Integer for an enumeration name |
| `EnumString` / `LocalSymbolString` / `ParentSymbolString` / `*RefSymbolString` | Name string for an enumeration |
| `LocalSymbolValue` / `ParentSymbolValue` / `GlobalSymbolValue` / `*RefSymbolValue` | Numeric/enum value via the same navigation family |

Example:

```
if( IfValidAnd( CommKitArea > 0 ) )
then  ValidOr( VentFlow, 0 )
else  0
endif
```

## Model mutation

These **change the building model**. Use them in named lists (prep, CSE, budget), not as a substitute for `DEFAULT` on a simple calculated property.

| Function | What it does |
|----------|----------------|
| `EvalRulelist( "Name" [, flags] )` | Run another `RULELIST` |
| `RuleLibrary( class, name [, extra] )` | Pull a library component into the model |
| `CreateComp` / `CreateCompFor` / `CreateChildren` | Create objects |
| `AssignOrCreateComp` | Reuse an existing object or create one |
| `CopyComp` | Duplicate an object |
| `DeleteComp` / `DeleteChildren` / `DeleteAllComps` | Remove objects (batch/sensitivity lists delete unused HVAC this way) |
| `ImportComponentFromFile` | Load a component from a file |
| `OrderAssignments( class, property )` | 2 args; CUAC — order assignment lists |
| `SetObjectFlagBySum` | Set a flag from a sum across objects (2025) |
| `SetNextArrayElement` | Append into the next unused array slot |

Library leftover HVAC objects in a model can fail simulation. Create from `RuleLibrary`, then `DeleteComp` anything unused.

## Messaging

| Function | What it does |
|----------|----------------|
| `PostError` | Error; typically blocks a successful compliance run |
| `PostWarning` | Warning |
| `PostMessageToLog` | Log only |
| `AppendMessage` | Append to a message property |
| `error` / `#E` | Older expression-error helper (3 args) |
| `MessageBox` | UI dialog; avoid in CLI-oriented rules |

## Domain helpers used in current rules

Not every report-object factory is described at the same depth. These are the ones authors hit in envelope, HVAC, CSE, and results work:

| Function | Role |
|----------|------|
| `PolyLoopArea` / `InitializePolyLoop` / `ScalePolyLoop` / `CreatePolyLoopChild` | Geometry polyloops |
| `ConsAssmUFactor` / `ConsUFactorRes` | Assembly U-factor |
| `DaylightableArea` | Daylit area |
| `HourlyResultSum` / `ApplyHourlyResultMultipliers` / `_NEM` / `_Neg` / `CopyHourlyResults` | Hourly results and TDV/LSC multipliers |
| `Psych_HAProps` / `Psych_HAPropsValid` | Psychrometrics |
| `WriteToSimInput` / `AddCSEReportCol` | CSE input / reports |
| `RetrieveCSVValue` / `EvalRulelistOnCSVColumns` | Drive lists from CSV columns (batch-style) |
| `OpenExportFile` / `WriteToExportFile` / `CloseExportFile` / `ExportFileConcat` / `WriteToFile` | Text export |
| `CreateSCSysRptObjects` / `CreateDHWRptObjects` / `CreateIAQRptObjects` / `CreateDwellUnit*` | Report-object factories (call sites already exist; do not invent new factories) |
| `FileExists` / `SplitPath` | Path helpers |
| `YrMoDaToSerial` / `SerialDateTo*` / `Date` / `CurrentTime` / `CurrentYear` | Dates; `YrMoDaToSerial(-1,-1,-1)` is “today” in `Rules_BatchRuns.rule` |

## Highlighter gaps

`Utils/vsce/cbecc-rule-formatter/syntaxes/rule.tmLanguage.json` omits `OrderAssignments` and `SetObjectFlagBySum`. They are valid in the engine.

---

## Appendix: every registered long name

Arity `VAR_ARGS` is a range, not “any number.” Commented-out historical entries (`RefIndex`, `SymIndex`) are omitted. `StoreBDBase` / `BDBaseDBID` are old names for `StoreBEMProc` / `BEMProcDBID`.

| Long name | Alias | Arity |
|-----------|-------|-------|
| `log` | — | 1 |
| `exp` | — | 1 |
| `pow` | — | 2 |
| `abs` | — | 1 |
| `strupper` | — | 1 |
| `strlower` | — | 1 |
| `min` | — | 2 |
| `max` | — | 2 |
| `int` | — | 1 |
| `log10` | — | 1 |
| `sin` | — | 1 |
| `asin` | — | 1 |
| `cos` | — | 1 |
| `acos` | — | 1 |
| `tan` | — | 1 |
| `atan` | — | 1 |
| `sqrt` | — | 1 |
| `ftoa` | — | 1 |
| `mod` | — | 2 |
| `strlen` | — | 1 |
| `find` | — | 2 |
| `findnocase` | — | 2 |
| `atof` | — | 1 |
| `round` | — | 2 |
| `substr` | — | 3 |
| `error` | `#E` | 3 |
| `SymValue` | `#SV` | 1 |
| `EnumValue` | `#SV` | 1 |
| `SumAll` | `#SA` | 1 |
| `ChildCount` | `#CC` | 1 |
| `RuleLibrary` | `#RL` | VAR_ARGS |
| `ChildRef` | `#CR` | 2 |
| `Parent` | `#P` | VAR_ARGS |
| `Parent2` | `#P2` | VAR_ARGS |
| `Parent3` | `#P3` | VAR_ARGS |
| `Local` | `#L` | VAR_ARGS |
| `Global` | `#G` | VAR_ARGS |
| `SumChildren` | `#SC` | VAR_ARGS |
| `LocalRef` | `#LR` | VAR_ARGS |
| `ParentRef` | `#PR` | VAR_ARGS |
| `Parent2Ref` | `#PR2` | VAR_ARGS |
| `Parent3Ref` | `#PR3` | VAR_ARGS |
| `SumRevRef` | `#SR` | VAR_ARGS |
| `MaxChild` | `#MC` | VAR_ARGS |
| `MaxAll` | `#MA` | VAR_ARGS |
| `MaxRevRef` | `#MR` | VAR_ARGS |
| `CurrentTime` | `#CT` | 0 |
| `Date` | `#D` | 3 |
| `EnsureSymbolExists` | `#ESE` | 0 |
| `EvalRulelist` | `#ER` | VAR_ARGS |
| `CreateChildren` | `#CCH` | VAR_ARGS |
| `DeleteChildren` | `#DCH` | VAR_ARGS |
| `CreateComp` | `#CCO` | VAR_ARGS |
| `DeleteComp` | `#DCO` | VAR_ARGS |
| `DeleteAllComps` | `#DAC` | VAR_ARGS |
| `StoreBDBase` | — | 2 |
| `StoreBEMProc` | `#SB` | 2 |
| `LocalCompAssigned` | `#LCA` | VAR_ARGS |
| `ParentCompAssigned` | `#PCA` | VAR_ARGS |
| `LocalIsDefault` | `#LID` | VAR_ARGS |
| `ParentIsDefault` | `#PID` | VAR_ARGS |
| `CurrentYear` | `#CY` | 0 |
| `MinChild` | `#MIC` | VAR_ARGS |
| `MinAll` | `#MIA` | VAR_ARGS |
| `MinRevRef` | `#MIR` | VAR_ARGS |
| `FltToStr` | `#F2S` | VAR_ARGS |
| `LocalSymbolString` | `#LSS` | VAR_ARGS |
| `ComponentIndex` | `#COI` | VAR_ARGS |
| `ChildIndex` | `#CHI` | VAR_ARGS |
| `Format` | `#FMT` | VAR_ARGS |
| `LocalStatus` | `#LST` | VAR_ARGS |
| `ParentStatus` | `#PST` | VAR_ARGS |
| `PostError` | `#PE` | VAR_ARGS |
| `PostWarning` | `#PW` | VAR_ARGS |
| `CompExists` | `#CE` | 2 |
| `GlobalSymbolString` | `#GSS` | VAR_ARGS |
| `GlobalStatus` | `#GST` | VAR_ARGS |
| `CountRefs` | `#CRS` | VAR_ARGS |
| `CompName` | `#CN` | 2 |
| `CountUniqueParentRefs` | `#CUPRS` | VAR_ARGS |
| `CountNoRefs` | `#CNRS` | VAR_ARGS |
| `MaxRevRefComp` | `#MRC` | VAR_ARGS |
| `MaxAllComp` | `#MAC` | VAR_ARGS |
| `ComponentCount` | `#CMPC` | 1 |
| `FirstBitwiseMatchComp` | `#FBMC` | VAR_ARGS |
| `BitwiseMatchCount` | `#BMC` | VAR_ARGS |
| `UniqueComponentName` | `#UCN` | VAR_ARGS |
| `EnsureStringUniqueness` | `#ESU` | VAR_ARGS |
| `FileExists` | `#FE` | 1 |
| `ImportComponentFromFile` | `#ICFF` | VAR_ARGS |
| `EnsureChildAssigned` | `#ECA` | 0 |
| `SplitPath` | `#SP` | 2 |
| `MaxRevRefArray` | `#MRA` | VAR_ARGS |
| `CountOccurrences` | `#CO` | 2 |
| `SumIntoArrayElement` | `#SIA` | 3 |
| `SumRevRefEx` | `#SRE` | VAR_ARGS |
| `LocalSymbolInvalid` | `#LSI` | VAR_ARGS |
| `MessageBox` | `#MBX` | VAR_ARGS |
| `GlobalRef` | `#GR` | VAR_ARGS |
| `BDBaseDBID` | — | 3 |
| `BEMProcDBID` | `#DBID` | 3 |
| `PostMessageToLog` | `#PM2L` | VAR_ARGS |
| `FindInString` | `#FIS` | 2 |
| `ReplaceInString` | `#RIS` | 2 |
| `LocalMaxStringLength` | `#LMSL` | VAR_ARGS |
| `ParentStringArrayElement` | `#PSAE` | 2 |
| `ParentComponentType` | `#PCT` | 0 |
| `LocalArrayIndex` | `#LAI` | 4 |
| `ComponentArray` | `#CA` | VAR_ARGS |
| `GlobalCompAssigned` | `#GCA` | VAR_ARGS |
| `HourlyResultSum` | `#HRS` | VAR_ARGS |
| `ApplyHourlyResultMultipliers` | `#HRM` | VAR_ARGS |
| `ComponentType` | `#CTP` | 1 |
| `SumAcrossIf` | `#SAI` | VAR_ARGS |
| `SumChildrenIf` | `#SCI` | VAR_ARGS |
| `PolyLoopArea` | `#PLA` | 0 |
| `ScalePolyLoop` | `#SPL` | VAR_ARGS |
| `WriteToFile` | `#W2F` | VAR_ARGS |
| `ConsAssmUFactor` | `#CAUF` | 1 |
| `DaylightableArea` | `#DA` | 1 |
| `LogDuration` | `#LD` | VAR_ARGS |
| `InitializePolyLoop` | `#IPL` | 0 |
| `GlobalValid` | `#GV` | VAR_ARGS |
| `LocalValid` | `#LV` | VAR_ARGS |
| `ParentValid` | `#PV` | VAR_ARGS |
| `Parent2Valid` | `#PV2` | VAR_ARGS |
| `Parent3Valid` | `#PV3` | VAR_ARGS |
| `IfValidAnd` | `#IVA` | VAR_ARGS |
| `CreatePolyLoopChild` | `#CPL` | VAR_ARGS |
| `ConsUFactorRes` | `#CUFR` | 1 |
| `CreateSCSysRptObjects` | `#CSCSO` | 0 |
| `AssignOrCreateComp` | `#ACC` | VAR_ARGS |
| `CreateDHWRptObjects` | `#CDO` | 0 |
| `CreateIAQRptObjects` | `#CIO` | 0 |
| `ValidOr` | `#VO` | VAR_ARGS |
| `LocalRefSymbolString` | `#LRSS` | VAR_ARGS |
| `ParentSymbolString` | `#PSS` | VAR_ARGS |
| `ParentRefSymbolString` | `#PRSS` | VAR_ARGS |
| `Parent2SymbolString` | `#P2SS` | VAR_ARGS |
| `Parent2RefSymbolString` | `#PR2SS` | VAR_ARGS |
| `Parent3SymbolString` | `#P3SS` | VAR_ARGS |
| `Parent3RefSymbolString` | `#PR3SS` | VAR_ARGS |
| `EnumString` | `#ES` | 1 |
| `CreateDwellUnitHVACSysObjects` | `#CDHO` | 0 |
| `MaxChildComp` | `#MCC` | VAR_ARGS |
| `MinChildComp` | `#MICC` | VAR_ARGS |
| `YrMoDaToSerial` | `#YMD2S` | 3 |
| `SerialDateToDayOfMonth` | `#D2D` | 1 |
| `SerialDateToMonth` | `#D2M` | 1 |
| `SerialDateToYear` | `#D2Y` | 1 |
| `YrMoDaToDayOfWeek` | `#YMD2DW` | 3 |
| `CreateDwellUnitRptObjects` | `#CDURO` | 0 |
| `ListRevRef` | `#LRR` | VAR_ARGS |
| `ListRevRefIf` | `#LRRI` | VAR_ARGS |
| `OpenExportFile` | `#OXF` | VAR_ARGS |
| `WriteToExportFile` | `#WXF` | VAR_ARGS |
| `CloseExportFile` | `#CXF` | 1 |
| `StrToFlt` | `#S2F` | 1 |
| `CreateDwellUnitDHWHeaters` | `#CDUWH` | 0 |
| `AddCSEReportCol` | `#ACRC` | 2 |
| `ApplyHourlyResultMultipliers_NEM` | `#HRMN` | VAR_ARGS |
| `ApplyHourlyResultMultipliers_Neg` | `#HRMNg` | VAR_ARGS |
| `CopyHourlyResults` | `#CHR` | 6 |
| `GlobalRefSymbolString` | `#GRSS` | VAR_ARGS |
| `ScheduleSum` | `#SS` | VAR_ARGS |
| `UniqueListRevRef` | `#ULRR` | VAR_ARGS |
| `UniqueListRevRefIf` | `#ULRRI` | VAR_ARGS |
| `Parent2ComponentType` | `#P2CT` | 0 |
| `Parent3ComponentType` | `#P3CT` | 0 |
| `SchDayHoursString` | `#SDHS` | VAR_ARGS |
| `WriteToSimInput` | `#WSI` | VAR_ARGS |
| `RetrieveCSVValue` | `#RCSVV` | 4 |
| `EvalRulelistOnCSVColumns` | `#ERCC` | VAR_ARGS |
| `AppendMessage` | `#AM` | VAR_ARGS |
| `GlobalSymbolValue` | `#GSV` | VAR_ARGS |
| `GlobalRefSymbolValue` | `#GRSV` | VAR_ARGS |
| `LocalSymbolValue` | `#LSV` | VAR_ARGS |
| `LocalRefSymbolValue` | `#LRSV` | VAR_ARGS |
| `ParentSymbolValue` | `#PSV` | VAR_ARGS |
| `ParentRefSymbolValue` | `#PRSV` | VAR_ARGS |
| `Parent2SymbolValue` | `#P2SV` | VAR_ARGS |
| `Parent2RefSymbolValue` | `#PR2SV` | VAR_ARGS |
| `Parent3SymbolValue` | `#P3SV` | VAR_ARGS |
| `Parent3RefSymbolValue` | `#PR3SV` | VAR_ARGS |
| `FormatNL` | `#FMTNL` | VAR_ARGS |
| `Psych_HAPropsValid` | `#PSHAPV` | VAR_ARGS |
| `Psych_HAProps` | `#PSHAP` | VAR_ARGS |
| `NumUniqueChildVals` | `#NUCV` | VAR_ARGS |
| `CreateCompFor` | `#CCF` | VAR_ARGS |
| `ExportFileConcat` | `#XFC` | VAR_ARGS |
| `SetNextArrayElement` | `#SNAE` | 1 |
| `MaxAcrossIf` | `#MXAI` | VAR_ARGS |
| `MinAcrossIf` | `#MNAI` | VAR_ARGS |
| `CopyComp` | `#CPC` | VAR_ARGS |
| `OrderAssignments` | `#OA` | 2 |
| `SetObjectFlagBySum` | `#SOFS` | VAR_ARGS |
