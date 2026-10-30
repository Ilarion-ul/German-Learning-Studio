# Codex repository instructions — German Learning Studio

## Product
Offline-first Qt 6/C++20/SQLite desktop learning app for an intensive A0→A2 German course (12 weeks, 30 hours weekly). See `APP_DEVELOPMENT_PLAN.md` for full requirements and `TEACHER_INSTRUCTIONS.md` for curriculum.

## Working rules
- Read the product plan and identify the smallest coherent change before coding.
- Keep UI (QML), domain/core (C++), storage (SQLite) and educational content (JSON) separate.
- Every DB schema change requires a migration and a test.
- Every scheduling, grading or timer change requires deterministic unit tests.
- Never claim an external app or service is integrated unless actually tested.
- Do not modify a real Anki collection database directly; use documented import/export or carefully validated optional adapters.
- Do not label a scheduler 'FSRS' unless it implements a verified FSRS algorithm and compatible tests.
- Do not copy copyrighted course or exam text/audio into the repository without appropriate permissions.
- Default offline. Ask before introducing paid dependencies, external API calls or data synchronization.
- Do not store API keys in source control. Use .gitignore and environment/config mechanisms.
- After each task report changed files, build/test commands actually run, results and remaining risks.
- Avoid wide refactors; preserve existing behavior and user data.

## First milestone
Set up Qt 6 Quick CMake app, SQLite migration mechanism, unit test harness, README with Windows build commands, and a bootable dashboard shell. Do not start with cloud/AI integration.
