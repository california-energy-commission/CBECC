# Release Compliance Models

Dedicated models for water-marking testing during release validation. See [#932](https://github.com/NOR-Codes-Stds/CBECC-Dev/issues/932).

## Purpose

SACriswell and the CEC team requested a dedicated folder of models used to verify that compliant projects produce reports **without** water marking at release time. Models in this directory do **not** need to have zero compliance margin; they only need to demonstrate positive compliance so report output is unmarked.

## Pre-Release Workflow

Before a release, run this set of models and confirm each produces positive compliance results with no water marking on generated reports. This step is required on all release checklists (major, minor, and patch); see `scripts/release-issue-templates/release-checklist-data.yaml`.

## Models

Models here are **severed copies** of counterparts elsewhere in `Projects/`. They are maintained independently so release testing does not depend on ongoing changes in development or standard model directories.
