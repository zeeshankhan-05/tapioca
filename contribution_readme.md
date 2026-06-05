# Contribution 1: Automatically resolve superclass redefinition errors while generating gem RBIs

**Contribution Number:** 1
**Student:** Zeeshan Khan
**Issue:** https://github.com/Shopify/tapioca/issues/1834
**Status:** Phase I In Progress

---

## Why I Chose This Issue

I chose this issue because it is open, unassigned, labeled `good-first-issue`, and has a clear expected behavior. The issue is in `Shopify/tapioca`, which is a recognizable open-source project from Shopify, so it gives me a chance to work on a real developer-tooling project while also building a contribution that would be valuable to discuss on my resume.

This issue also seems realistic for a first open-source PR because the problem is specific and connected to an existing part of the codebase: RBI validation. I am interested in learning more about how Tapioca works with Sorbet, how generated RBI files are checked, and how open-source projects handle errors in a way that helps users without hiding important information.

---

## Understanding the Issue

### Problem Description

Tapioca generates RBI files for Ruby projects, then validates those generated files using Sorbet. The issue is that some generated RBI files can cause superclass redefinition errors. Right now, Tapioca does not automatically add the needed Sorbet config suppression for those specific constants, so users may have to deal with the error manually.

### Expected Behavior

When Tapioca detects a superclass redefinition error during RBI validation, it should update `sorbet/config` with the appropriate suppression line:

`--suppress-payload-superclass-redefinition-for=ConstantName`

However, Tapioca should still inform the user about the mismatch instead of silently hiding the problem.

### Current Behavior

Tapioca already validates generated RBI files and has logic related to changing RBI strictness when needed, but it does not currently appear to automatically add the superclass redefinition suppression to `sorbet/config`.

### Affected Components

The main area likely involved is:

* `Tapioca::Helpers::RbiFilesHelper#validate_rbi_files`

Other possible areas may include:

* Sorbet config handling
* RBI validation logic
* Tests related to generated RBI validation or strictness changes

---

## Reproduction Process

### Environment Setup

Not started yet. This will be completed in Phase II.

### Steps to Reproduce

Not started yet. This will be completed in Phase II after setting up the local development environment.

### Reproduction Evidence

* **Commit showing reproduction:** Not started yet.
* **Screenshots/logs:** Not started yet.
* **My findings:** Not started yet.

---

## Solution Approach

### Analysis

Not started yet. This will be completed after reproducing the issue in Phase II.

### Proposed Solution

Based on the issue description, the likely solution will involve detecting superclass redefinition errors during RBI validation and adding the correct `--suppress-payload-superclass-redefinition-for=ConstantName` entry to `sorbet/config`, while still showing a warning or message to the user.

### Implementation Plan

Using UMPIRE framework:

**Understand:** Tapioca should handle Sorbet superclass redefinition errors more automatically during generated RBI validation.

**Match:** I will look for existing patterns in the codebase around RBI validation, automatic strictness changes, user warnings, and config file updates.

**Plan:** Not started yet. This will be completed in Phase II after reproducing the issue and reading the relevant code.

**Implement:** Not started yet.

**Review:** Not started yet.

**Evaluate:** Not started yet.

---

## Testing Strategy

### Unit Tests

* [ ] Not started yet.

### Integration Tests

* [ ] Not started yet.

### Manual Testing

Not started yet.

---

## Implementation Notes

### Week 1 Progress

Selected Shopify/tapioca issue #1834 for my CodePath AI301 Open Source Capstone. I commented on the GitHub issue expressing interest, claimed the issue on the CodePath Google Sheet, and started documenting my understanding in this Contribution README.

### Code Changes

* **Files modified:** Not started yet.
* **Key commits:** Not started yet.
* **Approach decisions:** Not started yet.

---

## Pull Request

**PR Link:** Not submitted yet.

**PR Description:** Not started yet.

**Maintainer Feedback:**

* No maintainer feedback received yet.

**Status:** Not started yet.

---

## Learnings & Reflections

### Technical Skills Gained

Not started yet.

### Challenges Overcome

Not started yet.

### What I'd Do Differently Next Time

Not started yet.

---

## Resources Used

* Shopify/tapioca issue #1834: https://github.com/Shopify/tapioca/issues/1834
* Shopify/tapioca repository: https://github.com/Shopify/tapioca
* CodePath AI301 Phase I instructions
* First Contributions tutorial
