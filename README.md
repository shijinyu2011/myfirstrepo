# Tasks: Count Directory Files (num_project)

**Input**: Design documents from `specs/001-count-dir-files/`
**Prerequisites**: plan.md (required), spec.md (required)

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [X] T001 Create project directory structure: num_project/, tests/
- [X] T002 Create num_project/__init__.py with version info
- [X] T003 [P] Create num_project/__main__.py entry point
- [X] T004 [P] Create tests/__init__.py
- [X] T005 Create README.md with usage documentation

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: None needed for this simple project. No shared infrastructure beyond project structure.

**Checkpoint**: Foundation ready - user story implementation can now begin.

---

## Phase 3: User Story 1 - Count Files in a Directory (Priority: P1) MVP

**Goal**: User can specify a directory and get the file count.

**Independent Test**: Run `python -m num_project <dir>` and see the correct count.

### Implementation for User Story 1

- [X] T006 [US1] Implement count_files() function in num_project/counter.py
- [X] T007 [US1] Implement CLI argument parsing and main() in num_project/cli.py
- [X] T008 [US1] Wire up __main__.py to call cli.main()
- [X] T009 [US1] Write unit tests for count_files() in tests/test_counter.py
- [X] T010 [US1] Write CLI integration tests in tests/test_cli.py

**Checkpoint**: `python -m num_project <directory>` returns the correct file count.

---

## Phase 4: User Story 2 - Recursive File Counting (Priority: P2)

**Goal**: User can count files recursively with --recursive flag.

**Independent Test**: Run `python -m num_project <dir> --recursive` and verify nested files are counted.

### Implementation for User Story 2

- [X] T011 [US2] Add count_files_recursive() to num_project/counter.py
- [X] T012 [US2] Add --recursive / -r argument to CLI in num_project/cli.py
- [X] T013 [US2] Add recursive counting tests in tests/test_counter.py
- [X] T014 [US2] Add recursive CLI tests in tests/test_cli.py

**Checkpoint**: `python -m num_project <dir> -r` counts all files in the tree.

---

## Phase 5: User Story 3 - Error Handling (Priority: P1)

**Goal**: User sees clear error messages for invalid inputs.

**Independent Test**: Run with non-existent path and verify error output.

### Implementation for User Story 3

- [X] T015 [US3] Add path validation logic in num_project/counter.py
- [X] T016 [US3] Add error handling and --json output in num_project/cli.py
- [X] T017 [US3] Add error scenario tests in tests/test_counter.py
- [X] T018 [US3] Add error and JSON output CLI tests in tests/test_cli.py

**Checkpoint**: All error scenarios produce clear messages, --json flag works.

---

## Phase 6: Polish & Cross-Cutting Concerns

- [X] T019 [P] Final README.md update with all features documented
- [X] T020 Run all tests and verify 100% pass rate

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies
- **US1 (Phase 3)**: Depends on Setup
- **US2 (Phase 4)**: Depends on US1 (extends counter.py and cli.py)
- **US3 (Phase 5)**: Depends on US1 (adds validation to existing modules)
- **Polish (Phase 6)**: Depends on all user stories

### Parallel Opportunities

- T003 and T004 can run in parallel (different files)
- T009 and T010 can run in parallel after T006-T008
- T019 can run in parallel with T020

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 3: User Story 1
3. **STOP and VALIDATE**: Test basic counting works
4. Continue with US2 and US3

### Summary

- **Total tasks**: 20
- **Phase 1 (Setup)**: 5 tasks
- **Phase 3 (US1 - Count)**: 5 tasks
- **Phase 4 (US2 - Recursive)**: 4 tasks
- **Phase 5 (US3 - Errors)**: 4 tasks
- **Phase 6 (Polish)**: 2 tasks
