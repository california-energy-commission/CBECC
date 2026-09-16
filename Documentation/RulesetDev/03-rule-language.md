# Rule language

Shared expression grammar used inside DataModel `RULE` blocks **and** procedural `RULELIST` assignments. The two file layouts differ ([04-datamodel-vs-procedural.md](04-datamodel-vs-procedural.md)); the expressions inside them do not.

Functions (`SumChildren`, `IfValidAnd`, …) are listed in [06-rule-functions.md](06-rule-functions.md). This page is operators, control flow, property paths, tables, reserved values, and comments.

The 2011 FRS §5 is historical. This page is rewritten against current `RulesetSrc` and engine `src/BEMProc/expFormula.c`.

## Comments

| Marker | Use |
|--------|-----|
| `//` | Permanent comments (license, section headers, `TO DO:`, `ISSUE:`). Keep these in committed files. |
| `;` | Instructional notes in templates, or end-of-line comments. Delete `;` guideline comments from production copies of the blank templates. |

Inside a DataModel `RULE` body, a `;` also starts a comment to end of line.

## Literals

- Numbers: `0`, `1.5`, `-3`
- Strings: `"quoted"` (escape with `\` if needed)
- Booleans in expressions are numeric: `0` false, non-zero true
- Enumeration comparisons use the symbol **name** as a string, or `EnumValue` / `SymValue` for the integer

## Operators

Arithmetic: `+` `-` `*` `/` `^` (power). Use `pow(a,b)` or `mod(a,b)` as functions when that is clearer.

Comparison: `==` `!=` (also `<>`) `<` `>` `<=` `>=`

Logical (dot form is what current rules use):

```
.AND.   .OR.   .NOT.
```

Group with parentheses. `min` / `max` take **two** arguments; nest them for more.

String concatenate with `+` when both sides are strings (`DefaultProjPath + BatchDefsCSV`).

## Control flow

Keywords are recognized in several capitalizations (`if` / `IF` / `If`). Prefer lowercase `if` / `then` / `else` / `endif` to match current NRMF files.

```
if ( condition )
then  expression-if-true
else  expression-if-false
endif
```

Chain with `else if`:

```
if (LocalStatus( BatchDefsCSV ) < 1) then  UNDEFINED
else if (strlen( BatchDefsCSV ) < 1) then  UNDEFINED
else  BatchDefsCSV
endif endif
```

Each `if` needs a matching `endif`. Nested `if`s therefore end with multiple `endif`s.

`switch` / `case` / `default` / `endswitch` exist in the engine (`expFormula.c` keyword table). Most Title-24 rules use `if` / `then` instead.

## Property paths

A bare name (`VentFlow`) is a property on the **current object** (the object the `RULE` or assignment is evaluating).

| Form | Meaning |
|------|---------|
| `Local( Prop )` | Same as the bare name; optional extra args for array index |
| `Parent( Prop )` | Parent object |
| `Parent2( Prop )` / `Parent3( Prop )` | Grandparent / great-grandparent |
| `Global( Class:Prop )` | Named class (often `Proj:`), not necessarily an ancestor |
| `LocalRef( RefProp, Prop )` | Follow an object-reference property, then read `Prop` |
| `ChildRef( Class, index )` | Child of the given class by 1-based index |
| `u:Spc:Area` | Property on another object **in a named transform** (DataModel) |

Transform prefixes come from the `TRANSFORMATIONS` block in `T24N_YYYY.txt`:

| Prefix | Transform |
|--------|-----------|
| `u:` | USER (user / proposed input copy) |
| `zp:` / `z:` | SIZING_PROPOSED |
| `zb:` | SIZING_BASELINE |
| `ap:` / `a:` | ANNUAL_PROPOSED |
| `ab:` | ANNUAL_BASELINE |

Do not invent prefixes. SFam has no `TRANSFORMATIONS` block; proposed vs standard is done with **named rulelists**, not `u:`/`ab:` paths.

Array elements: `Prop[1]` (1-based in most rules) or pass the index as a function argument (`Local( Prop, 1 )`).

## Reserved assignment values

These are **targets**, not numbers:

| Token | Effect |
|-------|--------|
| `UNDEFINED` | Clear the property (no valid input or default) |
| `UNCHANGED` | Leave whatever is already stored |
| `NONE` | For object-reference properties: assign no object |

Typical pattern: only rewrite a field when a condition holds, otherwise `UNCHANGED`.

## Table lookup

Compiled tables are named in the year manifest (`TABLEFILES` / `TABLELIST`) or in a `TABLE` … `ENDTABLE` block inside a `.rule` file.

DataModel lookup (column name, then alternating search-column name and value):

```
BaseVerticalFenPerformance:UFactor("ProductType", bz:FenProdType, "OccupancyClass", bz:FenestrationOccupancy)
```

SFam `TABLELIST` tables are called by table name and column similarly. See [Syntax Examples/Table Lookup Syntax.rule](Syntax%20Examples/Table%20Lookup%20Syntax.rule).

If a cell holds a warning token rather than a number, the lookup can return that string; handle it in the expression.

## DataModel expressions (no braces)

Inside `DEFAULT`, `PROPOSED`, `BASELINE`, `CHECKSIM`, … the expression is **bare**:

```
DEFAULT
  if( IfValidAnd( CommKitArea > 0 ) )
  then  ValidOr( VentFlow, 0 )
  else  0
  endif
```

## Procedural expressions (braces)

Each assignment is `Class:Prop = { expression }`:

```
"Set BatchRuns:HaveBatchDefsCSV"    BatchRuns:HaveBatchDefsCSV  = {
     if (LocalStatus( FullBatchDefsCSV ) < 1) then  0
     else if (FileExists( FullBatchDefsCSV )) then  1
     else  0  endif endif  }
```

A constant still uses braces by convention: `Proj:EnergyCodeYearNum = { 2025 }`.

## Error posting in expressions

`PostError( "message" )` and `PostWarning( "message" )` write to the analysis log and can fail compliance. `error(...)` / `#E` is the older expression-error helper. Prefer `PostError` / `PostWarning` in new rules. `MessageBox` is UI-oriented; do not rely on it in CLI analysis.

## Where this is parsed

Engine: `src/BEMProc/expFormula.c` (tokenizer, `functable[]`, evaluation) and `src/BEMProc/expRuleFile.cpp` (file / `RULE` / `RULELIST` layout). Authors do not edit those files in this repository.
