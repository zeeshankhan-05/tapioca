# Contribution 1: Automatically resolve superclass redefinition errors while generating gem RBIs

**Contribution Number:** 1
**Student:** Zeeshan Khan
**Issue:** [https://github.com/Shopify/tapioca/issues/1834](https://github.com/Shopify/tapioca/issues/1834)
**Status:** Phase II Complete

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

- `Tapioca::Helpers::RbiFilesHelper#validate_rbi_files`

Other possible areas may include:

- Sorbet config handling
- RBI validation logic
- Tests related to generated RBI validation or strictness changes

---

## Reproduction Process

### Environment Setup

Completed on June 7, 2026.

**Local environment:**

- **OS:** macOS arm64, darwin 25.5.0
- **Local repo path:** `~/Projects/tapioca`
- **Fork URL:** [https://github.com/zeeshankhan-05/tapioca](https://github.com/zeeshankhan-05/tapioca)
- **Upstream URL:** [https://github.com/Shopify/tapioca](https://github.com/Shopify/tapioca)
- **Ruby:** 4.0.2 through rbenv
- **Bundler:** 4.0.10
- **Homebrew:** 5.1.15

**Setup verification:**

- Dependencies installed successfully with `bundle install`.
- Smoke test passed with `bundle exec exe/tapioca help`.
- Targeted test passed with `bin/test spec/tapioca/cli/help_spec.rb`.
- Targeted test result: 3 tests, 12 assertions, 0 failures.
- Full test suite was not completed yet.
- No setup errors occurred.

**Repository setup notes:**

- No `CONTRIBUTING.md` file was found in the repo.
- Setup references are `README.md` and `dev.yml`.
- `dev.yml` includes `bin/test`, `bin/typecheck`, and `bin/style`.

### Steps to Reproduce

This is a minimal Sorbet-level reproduction of the superclass redefinition behavior that issue #1834 asks Tapioca to handle.

1. From the Tapioca repo, create a temporary repro directory outside the repo:
  ```bash
   rm -rf /tmp/tapioca-1834-repro
   mkdir -p /tmp/tapioca-1834-repro/gem_rbis /tmp/tapioca-1834-repro/dsl_rbis
  ```
2. Create `/tmp/tapioca-1834-repro/gem_rbis/net_imap_literal.rbi`:
  ```rbi
   # typed: true

   class Net::IMAP::Literal < String
   end
  ```
3. Run Sorbet against the temporary RBI directories:
  ```bash
   bundle exec srb tc --no-config --error-url-base=https://srb.help/ /tmp/tapioca-1834-repro/dsl_rbis /tmp/tapioca-1834-repro/gem_rbis
  ```
4. Confirm that Sorbet reports a payload superclass redefinition error for `Net::IMAP::Literal`.
5. Confirm the suppression flag works:
  ```bash
   bundle exec srb tc --no-config --error-url-base=https://srb.help/ --suppress-payload-superclass-redefinition-for=Net::IMAP::Literal /tmp/tapioca-1834-repro/dsl_rbis /tmp/tapioca-1834-repro/gem_rbis
  ```

### Reproduction Evidence

- **Working branch:** [https://github.com/zeeshankhan-05/tapioca/tree/fix-issue-1834](https://github.com/zeeshankhan-05/tapioca/tree/fix-issue-1834)
- **Reproduction command:**
  ```bash
  bundle exec srb tc --no-config --error-url-base=https://srb.help/ /tmp/tapioca-1834-repro/dsl_rbis /tmp/tapioca-1834-repro/gem_rbis
  ```
- **Observed error excerpt:**
  ```text
  /tmp/tapioca-1834-repro/gem_rbis/net_imap_literal.rbi:3: Parent of class `Net::IMAP::Literal` redefined from `Net::IMAP::CommandData` to `String` https://srb.help/5012
       3 |class Net::IMAP::Literal < String
                                     ^^^^^^
      https://github.com/sorbet/sorbet/tree/bd88920e92609a8965bf8ccce34b5b61d20ff271/rbi/stdlib/net.rbi#L5040: Originally defined here
      5040 |class Net::IMAP::Literal < Net::IMAP::CommandData
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    Note:
      Pass `--suppress-payload-superclass-redefinition-for=Net::IMAP::Literal` at the command line or in the `sorbet/config` file to silence this error.
  Errors: 1
  ```
- **Suppression verification:** Running the same command with `--suppress-payload-superclass-redefinition-for=Net::IMAP::Literal` returned `No errors! Great job.`
- **Tapioca validation-path note:** The current `validate_rbi_files` command shape uses `--stop-after namer`; with that flag, this temporary payload conflict did not surface and Sorbet returned `No errors! Great job.`
- **Tapioca CLI note:** A controlled `tapioca gem net-imap` run using `/tmp` output completed successfully and changed the generated RBI strictness to `typed: false`; it did not reproduce the payload suppression behavior directly through the CLI.
- **My findings:** The underlying Sorbet behavior is confirmed. The error code is `5012`, the constant is `Net::IMAP::Literal`, and Sorbet explicitly recommends adding `--suppress-payload-superclass-redefinition-for=Net::IMAP::Literal` to suppress this class of payload superclass mismatch.

---

## Solution Approach

### Analysis

Sorbet reports payload superclass redefinition as error `5012`. In my minimal reproduction, the error output included the exact constant (`Net::IMAP::Literal`) and Sorbet’s suggested suppression flag (`--suppress-payload-superclass-redefinition-for=Net::IMAP::Literal`).

Tapioca already validates generated RBI files in `Tapioca::Helpers::RBIFilesHelper#validate_rbi_files`. That method:

- runs Sorbet with `--no-config`, `--error-url-base=...`, and `--stop-after namer` against the DSL and gem RBI directories
- parses Sorbet output with `Spoom::Sorbet::Errors::Parser.parse_string`
- handles parse errors (`code < 4000`) by raising `Tapioca::Error`
- handles other validation conflicts through `auto_strictness`, which currently filters error `4010` and calls `update_gem_rbis_strictnesses` to change conflicting gem RBI files to `typed: false`

Tapioca does not currently appear to automatically convert payload superclass redefinition errors into `sorbet/config` suppression entries. The repo already contains manual suppressions in `sorbet/config` for `Net::IMAP::CommandData` and `Net::IMAP::Literal`, which suggests maintainers have handled this case manually before.

Important caveat from reproduction: the confirmed reproduction was Sorbet-level, not through the current Tapioca CLI validation path. With the current helper command shape (`--stop-after namer`), the temporary payload conflict did not surface. That means the implementation likely needs either:

- an adjustment to the validation command so payload superclass errors are detected, or
- targeted test coverage that exercises the full Sorbet validation behavior needed for error `5012`

### Proposed Solution

Extend `validate_rbi_files` so that when Sorbet reports error `5012`, Tapioca extracts the constant name, appends the matching `--suppress-payload-superclass-redefinition-for=ConstantName` line to `sorbet/config` if it is not already present, and prints a clear user-facing warning about the superclass mismatch and the suppression that was added.

### Implementation Plan

Using UMPIRE framework:

**Understand:** When Tapioca validates generated gem RBIs, some generated definitions can redefine a Sorbet payload class with a different superclass. Sorbet reports this as error `5012` and recommends a suppression flag. Tapioca should add that suppression to `sorbet/config` automatically, but still tell the user what happened instead of silently hiding the mismatch.

**Match:** Closest existing patterns in the codebase:

- **Sorbet error parsing:** `validate_rbi_files` already parses Sorbet stderr with `Spoom::Sorbet::Errors::Parser` and branches on `error.code`
- **Automatic remediation for validation conflicts:** `auto_strictness` already handles error `4010` by calling `update_gem_rbis_strictnesses`
- **User-facing output style:** `update_gem_rbis_strictnesses` uses `say(..., [:yellow, :bold])`; other commands use `say_error(..., :yellow)` for warnings
- **Config file reading:** `Spoom::Sorbet::Config.parse_file` is already used in `lib/tapioca/static/requires_compiler.rb`
- **Config file creation:** `lib/tapioca/commands/configure.rb` creates `sorbet/config`, but there is no existing helper for appending suppression flags during validation
- **Existing tests:** strictness behavior is covered in `spec/tapioca/cli/gem_spec.rb` and `spec/tapioca/cli/dsl_spec.rb` through CLI integration tests

**Plan:**

1. Inspect how `Spoom::Sorbet::Errors::Error` represents error `5012`, including whether the constant name can be parsed from `message` or `more` lines containing `--suppress-payload-superclass-redefinition-for=...`.
2. Add handling in `validate_rbi_files` for Sorbet error `5012` alongside the existing `4010` auto-strictness path.
3. Extract the constant name from Sorbet’s suggested suppression flag or error output.
4. Add helper logic to update `sorbet/config` with:
  `--suppress-payload-superclass-redefinition-for=ConstantName`
5. Avoid duplicate suppression entries if `sorbet/config` already contains the line.
6. Print a clear user-facing warning explaining:
  - which constant had a payload superclass mismatch
  - what suppression line was added
  - that the user should review the generated RBI and config change
7. Decide whether the current validation command needs adjustment so error `5012` is actually detected during Tapioca validation, since `--stop-after namer` did not surface the reproduced payload conflict.
8. Add or update tests covering:
  - detection of error `5012`
  - writing the suppression line to `sorbet/config`
  - not duplicating an existing suppression line
  - user still gets informed through stdout/warning output
  - existing `4010` auto-strictness behavior still works

**Implement:** [https://github.com/zeeshankhan-05/tapioca/tree/fix-issue-1834](https://github.com/zeeshankhan-05/tapioca/tree/fix-issue-1834)

**Review:** Before opening a PR, check:

- `bin/style`
- `bin/typecheck`
- relevant tests with `bin/test`
- only intended files changed
- no unrelated `Gemfile.lock` setup change is included

**Evaluate:** I will verify the fix by:

- reproducing the Sorbet-level failure from Phase II Step 3 before making changes
- running the same scenario after the fix through Tapioca’s validation path or targeted tests
- confirming `sorbet/config` receives the expected suppression line
- confirming the user-facing warning/message appears
- running targeted tests and available project checks (`bin/test`, `bin/typecheck`, `bin/style`)

---

## Testing Strategy

### Unit Tests

- [ ] Add helper-level coverage for parsing error `5012` and extracting the constant name from Sorbet output.
- [ ] Add helper-level coverage for appending `--suppress-payload-superclass-redefinition-for=ConstantName` to `sorbet/config`.
- [ ] Add helper-level coverage to ensure duplicate suppression lines are not added when the config already contains the constant.

### Integration Tests

- [ ] Add CLI/integration coverage in `spec/tapioca/cli/gem_spec.rb` and/or `spec/tapioca/cli/dsl_spec.rb` for the new suppression behavior.
- [ ] Verify the user still receives a warning/message when Tapioca adds a suppression entry.
- [ ] Verify existing `4010` auto-strictness behavior still works and is not broken by the new logic.

### Manual Testing

- [ ] Re-run the minimal Sorbet-level reproduction from `/tmp/tapioca-1834-repro` to confirm the underlying error still reproduces before the fix.
- [ ] After implementation, run the Tapioca validation path or targeted tests and confirm `sorbet/config` is updated as expected.
- [ ] Confirm the warning output explains the superclass mismatch and the added suppression line.

---

## Implementation Notes

### Week 1 Progress

Selected Shopify/tapioca issue #1834 for my CodePath AI301 Open Source Capstone. I commented on the GitHub issue expressing interest, claimed the issue on the CodePath Google Sheet, and started documenting my understanding in this Contribution README.

### Week 2 Progress

Completed Phase II by setting up the local environment, creating the working branch, reproducing the superclass redefinition behavior at the Sorbet level, and writing a UMPIRE-based solution plan.

### Code Changes

- **Files modified:** Not started yet.
- **Key commits:** Not started yet.
- **Approach decisions:** Not started yet.

---

## Pull Request

**PR Link:** Not submitted yet.

**PR Description:** Not started yet.

**Maintainer Feedback:**

- No maintainer feedback received yet.

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

- Shopify/tapioca issue #1834: [https://github.com/Shopify/tapioca/issues/1834](https://github.com/Shopify/tapioca/issues/1834)
- Shopify/tapioca repository: [https://github.com/Shopify/tapioca](https://github.com/Shopify/tapioca)
- CodePath AI301 Phase I instructions
- First Contributions tutorial

