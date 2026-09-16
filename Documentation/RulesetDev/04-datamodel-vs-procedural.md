# DataModel vs procedural

Two authorship styles share the expression language ([03-rule-language.md](03-rule-language.md)) but **not** the file layout. Mixing them in one file will not compile.

| | DataModel | Procedural |
|--|-----------|------------|
| Used by | `T24NRMF` (nonres + multifamily) | `T24SFam` (single-family) and many `shared/` files |
| Manifest | `T24N_YYYY.txt` with `FORMAT DataModel` | `Rules-YYYY.txt` (`RULESET_STRUCT_VERSION`, `RULEFILE`, `TABLELIST`, …) |
| Property rules | `RULE` / `RULE NEW` … `ENDRULE` | Assignments inside `RULELIST` … `END` |
| Expression wrapper | Bare expression in a transform block | `{ expression }` after `Class:Prop =` |
| Proposed vs baseline | Transform tags (`DEFAULT`, `PROPOSED`, `BASELINE`, `SIZING_*`, `ANNUAL_*`) | Separate named lists (`ProposedInput`, `ProposedCompliance`, CSE / budget lists, …) |
| Input class / ranges | `INPUTCLASS`, `MINIMUM`/`MAXIMUM`, `OPTION` on the `RULE` | `Datatype.txt`, `Ranges.txt`, `Symbols.txt` |

NRMF **also** contains `RULELIST` files (`RULELISTFILES` in `T24N_YYYY.txt`) for named lists the engine or `EvalRulelist` calls (CSE prep, CUAC, CF1R). Those lists use procedural **assignment syntax** inside a DataModel **ruleset**. That is expected. What you must not do is put a `RULE`/`ENDRULE` block into a SFam `Rules-YYYY` procedural file, or curly-brace assignments into a DataModel `RULE` body.

## DataModel `RULE`

Skeleton and field meanings: [Templates/BlankRule_Definitions.rule](Templates/BlankRule_Definitions.rule) and [BlankRuleNew_Definitions.rule](Templates/BlankRuleNew_Definitions.rule).

```
RULE Spc:SomeProp
  DESCRIPTION
    "…"
  INPUTCLASS
    Optional
  DEFAULT
    0
  ANNUAL
    Local( SomeProp )
ENDRULE
```

`RULE NEW` adds a property that is not in BEMBase (`DATATYPE`, `LONGFORM` / `SHORTFORM`). Prefer BEMBase for shared schema; use `RULE NEW` for ruleset-only or reporting fields.

### Transform / analysis blocks

The compiler attaches each expression to an analysis phase. Names in current NRMF templates and rules:

| Block | When it runs |
|-------|----------------|
| `DEFAULT` | User-model defaulting (analysis step 1) |
| `CHECKSIM` | Simulation-oriented errors/warnings; does not change the model |
| `CHECKCODE` | Code-compliance errors/warnings; does not change the model |
| `SIZING` or `SIZING_PROPOSED` / `SIZING_BASELINE` | After copies to sizing transforms |
| `ANNUAL` or `ANNUAL_PROPOSED` / `ANNUAL_BASELINE` | After copies to annual transforms |
| `PROPOSED` / `BASELINE` | Older aliases still seen in templates |
| `NRCCPRF` | After results, before NRCC-PRF XML export |

`PROPOSEDSIZING` / `BASELINESIZING` in the blank template map to the sizing transforms. Use the names already in the file you are editing.

A block may be omitted. If omitted, that phase does not assign the property from this `RULE`.

### `INPUTCLASS`

Controls whether the user must enter the value and whether it appears in the Input Data Model dump. Values: `Compulsory`, `Required`, `CondRequired`, `Optional`, `Default`, `CriticalDefault` (not supported), `Prescribed`, `NotInput`. `NotInput` may be followed by `AllowUIReset` / `ErrorIfInput` / `IgnoreUserInput` and an optional quoted log message. Full notes are in the definitions template.

### Other `RULE` clauses

`RULESETS`, `HELP`, `REFERENCE`, `PREVIOUSNAMES`, `OPTION`, `MINIMUM`/`MAXIMUM`, `COMMONMINIMUM`/`COMMONMAXIMUM`, `UNITS`, `REPORTPRECISION`, `RESETS` — see the definitions template. `RESETS` apply to the **UI**, not to CLI analysis.

## Procedural `RULELIST`

Header from `Rules-2025.txt` and [Runset-Definition-Info](https://github.com/NOR-Codes-Stds/CBECC-Dev/wiki/Runset-Definition-Info):

```
RULELIST "ListName"  F1 F2 F3 F4
   "Rule id string"    Class:Property  = { expression }
END
```

The four flags are `0` or `1`:

| # | Meaning |
|---|---------|
| 1 | `1` = evaluate every rule regardless of datatype. `0` = skip if the current value is Library or User defined, unless datatype is Prescribed or NotInput |
| 2 | Iteration / re-eval flag. **Ignored in the residential ruleset.** |
| 3 | `1` = classify values written by this list as user-defined; `0` = normal ruleset-defined |
| 4 | `1` = bypass SetBDData resets; `0` = allow resets (`RULESET_STRUCT_VERSION` ≥ 5) |

The quoted rule id is required. The compiler prefixes it with list index, rule index, and source line in compile logs.

Blank template: [Templates/BlankRuleList.rule](Templates/BlankRuleList.rule).

### Engine-called SFam lists (examples)

These names are called from Compliance Manager, not only from other rules:

| List | Role |
|------|------|
| `ProposedInput` (and `ProposedInput_*` splinters) | Default the user/proposed model |
| `ProposedModelCodeCheck` / `ProposedModelSimulationCheck` | Checks |
| `ProposedModelCodeAdditions` / `ProposedCompliance` | Proposed-model prep |
| CSE / budget / results / CF1R lists | Simulation input, standard design, reporting |

NRMF DataModel properties use transform blocks for the same idea; NRMF **named** lists still exist for CSE, CUAC, batch, hashes, and reports (`RULELISTFILES` in `T24N_2025.txt`).

## `EvalRulelist`

From either style:

```
EvalRulelist( "Default_BatchRuns_Simplified" )
EvalRulelist( "Default_BatchRuns_Simplified", 1 )
```

The optional second argument is a flags/mode integer used by current rules as `1` when the caller wants the nested list to run unconditionally. Do not call a DataModel transform name (`ANNUAL_PROPOSED`) as if it were a `RULELIST`.

## Checklist when adding a file

1. Pick the tree: `T24NRMF` vs `T24SFam` vs `shared`.
2. Match the style of neighboring files in that tree.
3. Register the file in the year manifest (`RULEFILES` / `RULEFILE` / `RULELISTFILES`).
4. Do not paste `RULE`/`ENDRULE` into a `RULELIST` file or `{ }` assignments into a `RULE` body.
