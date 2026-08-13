# Projects Directory
This directory is a collection of various CBECC models used for testing, development, and reference.

## Year Directory Components
### `research`
Not mirrored to public repo. This directory is used to store models used for memos or other research tasks. Create new sub-folders as needed for each new group of research items so related models stay grouped and easy to find.

### Common for `non-residential` & `multi-family` & `single-family`
- `EAA` (Existing Additions and/or Alterations)
- `examples`
- `prototypes`
- `standard`
- `user-support`

### Common for `non-residential` & `multi-family`
- `ruleset-implementation`
- `sensitivity`

### Common for `multi-family` & `single-family`
- `cuac`

### Common for `multi-family`
- `schema-related`
- `contains-nonres-elements.json` - This json file is used to track the files that contain non-residential elements, such that non-res developers can easily identify and test these files in addition to the non-residential models. Any of these files when running should also assume these models could also be under the sub-directory `~not-for-release`. 

### Common for `single-family`
- `TDS`

### `~not-for-release` Directories
This sub-directory should exist in directories that are typically packaged within a release. Please reference the `Projects/release-package.json` for detail. **TODO:** Create and note the workflow noted in #772

**NOTE:** the `.gitkeep` files in these directories are used to ensure the directories are tracked by Git even if they are empty. Do not remove these `.gitkeep` files.

### `~intentional-failures` Directories
This sub-directory should exist in directories that are used to test the system's ability to handle failures. Currently it only exists as needed. Files in these directories are also not-for-release.

**Important:** Assure to include a README.md file in each of these directories to explain how/why these files are intended to fail.

### `release-compliance` Directories
Per-year directory of severed model copies used to verify positive compliance and **no water marking** on generated reports during release testing. Models do not need zero compliance margin. See `Projects/<year>/release-compliance/README.md` in each year that has this folder ([#932](https://github.com/NOR-Codes-Stds/CBECC-Dev/issues/932)).

### `standard` Directories
These models are designed for ~0% compliance margin where Proposed model = Baseline model for each end-use. Useful for ruleset checking. 

### `user-support` Directories
Not mirrored to public repo. These directories in each of the core directories are used to store and track models submitted from user-support issues which contain valuable reach-cases that the current testing suite otherwise does not test. 

**TODO: A workflow/wiki page should be created to track and manage these files.**

### `README.md` Files
Add README files as needed to any directory if there is specific information to know about models in that directory.

# Other Key Files for Automated Workflows
## `release-package.json`
This file is used to track the files that are typically packaged within a release. Please reference the `Projects/release-package.json` for detail. **TODO:** Create and note the workflow noted in #772

## `pr-checks.json`
This file defines the sample models exercised by the **Test Small Sample Models** PR check (`.github/workflows/test_select_models.yml`).

- **When it runs:** The workflow runs only when the PR modifies files under `RulesetSrc/`, or when triggered manually (`workflow_dispatch`).
- **What it runs:** `prepare_and_run.py` reads each building-type section's `all` array (for example `nonres_multifam.all`) and runs those models through CBECC.
- **Adding models:** Add paths under the appropriate building type's `all` list. Paths may be directories (all IBD files copied) or individual `.cibd##` / `.ribd##` files.
- **`by_component`:** Reserved for future ruleset-component–specific model selection; not used by CI yet.

## `test-sets.json`
This file defines named groups of model files that developers can copy into
explicit destination layouts. A test set identifies the models used by a
workflow; it does not define the workflow's analysis or pass/fail criteria.

Run the test-set tool from the repository root:

```powershell
# Discover available sets
python TestingCBECC/workflows/test_sets/prepare_test_set.py list

# Inspect a copy plan without writing files
python TestingCBECC/workflows/test_sets/prepare_test_set.py preview sensitivity C:\CBECC-Tests\Sensitivity

# Prepare a set
python TestingCBECC/workflows/test_sets/prepare_test_set.py prepare pre-release-nr-mf C:\CBECC-Tests\PreRelease-NR-MF

# Explicitly replace an existing destination
python TestingCBECC/workflows/test_sets/prepare_test_set.py prepare pre-release-nr-mf C:\CBECC-Tests\PreRelease-NR-MF --clean

# Prepare and run (prompts for destination; confirms overwrite and model count)
python TestingCBECC/workflows/test_sets/prepare_test_set.py run sensitivity
```

Each set has a description, a default `content` selection, and one or more
source-to-destination `mappings`. An optional `options_csv` string is passed
through to CBECC when the set is run (omit the field to use CBECC defaults).
Source paths may use `{year}`; prepare/preview can select another year with
`--year` (run workflows use `DEFAULT_YEAR` from
`TestingCBECC/workflows/test_sets/_cli.py`). Only `.cibd##` and
`.ribd##` model files are copied, and their paths beneath each mapped source
are preserved.

```json
{
  "version": 1,
  "sets": {
    "example-set": {
      "description": "Example regular and not-for-release model set.",
      "content": ["regular", "not-for-release"],
      "mappings": [
        {
          "source": "Projects/{year}/non-residential/examples",
          "destination": "OtherTests"
        }
      ]
    }
  }
}
```

The available content categories are:

- `regular`: models outside either special directory.
- `not-for-release`: models beneath `~not-for-release`.
- `intentional-failures`: models beneath `~intentional-failures`.

Set defaults can be overridden by repeating `--content`. For example:

```powershell
# Add not-for-release models to the pre-release NR/MF set
python TestingCBECC/workflows/test_sets/prepare_test_set.py prepare pre-release-nr-mf C:\CBECC-Tests\PreRelease-NR-MF-All --content regular --content not-for-release

# Preview only the not-for-release portion of NR/MF
python TestingCBECC/workflows/test_sets/prepare_test_set.py preview pre-release-nr-mf C:\CBECC-Tests\PreRelease-NR-MF-NFR --content not-for-release
```

The tool validates every source and destination before copying. It rejects
duplicate destination paths, refuses to write under `Projects/`, and refuses a
non-empty destination unless the user confirms replacement or supplies
`--clean`. Files in `Projects/` are never moved or modified.
