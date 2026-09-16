# Ruleset authoring

This is the developer reference for writing Title-24 compliance rules in **CBECC-Dev**. The source of truth for rules is [`RulesetSrc/`](../../RulesetSrc/). Engine binaries compile and evaluate those rules; they are documented separately under [`../Engine/`](../Engine/README.md).

These pages are for ruleset authors. They are not ACM/Standards text and not the CBECC end-user manuals.

## DataModel vs procedural (read this first)

There are two authorship styles. **Do not mix them in one file.**

| Style | Used by | Manifest | Rule form |
|-------|---------|----------|-----------|
| **DataModel** | Nonresidential / multifamily (`T24NRMF`) | `T24N_YYYY.txt` with `FORMAT DataModel` | `RULE` / `RULE NEW` … `ENDRULE` blocks |
| **Procedural** | Single-family (`T24SFam`) and many `shared/` files | `Rules-YYYY.txt` | `RULELIST` … `END` with `Target = { expression }` |

NRMF files can also contain `RULELIST` blocks for named lists the engine calls (`EvalRulelist`, CSE prep, batch). That is still DataModel ruleset format; the procedural *manifest* style (`TABLELIST` / `DATATYPES` / `RULEFILE`) is SFam.

## Pages

1. [Ruleset structure](01-ruleset-structure.md) — `RulesetSrc` layout, year manifests, `shared`, compile vs deploy artifacts
2. [BEMBase and support files](02-bembase-and-support-files.md) — schema, enums, screens, SFam datatype/ranges, tables, libraries
3. [Rule language](03-rule-language.md) — expressions, operators, property paths, table lookup, reserved values
4. [DataModel vs procedural](04-datamodel-vs-procedural.md) — `RULE` blocks vs `RULELIST`, transforms, INPUTCLASS, rulelist flags
5. [Compile and runtime](05-compile-and-runtime.md) — `BEMCompiler` batches, CLI compile, when lists run, engine file map
6. [Expression functions](06-rule-functions.md) — what `SumChildren`, `IfValidAnd`, `EvalRulelist`, and the rest do

## Templates and examples

- [Templates](Templates/README.md) — blank DataModel `RULE` / `RULE NEW` and a procedural `RULELIST`
- [Syntax Examples/Table Lookup Syntax.rule](Syntax%20Examples/Table%20Lookup%20Syntax.rule)

## Related

- Engine / CLI: [`../Engine/`](../Engine/README.md)
- VS Code `.rule` highlighter: [`Utils/vsce`](../../Utils/vsce/README.md)
- Cursor agent rule for `.rule` files: [`.cursor/rules/cbecc-rule-files.mdc`](../../.cursor/rules/cbecc-rule-files.mdc)
- Historical FRS and training Word docs: [`../Archive/`](../Archive/README.md)
