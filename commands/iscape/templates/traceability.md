# Traceability Template

Template for `context/traceability.md` — requirements to phase mapping.

<template>

# Requirements Traceability Matrix

Maps PRD requirements to implementation phases.

## Functional Requirements

| ID | Requirement | PRD Section | Milestone | Phase | Status |
|----|-------------|-------------|-----------|-------|--------|
| FR-001 | [Requirement name] | 05-functional#section | v0.1 | 01-poc | Not Started |
| FR-002 | [Requirement name] | 05-functional#section | v0.1 | 02-name | In Progress |
| FR-003 | [Requirement name] | 05-functional#section | v0.1 | 02-name | Complete |

## Non-Functional Requirements

| ID | Requirement | PRD Section | Milestone | Phase | Status |
|----|-------------|-------------|-----------|-------|--------|
| NFR-001 | [Requirement name] | 06-non-functional#section | v0.1 | 01-poc | Not Started |

## Coverage Summary

| Milestone | Total Reqs | Not Started | In Progress | Complete | Coverage |
|-----------|------------|-------------|-------------|----------|----------|
| v0.1 | 0 | 0 | 0 | 0 | 0% |

---
*Last updated: [date]*

</template>

<guidelines>

**Purpose:**
- Single source of truth for requirement coverage
- Links PRD requirements to implementation
- Tracks status across milestones

**Columns:**
- ID: Unique identifier (FR-XXX, NFR-XXX)
- Requirement: Brief name/description
- PRD Section: Reference to PRD file and section
- Milestone: Which milestone addresses this
- Phase: Specific phase within milestone
- Status: Not Started / In Progress / Complete

**Coverage Summary:**
- Aggregate view per milestone
- Calculate coverage percentage
- Updated when phases complete

**Maintenance:**
- Update status when phase completes
- Add new requirements as PRD evolves
- Mark requirements deferred if moved to future milestone

</guidelines>
