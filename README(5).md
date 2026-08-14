# GUTS WP2 Behavioral Data Cleaning

This repository documents the behavioral data-cleaning workflow used for GUTS Work Package 2 (WP2) across Waves T0–T3.

The repository is intended to make the cleaning logic transparent and reproducible while keeping participant data and direct identifiers outside version control. It contains the wave-specific R Markdown cleaning scripts together with documentation describing the overall process.

## Repository structure

```text
.
├── .gitignore
├── LICENSE
├── README.md
├── config.private.R.example
├── cleaning/
│   ├── T0/
│   │   └── wave0_cleaning.Rmd
│   ├── T1/
│   │   └── wave1_cleaning.Rmd
│   ├── T2/
│   │   └── wave2_cleaning.Rmd
│   └── T3/
│       └── wave3_cleaning.Rmd
└── docs/
    └── Behavioral_data_cleaning.pptx
```

## What the pipeline does

Although the exact survey structure differs across waves, the scripts follow the same general cleaning strategy:

1. Import the wave-specific raw survey components.
2. Standardize identifiers, names, timestamps, and other linkage fields.
3. Identify likely test or researcher-generated responses.
4. Resolve missing or shared participant identifiers where possible.
5. Resolve repeated informed-consent submissions and other identity collisions.
6. Review incomplete or ambiguous records through explicit audit files.
7. Link cleaned participant records to the stable project-level `gutsID`.
8. Perform additional quality-control checks before export.
9. Split cleaned questionnaires into measure-level datasets using the project measure registry/overview.
10. Export cleaned, consistently named measure-level outputs for downstream analysis.

For Waves T1–T3, the pipeline also processes sociometric nomination data. Participant names embedded in sociometric metadata are mapped to `gutsID`, identifying labels are removed, and the nominations are converted into network-analysis formats such as directed edgelists and participant-level nomination rosters.

Wave T0 does not contain the sociometric processing component.

## Audit-based cleaning

A central feature of the pipeline is that ambiguous data-quality problems are **not resolved silently by automated rules**.

Qualtrics-based longitudinal data can contain cases such as:

* researcher or test responses;
* survey restarts;
* missing participant identifiers;
* shared survey links or identifiers;
* multiple informed-consent submissions;
* incomplete submissions;
* ambiguous identity or linkage matches; and
* unmatched sociometric nominations.

These cases are handled through a staged:

**write → edit → read → apply**

workflow.

When an editable audit file is required, the script writes the review file to secure project storage and stops. A reviewer then edits only the designated decision fields. When the relevant section is run again, the existing audit file is preserved, checked, and the reviewed decisions are applied.

This design keeps subjective decisions explicit and traceable while allowing the remainder of the pipeline to remain deterministic.

The scripts therefore are **not intended to be knitted blindly from beginning to end on a new dataset**. Manual audit stages should be completed when prompted.

## Data privacy

**No participant-level data are included in this repository.**

The following files should remain in approved secure project storage and must not be committed to the public repository:

* raw SPSS/Qualtrics exports;
* participant names or other direct identifiers;
* the private name-to-`gutsID` linkage workbook;
* editable audit and review CSVs;
* identity ledgers containing names or Qualtrics `ResponseId` values;
* participant-level receipts;
* private participant-specific correction files;
* processed participant-level datasets; and
* rendered HTML generated from the real data.

Some cleaning steps necessarily use identifying information during identity resolution. The public scripts are structured so that participant- or researcher-specific corrections can be supplied locally without embedding those values in the repository.

Always inspect `git status` before committing files.

## Local configuration

The scripts expect the private project data to exist outside the repository.

### Data root

Set the `GUTS_DATA_ROOT` environment variable to the secure project data directory.

For example, in a local `.Renviron` file:

```text
GUTS_DATA_ROOT=X:/secure/project/data
```

The scripts build their wave-specific input and output paths relative to this directory.

### Private corrections

Some waves require researcher/test names or participant-specific corrections to reproduce historical cleaning decisions. These values must remain outside the public repository.

Set `GUTS_PRIVATE_CONFIG` to a private R configuration file:

```text
GUTS_PRIVATE_CONFIG=X:/secure/private/config.private.R
```

Depending on the wave, the private configuration may define:

```r
researcher_names <- character(0)

linkage_name_patches <- character(0)

sociometric_name_patches <- character(0)

sociometric_name_regex_patches <- character(0)
```

Real names or identifying corrections should never be entered into the public example configuration or committed to GitHub.

## Wave-specific notes

### T0

T0 uses the common identity-cleaning and linkage framework but has a different survey structure from the later waves. Researcher/test-name flags and participant-specific linkage corrections, when required, are supplied through the private configuration.

T0 does **not** include the sociometric nomination pipeline.

### T1

T1 applies the common audit and linkage framework across its survey components and includes sociometric nomination cleaning. Sociometric nominations are anonymized to `gutsID` and prepared for social-network analysis.

### T2

T2 follows the same general framework as T1. Participant-specific linkage corrections and any exceptional sociometric parsing corrections are supplied through the private configuration rather than being embedded in the public script.

### T3

T3 continues the common audit, linkage, measure-export, and sociometric workflows. Participant-specific linkage and sociometric parsing corrections are likewise kept outside the repository.

## R dependencies

Across the four wave scripts, the pipeline uses the following R packages:

* `haven`
* `dplyr`
* `stringr`
* `stringi`
* `tibble`
* `readxl`
* `readr`
* `tidyr`
* `purrr`
* `janitor`
* `labelled`
* `sjmisc`
* `stringdist`
* `knitr`

Not every package is used by every wave.

Package versions are not currently locked in this repository. An `renv` lockfile could be added in the future if exact long-term package reproduction is required.

## Outputs

The cleaning scripts generate outputs in secure project storage rather than inside the Git repository.

Depending on the wave, these include:

* cleaned participant-level survey components linked to `gutsID`;
* audit reports and decision receipts;
* measure-level behavioral datasets;
* measure metadata;
* quality-control summaries; and
* for T1–T3, sociometric edgelists and nomination rosters.

The repository documents how these outputs are created; it does not distribute the participant-level outputs themselves.

## Process documentation

The `docs/` directory contains `Behavioral_data_cleaning.pptx`, a presentation originally prepared for the research team that provides a higher-level overview of the behavioral cleaning process and the rationale behind the audit-based workflow.

For implementation details, the wave-specific R Markdown scripts are the authoritative source.
