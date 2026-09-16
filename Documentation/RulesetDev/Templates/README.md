# Rule file templates

Use these as starting points. Delete instructional `;` comments before committing production rules. Keep `//` license and section headers.

## DataModel (NRMF)

| File | Use |
|------|-----|
| [BlankRule.rule](BlankRule.rule) | `RULE Object:Property` for a property that already exists in BEMBase |
| [BlankRule_Definitions.rule](BlankRule_Definitions.rule) | Same, with INPUTCLASS / RESETS / CHECKSIM notes |
| [BlankRuleNew.rule](BlankRuleNew.rule) | `RULE NEW` for a property that does not yet exist in BEMBase |
| [BlankRuleNew_Definitions.rule](BlankRuleNew_Definitions.rule) | Same, with DATATYPE / LONGFORM notes |

Naming convention (from the definition files): `<ACMSectionCamelCase>-<DescriptionCamelCase>.rule`, for example `HVACSecondary-CoolingCoil-General.rule`. Soft line length 80 characters.

## Procedural (SFam / shared RULELIST files)

| File | Use |
|------|-----|
| [BlankRuleList.rule](BlankRuleList.rule) | `RULELIST` skeleton with the four 0/1 flags |

## Historical (do not use for new work)

`2013ACMRule.jgcscs` and `2013ACMRule.jgfns` are 2013 ACM editor artifacts.

## Syntax example

Table lookup snippet: [../Syntax Examples/Table Lookup Syntax.rule](../Syntax%20Examples/Table%20Lookup%20Syntax.rule)
