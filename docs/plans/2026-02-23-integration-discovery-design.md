# Integration Discovery — Design

## Problem

The skill creates good standalone UPM packages, but doesn't consider existing packages when designing new ones. Packages end up isolated — no planned event subscriptions, no shared variable awareness, no integration surface documentation.

## Solution

Two additions to the skill:

### 1. Per-Package Integration Doc

Every package must generate `docs/packages/<package-name>.md` documenting its public integration surface: event channels, ScriptableVariables, RuntimeSets, interfaces, assembly info, and integration examples.

### 2. Integration Discovery Phase

Before designing a new package, the skill mandates reading all existing `docs/packages/*.md` files and producing an Integration Plan that covers:

- Existing events to listen to (via Version Defines)
- New events to publish
- Existing ScriptableVariables to read/write
- New ScriptableVariables to expose
- Interfaces to implement or expose
- Whether a Bridge Package is needed
- Suggested changes checklist for existing packages

## Changes

1. New SKILL.md section: "Integration Discovery" (after communication matrix)
2. New SKILL.md section: "Package Integration Doc" (after Sample Scene Generator)
3. New rules table row for mandatory integration docs
4. Two new checklist items
5. New reference file: `references/integration-discovery.md`
