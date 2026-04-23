# PlanCode Specification v0.1

## Purpose

PlanCode defines a planning-as-code format for project schedules.

It is designed to be:

- human-readable
- machine-validatable
- deterministic
- renderer-agnostic

## Top-level Structure

A PlanCode document may contain:

```yaml
version: "0.1"
project: {}
calendar: {}
groups: []
tasks: []
milestones: []
links: []
render: {}
```

## Project

```yaml
project:
  id: string
  name: string
  timezone: string
  start_date: YYYY-MM-DD
```

## Calendar

```yaml
calendar:
  working_days: [Mon, Tue, Wed, Thu, Fri]
  holidays:
    - YYYY-MM-DD
```

Semantics:

- `working_days` is required
- `holidays` is optional
- `duration: 1d` means one working day

## Groups

```yaml
groups:
  - id: "dev"
    title: "Development"
    parent: "phase1"
```

## Tasks

```yaml
tasks:
  - id: "TASK-1"
    title: "Requirements"
    group: "plan"
    start: 2026-03-10
    end: 2026-03-16
    duration: 5d
    depends_on:
      - task: "TASK-0"
        type: "FS"
        lag: 0d
```

Rules:

- `id` and `title` are required
- a task must define enough information to resolve dates:
  - `start + duration`, or
  - `start + end`, or
  - `depends_on + duration`, or
  - `depends_on + end`

## Dependencies

Supported dependency types:

- `FS` finish-to-start
- `SS` start-to-start
- `FF` finish-to-finish
- `SF` start-to-finish

v0.1 implementation focuses on `FS`.

## Milestones

```yaml
milestones:
  - id: "ALPHA"
    title: "Alpha Release"
    depends_on: ["IOS"]
    date_policy: "auto"
```

## Links

```yaml
links:
  - target: "IOS"
    kind: "github_issue"
    ref: "#123"
```

## Duration Format

Supported units:

- `d` = working days
- `w` = working weeks
- `h` = working hours

Examples:

```yaml
duration: 5d
duration: 2w
duration: 8h
```

## Scheduling Semantics

Resolution order:

1. explicit start
2. dependency result
3. project start date

Rules:

- holidays are skipped
- non-working days are skipped
- cyclic dependencies are invalid
- an `FS` dependency means the successor starts on the next working day after predecessor finish
- `lag` shifts the successor after dependency resolution

## Validation

### Errors

- duplicate IDs
- unknown group reference
- unknown dependency reference
- cyclic dependency
- invalid date format
- invalid duration format
- unresolved task lacking scheduling basis

### Warnings

- missing owner
- missing group
- orphan milestone

## Rendering

Renderers consume the resolved plan, not the raw plan.
