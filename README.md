# GitHub Actions Firmware Project Automation Demo

## Project Purpose
This project is a GitHub Actions demo that automates issue intake and project tracking for firmware work.

The workflow is designed to:

- detect firmware-related issues
- add qualifying issues into a GitHub Project v2 board
- initialize project fields such as `Status` and `Priority`
- keep project status synchronized with later issue lifecycle events
- support a lightweight manual override through an opt-out label

The goal is to show a realistic but still manageable workflow automation example that combines:

- GitHub Actions
- Project v2 GraphQL mutations
- rule-based issue triage
- status synchronization over time

---

## Workflow Rules

### Scope Rule
Only issues with the `firmware` label are handled by the automation.

If an issue does not have `firmware`, the workflow logs that it was skipped and exits normally.

### Day 8 Rules: Initial Field Population
When a new firmware issue is opened:

- add the issue to the target GitHub Project v2
- set `Status = Todo`
- if the issue has `bug`, set `Priority = High`
- if the issue has `enhancement`, set `Priority = Medium`

### Day 9 Rules: Status Synchronization
For later issue events, the workflow recomputes status and updates the existing project item:

- `opened` -> `Todo`
- issue has `in-progress` label -> `In Progress`
- `closed` -> `Done`

Status precedence is:

`closed > in-progress > Todo`

### Day 10 Rule: Manual Override
If an issue has the `automation:skip` label:

- the workflow still records the event
- the workflow does not modify the project item
- logs clearly show that automation was intentionally skipped

### Day 11 Improvement: Existing Item Lookup
Before add or sync operations, the workflow checks whether the issue already exists in the project.

This supports:

- clearer logging
- safer update behavior
- a foundation for duplicate add protection

---

## Permissions and Secrets

The workflow requires a GitHub token with access to Project v2 operations.

### Required Secret

- `PROJECTS_TOKEN`

### Expected Capabilities
The token should be able to:

- read project metadata
- add issues to Project v2
- update Project v2 fields

If the token is missing or lacks permission, GraphQL queries or mutations will fail.

---

## Setup

### 1. Create or choose a test repository
Use a GitHub repository where you can safely test issue automation.

### 2. Create a GitHub Project v2
Create a user-owned or org-owned Project v2 board.

### 3. Add project fields
At minimum, configure:

- `Status`
  - `Todo`
  - `In Progress`
  - `Done`
- `Priority`
  - `High`
  - `Medium`
  - `Low`

### 4. Create issue labels
At minimum, create:

- `firmware`
- `bug`
- `enhancement`
- `in-progress`
- `automation:skip`

### 5. Add the workflow file
Place the workflow in:

`/.github/workflows/firmware-issue-demo.yml`

### 6. Add the secret
In repository settings, add:

- `PROJECTS_TOKEN`

### 7. Update project-specific IDs
If the workflow hardcodes Project v2 field IDs or option IDs, update them to match your project.

Typical values include:

- project id
- Status field id
- Todo / In Progress / Done option ids
- Priority field id
- High / Medium option ids

---

## How to Test

### Test 1: Initial Intake
Create a new issue with:

- `firmware`

Expected result:

- added to Project v2
- `Status = Todo`

### Test 2: Priority High
Create a new issue with:

- `firmware`
- `bug`

Expected result:

- added to Project v2
- `Status = Todo`
- `Priority = High`

### Test 3: Priority Medium
Create a new issue with:

- `firmware`
- `enhancement`

Expected result:

- added to Project v2
- `Status = Todo`
- `Priority = Medium`

### Test 4: In Progress Sync
On an existing firmware issue, add:

- `in-progress`

Expected result:

- workflow runs on `labeled`
- existing project item is found
- `Status = In Progress`

### Test 5: Done Sync
Close a firmware issue.

Expected result:

- workflow runs on `closed`
- existing project item is found
- `Status = Done`

### Test 6: Manual Override
Add:

- `automation:skip`

to a firmware issue, then trigger another issue event.

Expected result:

- workflow logs that automation was skipped
- project automation step does not run

---

## Re-run and Skip Behavior

### Re-run
If you want to re-run the workflow for a supported event:

- use GitHub Actions `Re-run jobs` for a past run, or
- trigger a new supported issue event such as adding or removing a label

### Skip
If a specific issue should not be modified by automation:

- add the `automation:skip` label

This acts as a manual override without disabling the workflow for the whole repository.

---

## Logging Approach

The workflow is designed to log three kinds of information:

### Context Logs
- event action
- issue title / number
- issue state

### Decision Logs
- whether the issue has `firmware`
- whether the issue has `automation:skip`
- the computed target status

### Action Logs
- whether an existing project item was found
- whether an item was added
- whether status or priority was updated
- whether automation was skipped

This makes it easier to distinguish:

- expected skips
- status decisions
- actual failures

---

## Known Limitations

- The workflow depends on hardcoded Project v2 field IDs and option IDs unless extra lookup logic is added.
- Existing item lookup currently assumes the relevant project item can be found within the queried item window.
- PR-merged to issue-done synchronization is not included in the first version.
- Duplicate add protection is partially addressed through existing-item lookup, but more defensive logic could still be added depending on how the opened path is extended.
- The workflow is optimized for demo clarity and interview scope, not full production hardening.

---

## Why This Project Is Useful

This demo is small enough to understand end to end, but realistic enough to discuss:

- workflow design
- GraphQL-based project automation
- triage rules
- state synchronization
- manual override design
- logging and operational safety

It is meant to show not just that automation can run, but that it can be reasoned about, tested, and explained clearly.
# github-actions-firmware-demo-
GitHub Actions test repo for firmware issue automation
