---
name: tdd
description: Use when implementing any feature or bugfix to follow test-driven development. Guides the specify-encode-fulfill workflow.
argument-hint: [specification]
disable-model-invocation: false
---

# Test-Driven Development

## Initial Specification

$ARGUMENTS

## The Specify-Encode-Fulfill Loop

I use something called "specify-encode-fulfill":

1. **Specify**: Come up with the specifications for what you want to build
2. **Encode**: Encode those specifications as automated tests (executable specifications)
3. **Fulfill**: Write the code to fulfill the specifications

At a finer grain:

1. Write a list of the specifications within scope of the current TDD session
2. Encode one item in the list as an automated test
3. Change the code *just barely enough* to *make the current test failure go away*. Avoid "speculative coding" — writing more code than necessary to make the current test failure go away risks having code never exercised by any test
4. Optionally refactor, but not before committing the behavior change. Never mix behavior changes with refactoring
5. Until the list is empty, go back to #2

This follows Kent Beck's [Canon TDD](https://tidyfirst.substack.com/p/canon-tdd).

## Clarifying Specifications

Before writing tests, follow this loop:

1. Repeat my specifications back to me in your own words
2. Ask me to confirm your articulation is correct or explain how it's wrong
3. If confirmed, proceed to writing tests; otherwise use my response and go back to step 1

Specifications should take the form: "under scenario A, X happens; under scenario B, Y happens".

For guidance on designing good specifications, see [test-design-review/SKILL.md](../test-design-review/SKILL.md).

## Translating Specifications into Tests

Each scenario should map to a dedicated test case scoped to that scenario. If the specification is "when a test suite run's status is 'passed', its label says 'Passed'", then the test should look like this:

```
describe "#label"
  context "when status is 'passed'"
    it "returns 'Passed'"
      test_suite_run = new TestSuiteRun(status: "passed")
      expect(test_suite_run.label).to eq("Passed")
```

Here's an example of a BAD way to write such a test:

```
describe "#label"
  it "returns the correct value"
    test_suite_run = new TestSuiteRun(status: "passed")
    expect(test_suite_run.label).to eq("Passed")
```

Of course it returns the "correct" value. What else could we ever want from our code? Never assert that a behavior "works correctly" or "works properly" or "handles" a certain scenario. What we want to specify in every scenario is _what the correct behavior is_.

Use your project's test framework and language conventions when writing actual test code — the pseudocode above shows structure and intent, not syntax.

## Workflow

1. I invoke /tdd with a draft specification
2. After back-and-forth, we agree on "final" specifications
3. See if we need to "clean the kitchen before we make dinner". Refer to the "Cleaning the Kitchen" section below.
4. You write just one test (per Canon TDD)
5. Show me the test and ask for approval before continuing
6. Write the application code, show it to me, and ask for approval before committing. Refer to the "Fulfilling Test Specifications" section below.
7. I provide a new specification and we start over from step 2

### Cleaning the Kitchen

Before you write a test, picture the test you're going to write and where you're going to put it. Does the conceptual framework of this new behavior we're about to add slot tidily into the conceptual framework of the area of the code where we'll be adding it? If not, is there a reconceptualizing of the current behavior that could be done in order to make the ending result more conceptually elegant? If such a reconceptualizing is called for, suggest it to the user. If the user approves, abandon the current change, get to a clean working state, and, on a new branch, perform a refactoring. "Clean the kitchen before you make dinner." Then pause and consult the user and we'll begin again.

### Fulfilling Test Specifications

When writing the application code to fulfill a certain specification, write ONLY ENOUGH CODE to make the current test failure go away. Never use "defensive coding". Defensive coding is almost always just speculative coding — code added without justification or feedback. Once you've written the test, invoke a separate subagent with the /test-design-review command to scrutinize your test code. Then invoke another subagent with the /software-design-review command to scrutinize your application code.

### Don't Be Sloppy

This kind of thinking is bad:

> That failure is pre-existing (unrelated to our change). Our new specs pass. Want me to commit and push?

We don't make dinner in a dirty kitchen. If we discover a pre-existing failure, the right move is to pause, stash our changes, fix the pre-existing failure, then resume.

### Don't Be Lazy

This is stupid and bad:

> There's a version mismatch. The tests can't run here. But the changes are straightforward — the logic looks right.

Don't abandon tests immediately upon encountering the slightest difficulty.

### Wrapping Up

Once the specification list is empty and the change is committed, consider whether the
work is done, or whether it left a trail behind. If a real design decision was made along
the way (a library/pattern/architecture choice, a reversal of a prior approach), suggest
`/adr` to record it. If the change affects anything README.md, CLAUDE.md, or a project's
`docs/` folder describes, suggest `/docs-sync`. Skip both suggestions for trivial changes —
they're for when the work leaves the codebase's shape or documented behavior different from
before, not for every commit.
