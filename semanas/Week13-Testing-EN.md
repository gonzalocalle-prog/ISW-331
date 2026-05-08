# Week 13 - Testing

> **Block 4: DevOps and CI/CD**
> This week covers why automated testing matters, how the testing pyramid helps teams balance speed and confidence, and how unit and integration tests catch different classes of failure. The goal is not only to write tests, but to understand what each test level protects, when to use it, and how good tests support safer refactoring and faster delivery.

---

## Part 1 - Why Testing Exists

### Testing is about managing risk, not chasing perfection

Students often think testing exists to prove that software has no bugs. That is impossible.

The real purpose of testing is this:

> **Testing reduces uncertainty so teams can change software with less fear.**

Every code change creates risk:

- A new feature can break an old one
- A refactor can change behavior accidentally
- A bug fix can introduce a second bug somewhere else
- A UI change can disconnect from the API contract

Without tests, teams rely on memory, manual clicking, and luck. That approach fails as soon as the codebase grows.

### The hidden cost of not testing

When teams skip testing, the cost does not disappear. It moves later in the process, where it becomes more expensive.

| When a bug is found | Typical cost |
|---------------------|--------------|
| **While writing the code** | Cheap and fast to fix |
| **During code review or CI** | Manageable, but interrupts flow |
| **During QA or staging** | Slower, affects other people |
| **In production** | Expensive, stressful, public |

Testing shortens the feedback loop.

```text
Code change
   -> Automated test feedback
   -> Fix while context is still fresh
   -> Merge with more confidence
```

> **Key insight:** Good tests do not remove all bugs. They make bugs cheaper to detect and safer to prevent.

---

## Part 2 - The Testing Pyramid

The testing pyramid is a strategy for balancing speed, cost, and confidence.

```text
             /  \
            /    \
           /  E2E \
          /--------\\
         /Integration\\
        /-------------\\
       /  Unit Tests   \\
      /_________________\\
```

The idea is simple:

- Have **many unit tests** because they are fast and focused
- Have **some integration tests** because they validate real collaboration between parts
- Have **few end-to-end tests** because they are slower and more fragile

### Why the pyramid matters

If a team writes only end-to-end tests, feedback becomes slow and failures are hard to diagnose.

If a team writes only unit tests, important system-level bugs may slip through.

The pyramid is not a law. It is a design principle:

> **Place most of your tests where they are cheap and precise, then add fewer higher-level tests where they provide unique confidence.**

### A practical interpretation for this course

For Week 13, the most useful focus is:

- **Unit tests** for pure logic, helpers, hooks, services, and validation rules
- **Integration tests** for UI behavior, component interaction, API boundaries, and database-backed flows

You can mention end-to-end tests conceptually, but the course deliverable for this week emphasizes unit and integration tests.

---

## Part 3 - Unit Tests: Fast, Focused, and Local

### What a unit test checks

A unit test verifies one small piece of behavior in isolation.

Examples:

- A function that calculates totals
- A password validator
- A React custom hook
- A service method that transforms API data
- A permission rule such as `canEditProject(user, project)`

### Why unit tests are valuable

Unit tests are usually:

- Fast to run
- Easy to diagnose when they fail
- Cheap to maintain when they test behavior instead of implementation details

This makes them ideal for protecting core business logic.

### What a good unit test looks like

A strong unit test is:

- **Small** - one clear scenario
- **Deterministic** - same result every time
- **Readable** - another developer can understand what behavior matters
- **Behavior-focused** - it checks outcomes, not internal implementation trivia

### Example: testing pure logic with Vitest

```ts
import { describe, expect, it } from 'vitest';
import { calculateDiscount } from './pricing';

describe('calculateDiscount', () => {
  it('applies a 10 percent discount for premium users', () => {
    expect(calculateDiscount(100, 'premium')).toBe(90);
  });

  it('does not reduce the price for standard users', () => {
    expect(calculateDiscount(100, 'standard')).toBe(100);
  });
});
```

This test is good because it checks behavior directly, uses simple inputs, and makes failures easy to interpret.

---

## Part 4 - Integration Tests: Verifying That Parts Work Together

### What integration testing means

An integration test verifies that multiple parts of the system collaborate correctly.

Examples:

- A form submits data and shows validation errors
- A React page fetches API data and renders the result
- An authenticated route redirects unauthenticated users
- A backend endpoint writes correctly to the database

Integration tests answer a different question from unit tests:

> **Does the system still behave correctly when real pieces interact?**

### Why integration tests matter

Many real bugs do not live inside one function. They happen at the boundary between pieces:

- The component expects one data shape, but the service returns another
- The API accepts invalid input because validation is missing
- The UI loads correctly, but error handling is broken
- The database relation works in theory, but not in the real request flow

Unit tests usually miss those failures. Integration tests are meant to catch them.

---

## Part 5 - Jest, Vitest, and Testing Library

### Jest and Vitest

Jest and Vitest solve similar problems: they run tests, provide assertions, and support mocks and setup utilities.

| Tool | Best known for | Why teams use it |
|------|----------------|------------------|
| **Jest** | Longstanding ecosystem support | Mature, widely documented, common in many projects |
| **Vitest** | Fast modern runner for Vite projects | Excellent developer experience, strong TypeScript and ESM support |

For course purposes, the conceptual ideas matter more than the brand of test runner.

The important questions are:

- Can the tool run tests consistently in CI?
- Is it easy to read failures?
- Does it match the stack your project already uses?

### Testing Library

Testing Library is not mainly about writing more tests. It is about writing **better kinds of tests** for UI.

Its core philosophy is:

> **The more your tests resemble the way users interact with your software, the more confidence they can give you.**

That means:

- Prefer selecting elements by role, label, and text
- Prefer simulating user behavior over calling internal methods
- Avoid testing private implementation details

### Why this philosophy matters

Tests that depend on CSS classes, component internals, or exact DOM structure often break during harmless refactors.

Tests that depend on user-visible behavior usually survive longer and provide better signal.

---

## Part 6 - What Makes a Test Valuable

Not every automated test is a good test.

A valuable test protects the team from a meaningful failure.

### High-value tests usually have these traits

- They check an important behavior, not a trivial detail
- They fail for a clear reason
- They are stable across repeated runs
- They are easy to maintain
- They support refactoring rather than blocking it

### Low-value tests often look like this

- They only verify that a mocked function was called without checking user-visible impact
- They snapshot huge UI trees that nobody reviews carefully
- They duplicate what TypeScript or the framework already guarantees
- They break every time the UI markup changes slightly

### A useful mental model

Ask this question before writing a test:

> **If this behavior broke tomorrow, would I want this test to warn me?**

If the answer is no, the test may not be worth its maintenance cost.

---

## Part 7 - Mocking, Isolation, and Trade-offs

Mocking is useful, but easy to misuse.

### Why mocks exist

Mocks help isolate the unit under test from slow, random, or external dependencies such as:

- Network calls
- Databases
- Current time
- Payment providers
- Email services

This makes unit tests fast and deterministic.

### The danger of over-mocking

If everything is mocked, tests can pass while the real system is broken.

For example:

- A service test mocks the API response incorrectly
- A component test mocks a hook that never behaves that way in production
- A repository test mocks the database so heavily that query mistakes remain invisible

### Practical rule

- Use mocks freely in **unit tests** when isolation is the point
- Use fewer mocks in **integration tests** because interaction is the point

The level of realism should match the goal of the test.

---

## Part 8 - TDD: Red, Green, Refactor

Test-Driven Development is not "write a test sometimes." It is a disciplined loop.

```text
Red      -> Write a failing test
Green    -> Write the smallest code that makes it pass
Refactor -> Improve the design while keeping tests green
```

### Why TDD helps

Done well, TDD can improve design because it forces you to think about:

- What behavior you actually want
- How the code will be used from the outside
- Whether the API of your function or component is too complicated

### What TDD is not

- It is not writing dozens of tests after the code is already finished
- It is not proof that your design is automatically good
- It is not required for every single task

### When TDD is especially useful

- Business rules with many edge cases
- Validation logic
- Parsers and transformations
- Utility functions where expected input/output is clear

### Example TDD cycle

1. Write a test that says a task title cannot be empty.
2. Run it and watch it fail.
3. Add the smallest validation logic.
4. Run tests again and see them pass.
5. Refactor naming or structure without changing behavior.

The value is not ceremony. The value is fast feedback and deliberate design.

---

## Part 9 - Common Testing Mistakes

### Mistake 1: Testing implementation details

If tests care more about internal state than visible behavior, they become brittle.

### Mistake 2: Writing too many broad, slow tests

If every test boots the full application, feedback becomes too slow.

### Mistake 3: Not testing error states

Many student projects test the happy path only. Real software fails in many ways.

Make sure you test:

- Invalid input
- Empty states
- Loading states
- API errors
- Authorization failures

### Mistake 4: Treating coverage as the goal

Coverage metrics can be useful, but they can also be misleading.

> **High coverage does not guarantee meaningful testing.**

A project with 95 percent coverage can still miss important behaviors if the tests are shallow.

### Mistake 5: Leaving tests out of the delivery pipeline

Tests add the most value when they run automatically in CI. If tests only run on your laptop, they do not protect the shared codebase reliably.

---

## Part 10 - Suggested Exercise and Deliverable

### Suggested in-class exercise

Build a small testing suite for one feature of your project that includes:

1. At least two unit tests for pure logic or validation rules
3. One failing test written first using the Red-Green-Refactor cycle
4. Clear assertions for both success and failure cases

Possible examples:

- Login form validation
- Shopping cart totals and checkout rules
- Protected route behavior
- Task creation form with API submission
- Backend endpoint validation and persistence

### Deliverable expectations

A Week 13 deliverable should include:

- At least **5 unit tests**
- Tests that cover both happy paths and failure paths
- A test command that runs locally and in CI
- A short README section explaining what is tested and how to run the suite

### Reflection questions

1. Which bugs are easiest to catch with unit tests in your project?
2. Which behaviors require integration tests to be trustworthy?
3. Where are you currently over-mocking or under-testing?
4. If a teammate changes your code next week, which tests would protect them most?

---

## Recommended Reading and Exploration

- [Testing Library Guiding Principles](https://testing-library.com/docs/guiding-principles/) - Why behavior-focused UI tests matter
- [Vitest Guide](https://vitest.dev/guide/) - Official documentation for modern JavaScript and TypeScript testing
- [Jest Docs](https://jestjs.io/docs/getting-started) - Reference for syntax, assertions, and mocking
- [Kent C. Dodds - Write tests that resemble how users use your software](https://kentcdodds.com/blog/tests) - A practical testing mindset
- [Martin Fowler - Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) - Why the pyramid is a useful strategy
