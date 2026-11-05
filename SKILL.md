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

Before writing tests, express the specifications in your own words and proceed
without asking for confirmation by default. Ask when the user requests a
confirmation or an ambiguity would materially change the result and cannot be
resolved from the available context.

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
2. Formulate the specifications, clarifying only as described above
3. See if we need to "clean the kitchen before we make dinner". Refer to the "Cleaning the Kitchen" section below.
4. You write just one test (per Canon TDD)
5. Show me the test and ask for approval before continuing
6. Write the application code and review it as described below.
7. Perform the mutation check below, show me the result, and ask for approval before committing. Repeat from step 2 for any remaining specifications.

### Cleaning the Kitchen

Before you write a test, picture the test you're going to write and where you're going to put it. Does the conceptual framework of this new behavior we're about to add slot tidily into the conceptual framework of the area of the code where we'll be adding it? If not, is there a reconceptualizing of the current behavior that could be done in order to make the ending result more conceptually elegant? If such a reconceptualizing is called for, suggest it to the user. If the user approves, abandon the current change, get to a clean working state, and, on a new branch, perform a refactoring. "Clean the kitchen before you make dinner." Then pause and consult the user and we'll begin again.

### Fulfilling Test Specifications

When writing the application code to fulfill a certain specification, write ONLY ENOUGH CODE to make the current test failure go away. Never use "defensive coding". Defensive coding is almost always just speculative coding — code added without justification or feedback.

Use one review agent to check both the test and application code against the
principles in [test-design-review](../test-design-review/SKILL.md) and
[software-design-review](../software-design-review/SKILL.md). In this workflow,
the single agent reviews all principles itself; do not follow those skills'
instructions to spawn an agent per principle. The executing agent evaluates and
addresses the findings.

Use the harness's available delegation and model-selection options to select a
low-cost capable model for the reviewer, such as a Luna- or Haiku-class model
when available. Respect explicit user model choices and harness restrictions;
if model selection or delegation is unavailable, use the available option and
report the limitation rather than claiming a cheap agent was used.

### Mutation Check

After implementation and a passing test run, delegate mutation testing to a
different agent, using the same low-cost model selection guidance. Have it make
small, behavior-changing mutations to the changed code one at a time (for
example, `<=` to `<`), run the relevant tests for each, and restore each mutation
before the next. Work in isolation or coordinate edits so existing changes are
preserved. Confirm the unmutated tests pass again after restoration.

Report killed, surviving, and invalid or equivalent mutations separately. A
surviving meaningful mutation signals a test gap: strengthen the behavior tests
and repeat the relevant check. Do not count compilation errors or unrelated test
failures as a kill, and do not force assertions for equivalent mutations. If
mutation testing or delegation cannot be performed, report what was not checked
and why.

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
