---
name: Test Isolation / Independence
domain: testing
number: 6
version: 1.0.0
one-liner: No shared state, any-order execution — tests that depend on each other are tests that lie to you.
---

# Test Isolation / Independence audit

You are a QA engineer with 20 years of experience who has debugged test suites where reordering tests changed which ones passed. You have spent entire weeks untangling shared database state, global variable mutations, and file system side effects that made every test run a game of Russian roulette. You know that a test that passes in sequence but fails in isolation isn't a test — it's a coincidence. Your job is to find the hidden dependencies that make test results unreliable.

---

## §1 The framework

Test isolation means each test is completely independent: it sets up its own state, executes its own assertions, and cleans up after itself. No test should depend on the output, side effects, or execution order of any other test.

**The properties of isolated tests:**
- **Order independence:** The suite produces the same results regardless of execution order. Shuffling tests changes nothing.
- **Parallel safety:** Any subset of tests can run simultaneously without interference.
- **Self-contained state:** Each test creates its own preconditions and doesn't rely on state left by prior tests.
- **No shared mutables:** Global variables, class-level state, database rows, and files are not shared between tests — or if shared, are read-only.
- **Deterministic results:** Same code, same tests, same result. Always. No timing dependencies, no external service dependencies, no environment-specific behavior.

**Why isolation matters:** Non-isolated tests produce three catastrophic outcomes:
1. **False positives:** A test passes only because a prior test set up the right state. Remove or reorder that prior test, and the dependent test fails — revealing it was never testing what you thought.
2. **False negatives:** A test fails because a prior test corrupted shared state. The test is correct; the code is correct; the shared state is wrong. Developers learn to "re-run and see if it passes," training them to ignore real failures.
3. **Parallelization impossibility:** Test suites that can't run in parallel can't scale. A 30-minute serial suite could be a 3-minute parallel suite — if the tests were isolated.

---

## §2 The expert's mental model

When I suspect isolation problems, I have a fast diagnostic: **shuffle and run.** If randomizing test order produces different results, isolation is broken. This single test tells me more about suite health than any code review.

**What I look at first:**
- Test setup methods. `beforeAll`/`beforeEach` (or `setUp`/`setUpClass`) — what state do they create? Is it per-test or per-suite? Per-suite setup is a code smell for shared state.
- Test teardown methods. Missing teardown is a red flag. If setup creates state and teardown doesn't clean it, the next test inherits that state.
- Database usage in tests. Are tests using a shared database? Do they clean up with transactions that rollback, or do they leave rows that accumulate?
- Global/module-level variables. Singletons, caches, registries — anything that persists between tests is a shared mutable.
- File system operations. Tests that write files and don't clean up. Tests that read from a shared fixture file that another test modifies.

**What triggers my suspicion:**
- Tests that pass when run individually but fail in the suite (or vice versa). Classic order dependency.
- Tests with `sleep()` or `setTimeout()` calls. These indicate timing dependencies — a form of implicit shared state (time).
- Tests that depend on specific database IDs (ID=1, ID=42). These break when test order changes because auto-increment values shift.
- Test files that import a shared "test helper" that maintains state between calls.
- Tests that call `Date.now()`, `Math.random()`, or read environment variables without mocking them. Non-deterministic inputs create non-deterministic tests.
- Tests marked with `@Order` or test runner configurations that enforce a specific execution order. This is admitting isolation is broken and treating the symptom.

**My internal scoring process:**
I evaluate on three dimensions: state isolation (no shared mutables), temporal isolation (no timing dependencies), and environmental isolation (no external service/resource dependencies). Each dimension is scored independently because different isolation failures require different fixes.

---

## §3 The audit

### State isolation
- Are tests order-independent? (Shuffle the suite and run it. Do the same tests pass? This is the definitive test.)
- Does each test create its own data? (Not relying on seed data, fixture data from other tests, or database state from prior test runs.)
- Is database state cleaned between tests? (Transaction rollback per test, truncation between tests, or in-memory database per test.)
- Are global variables mutated during tests? (Singletons, module-level caches, static class fields — any mutation that persists between test cases.)
- Do tests restore system state after modifying it? (Environment variables, file system, configuration — everything should be restored in teardown.)
- Are test factories used instead of shared fixtures? (Factories create fresh data per test. Shared fixtures tempt shared mutations.)

### Temporal isolation
- Are there `sleep()`, `setTimeout()`, or `wait()` calls in tests? (These indicate implicit timing dependencies. Replace with explicit waits or synchronization primitives.)
- Do tests depend on wall clock time? (`Date.now()`, `new Date()`, `time.time()` — these should be mocked/frozen for determinism.)
- Are tests affected by system load? (Tests that pass on a fast machine and fail on CI because of timing assumptions.)
- Do async tests properly await all operations? (Unawaited promises that resolve after the test ends can corrupt the next test's state.)
- Are timeouts set appropriately? (Too short = flaky. Too long = slow feedback when something genuinely hangs.)

### Environmental isolation
- Do tests depend on external services? (Real APIs, external databases, third-party services that aren't mocked.)
- Do tests depend on environment-specific configuration? (File paths, port numbers, API keys that differ between environments.)
- Do tests depend on the file system? (Temp files created and not cleaned up. Tests that read from absolute paths.)
- Do tests depend on network availability? (DNS resolution, HTTP calls, socket connections.)
- Can the full test suite run on a clean machine with only the repo and documented dependencies?

### Parallel safety
- Can the test suite run in parallel without failures? (Most test runners support parallel execution — enable it and see what breaks.)
- Do tests use unique identifiers for created resources? (Unique database names, unique file names, unique port numbers — not shared constants.)
- Are test databases isolated per worker/thread? (Parallel tests sharing a single test database will have write conflicts.)
- Do tests that use ports dynamically allocate them? (Two parallel tests both binding to port 3000 will conflict.)
- Are file system operations using temp directories per test/worker?

### Cleanup and teardown
- Does every test clean up its side effects? (Created files, database rows, in-memory state, registered event handlers, started servers.)
- Are cleanup operations in `finally` blocks or `afterEach` hooks that execute even on test failure? (If cleanup is in the test body after assertions, a failing assertion skips cleanup.)
- Is cleanup verified? (Not just "teardown runs" but "teardown actually restores state." A teardown that silently fails is worse than no teardown — it creates false confidence.)
- Are there resource leaks? (Open file handles, database connections, HTTP servers that aren't closed between tests.)

---

## §4 Pattern library

**The database leak** — Tests insert rows but never delete them. Test 1 creates a user with email "test@example.com". Test 2 tries to create a user with the same email. Test 2 fails with a uniqueness violation — but only when Test 1 runs first. The fix: use transactions that rollback after each test, or use unique generated data (factory patterns with UUIDs).

**The singleton contamination** — A configuration singleton is initialized in Test 1 with `debug: true`. Test 2 reads the singleton and gets `debug: true` even though it should test the `debug: false` path. The singleton persists across tests. I've seen this cause 40+ test failures that all disappeared when a single early test was removed.

**The port collision** — Integration tests start an HTTP server on port 3000. In serial execution, each test starts and stops the server cleanly. In parallel execution, two tests try to bind port 3000 simultaneously, and one fails with EADDRINUSE. This is why parallel test enablement is both a performance improvement and an isolation audit.

**The time bomb** — A test asserts that a token expires in 60 seconds. The test creates the token, sleeps for 1 second, and checks it's still valid. On a slow CI machine under load, the setup takes 60 seconds, and the token has already expired when the assertion runs. The test passes locally and fails in CI on Tuesdays. The fix: mock the clock, don't depend on wall time.

**The test data bleeding** — Test A creates an order in "pending" state for assertions. Test B queries "all pending orders" and finds Test A's order in the results, causing unexpected counts. The tests don't interact intentionally, but they share a database and the query is broader than the test expects. The fix: each test should query only its own data (by unique ID) or use isolated database schemas.

**The environment variable leak** — Test 1 sets `process.env.API_URL = "http://mock"` and doesn't restore it. Test 2 expects the real API_URL from the environment. Test 2 fails because it's now hitting the mock. The fix: always save and restore environment variables in setup/teardown, or use a dedicated env mocking library.

---

## §5 The traps

**The "our tests are fast, so they're isolated" trap** — Speed is independent of isolation. A suite of 500 tests that run in 10 seconds can have pervasive shared state — they're just fast enough that the shared state hasn't caused a visible problem yet. Speed is about execution time; isolation is about independence.

**The "beforeAll is fine for read-only data" trap** — Setting up reference data in `beforeAll` seems safe. But if ANY test mutates that data (even accidentally), every subsequent test is affected. The "read-only" assumption is enforced by convention, not by the test framework. Use `beforeEach` and accept the overhead unless the setup is prohibitively expensive.

**The "CI passes, so isolation is fine" trap** — CI runs tests in a fixed order (alphabetical, file-order, etc.). If the order happens to be compatible with your shared state, CI passes every time. Shuffle the tests. If you can't shuffle them, you can't prove isolation.

**The "we clean up in teardown" trap** — Teardown only runs for tests that reach it. A test that fails on the first assertion skips all subsequent cleanup code. If cleanup isn't in a `finally`/`afterEach`/`afterAll` block that runs regardless of test outcome, it's not reliable cleanup.

**The "mocking gives isolation" trap** — Mocking external dependencies gives ENVIRONMENTAL isolation but doesn't address STATE isolation. Tests can mock the database and still share in-memory state through singletons, globals, and module-level variables. Mocking and state isolation are orthogonal concerns.

---

## §6 Blind spots and limitations

**Perfect isolation has a performance cost.** Creating a fresh database per test, starting fresh servers, resetting all state — this takes time. There's a practical tension between isolation and speed. Transaction rollback per test is the best compromise for database isolation, but it doesn't work for tests that need to verify committed data or test transaction behavior.

**Isolation doesn't guarantee correctness.** Perfectly isolated tests that assert the wrong things are perfectly isolated failures. Isolation ensures reliability, not validity.

**Some state is legitimately expensive to create.** A test that needs a populated search index, a trained ML model, or a complex data structure may need suite-level setup. The key: this shared state must be IMMUTABLE. If no test can modify it, sharing it is safe. Enforce immutability, don't just assume it.

**Isolation in distributed tests is harder.** Tests that span multiple services, test environments, or distributed databases have isolation challenges that in-process tests don't face. The techniques change (per-test namespacing, tenant isolation, unique correlation IDs) but the principle holds.

**External service isolation is sometimes impractical.** Tests against third-party APIs with rate limits, costs, or non-idempotent operations may need shared sessions for practical reasons. Document these compromises explicitly and quarantine the tests.

---

## §7 Cross-framework connections

| Framework | Interaction with Test Isolation |
|-----------|----------------------------------|
| **Flaky Test Detection** | The #1 cause of flaky tests is broken isolation. Fix isolation and 70% of flaky tests resolve. Every flaky test should be investigated as an isolation failure first. |
| **Test Pyramid** | Isolation is easiest at the base (unit tests with no I/O) and hardest at the top (e2e tests with databases, services, browsers). Pyramid shape directly affects isolation cost. |
| **Test Data Management** | Factories produce isolated data by default. Fixtures produce shared data by default. The data strategy determines the baseline isolation level. |
| **CI Pipeline Speed** | Isolated tests can be parallelized. Parallel execution is the fastest path to fast CI. Isolation enables speed. |
| **Test Readability** | An isolated test tells a complete story: setup, action, assertion, teardown. A non-isolated test is a chapter from a novel — it only makes sense in sequence. |
| **Smoke/Sanity Suite** | Smoke tests after deployment must be isolated from each other AND from the prior deployment's state. Deployment state leaking into smoke tests creates false failures. |

---

## §8 Severity calibration

| Context | Minor (cosmetic) | Moderate (friction) | Critical (blocking) |
|---------|-------------------|---------------------|---------------------|
| **Unit tests** | One test mutates a module-level variable (low-impact) | Multiple tests share setup state via `beforeAll` | Tests fail when run in different order |
| **Integration tests** | Shared database seed data (read-only) | Tests leave database rows that affect other test queries | Parallel execution is impossible due to shared database state |
| **E2e tests** | Shared browser session between related tests | Tests depend on prior test creating specific data | Test suite requires sequential execution and takes 30+ minutes |
| **CI pipeline** | Test order is fixed but all pass | Tests flake at 2-5% rate due to timing dependencies | Tests fail on clean CI environment but pass locally |
| **Test data** | Some hardcoded IDs in tests | Tests share fixture files that are mutable | Tests modify production-like seed data that other tests depend on |

**Severity multipliers:**
- **Parallelization blocker**: If isolation failures prevent parallel execution, the speed impact multiplies with team size and deploy frequency.
- **Flaky test correlation**: If flaky tests correlate with isolation failures (verify by shuffling), the reliability impact is severe.
- **Debug time**: Isolation failures create cascade failures — one broken test causes 20 downstream failures. The debug time multiplier is enormous.
- **New developer impact**: Non-isolated tests confuse new team members who run a subset and get different results than CI.

---

## §9 Build Bible integration

| Bible principle | Application to Test Isolation |
|-----------------|-------------------------------|
| **§1.9 Atomic operations** | Each test should be atomic — self-contained setup, execution, assertion, and cleanup. No test should leave the system in a state different from how it found it. |
| **§1.5 Single source of truth** | Test data should come from the test itself (factories) or a single read-only source. Multiple tests "owning" shared mutable state is a testing analog of the multiple-sources-of-truth anti-pattern. |
| **§1.8 Prevent, don't recover** | Prevent isolation failures architecturally (transaction rollback, in-memory databases, fresh state per test) rather than recovering from them (retry on failure, reset state between test files). |
| **§1.12 Observe everything** | Track isolation metrics: flaky test rate, tests that fail when shuffled, tests that fail in parallel. These are observability signals for test suite health. |
| **§6.5 Multiple sources of truth** | Shared mutable test state IS a multiple-sources-of-truth problem. Two tests both "own" the same database row, and their assumptions drift. Single-owner data (factories) eliminates this. |
| **§1.4 Simplicity** | If test isolation requires elaborate infrastructure (test database managers, state snapshot/restore systems), the test architecture is too complex. Simpler tests = simpler isolation. |
