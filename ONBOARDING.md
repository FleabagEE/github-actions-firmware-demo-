# Onboarding Guide

## What This Repo Does
This repository contains a GitHub Actions workflow that automates issue handling for firmware work.

The workflow:

- filters for firmware issues
- adds them to a GitHub Project v2 board
- initializes fields such as `Status` and `Priority`
- keeps `Status` synchronized with issue lifecycle events
- supports opt-out with `automation:skip`

---

## First Things to Check

Before testing anything, confirm:

1. the workflow file exists under `.github/workflows/`
2. the repository has the required labels
3. the target GitHub Project v2 exists
4. the `PROJECTS_TOKEN` secret is configured
5. project field IDs and option IDs match the current project configuration

---

## Required Labels

Make sure these labels exist:

- `firmware`
- `bug`
- `enhancement`
- `in-progress`
- `automation:skip`

---

## Required Project Fields

The Project v2 board should include:

### Status
- `Todo`
- `In Progress`
- `Done`

### Priority
- `High`
- `Medium`
- `Low`

---

## Fast Smoke Test

### 1. Open a firmware issue
Create a new issue with:

- `firmware`

Expected:

- workflow runs
- issue is added to Project v2
- `Status = Todo`

### 2. Test priority mapping
Create or update an issue with:

- `firmware`
- `bug`

Expected:

- `Priority = High`

### 3. Test status sync
Add:

- `in-progress`

Expected:

- `Status = In Progress`

Then close the issue.

Expected:

- `Status = Done`

### 4. Test opt-out
Add:

- `automation:skip`

Expected:

- workflow logs a manual override skip
- project automation step does not run

---

## How to Debug

Go to:

`Actions -> select the latest run -> inspect-issue`

The most useful steps to inspect are:

- `Print issue context`
- `Check firmware label`
- `Add firmware issue to Project v2 and populate fields`
- `Skip automation for opt-out issues`

Important log lines usually include:

- event action
- parsed labels
- target status
- existing item lookup result
- whether add/update/skip happened

---

## Common Failure Patterns

### Project not found
Likely causes:

- wrong project owner
- wrong project number
- token cannot read the project

### Existing item not found
Likely causes:

- issue is not yet in the project
- lookup logic is incomplete
- queried item range does not include the target item

### Field update fails
Likely causes:

- field ID is wrong
- option ID is wrong
- project configuration changed

### Workflow skips unexpectedly
Likely causes:

- missing `firmware` label
- `automation:skip` label is present

---

## Maintenance Notes

- If project field configuration changes, update the hardcoded IDs in the workflow.
- If more automation rules are added, keep skip logic and status sync logic clearly separated.
- Prefer explicit logs for skip decisions so future debugging stays easy.

---

## Current Scope

This onboarding guide assumes the first working version of the workflow.

It does not yet cover:

- PR merged -> issue done sync
- dynamic field discovery at runtime
- production-grade retry or notification behavior

That is acceptable for a demo-focused automation project.
