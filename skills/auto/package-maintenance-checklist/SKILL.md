---
name: package-maintenance-checklist
description: Use when fixing bugs in a code package with repository rules for tests, typing, and release notes.
---
- Inspect package conventions and existing tests before editing.
- Do not modify existing test files; add new regression tests separately.
- Add a focused regression test for every bug fixed, and run the full test suite.
- Annotate every parameter and return value of each public function.
- Record each fix under the changelog’s `Unreleased` heading using `- fix(function_name): short description`.
- Review the diff to confirm only intended files changed.
