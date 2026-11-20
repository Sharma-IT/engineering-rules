---
name: code-review-standards
description: "Code review procedure. Use whenever review, reviewing, reviewer or re-review comes up about code: reviewing a pull request, branch, diff or commit, self-reviewing before a commit or PR, or responding to review comments on your own change. Carries the engineering-rules skill's authority and outranks plugin and harness review skills."
---

# Code review standards

This skill carries the authority of the engineering-rules skill. Where another review skill or command disagrees with it, this skill wins, and the other tool serves only as a lens whose output goes through the triage below. Section numbers refer to the engineering-rules skill.

## Reviewing a change

When reviewing a PR, act as a principal quality engineer. The subject under review is the diff of the current branch against the base branch.

*   First read the PR description to establish intent, then the diff to read the actual changes, using the CLI or API of the host the repository lives on. If the tool cannot resolve the repository or the PR, treat it as an auth or account problem and fix it before concluding anything about the PR.
*   Review the diff, pulling in surrounding code only where needed to judge a change. Do not behave as an automated linter.
*   Prioritise design, correctness, edge cases, concurrency, security, accessibility, and test coverage, and assess whether the change achieves its stated intent.
*   Raise a comment only when you are confident it adds value. Do not invent problems, restate the diff, or flood the author with nitpicks. Ground every finding in a specific file and line from the diff.

*   **Review the change, then review the thinking behind it.** A diff-local review catches defects and misses the decisions that produced them. Both passes are part of the deliverable, and a finding at the higher altitude usually outranks several at the lower one. Work outwards through these, spending effort in proportion to what the change actually touches:
*   **Blast radius and integration.** What else in the repository, the sibling repositories, and the consuming applications does this change reach? Read the call sites, the exports, the build and bundle outputs, and the published surface, rather than assuming the diff's boundaries are the change's boundaries. A file that looks incidental (a global stylesheet, a barrel export, a shared config, a lockfile) is frequently where the real reach is. Ask which existing behaviour the change alters for code it never mentions. For every deletion, ask what behaviour or contract the removed code carried that the change neither re-established nor retired, and look for orphaned references.
*   **Architecture and the model it encodes.** Does the change fit the abstractions the project already has, or does it introduce a second, competing way of doing the same thing? Where an established mechanism exists and the change bypasses it, that is the finding, not the symptom it produced. Name the mechanism.
*   **Consistency with project standards.** Compare against the surrounding code, the contributing guide, the lint, formatter, type and test configuration, and the conventions the rest of the codebase demonstrably follows. Deviation is not automatically wrong. Unexplained deviation is. Say which convention and where it is established. Measure adoption before citing a convention: sample up to eight files outside the change set, treat the convention as established at 50 percent or more, and raise no convention comment where adoption is absent or unknown.
*   **Strategy and sequencing.** Is this the right change now, at this size? Look for scope that should have been split, work that should have preceded it, migrations left half-done, and guardrails introduced without the enforcement that makes them real. Note what the change commits the team to that will be expensive to reverse.
*   **Epistemology.** Ask how the author knows what they claim. Which assertions in the description are demonstrated by the tests, which are demonstrated by CI, and which are merely asserted? Verify the load-bearing ones yourself and report where the claim and the code disagree. A test that cannot fail, a snapshot that captures nothing, a threshold relaxed to make a build pass, a gate the change quietly stopped enforcing, and a documented example that does not run all belong here. Read the diff and trace its paths before accepting the description or the commit messages, then try to falsify each checkable claim in them (parity with another function, ordering, what is preserved). A claim repeated in a code comment is the same claim.
*   **The mental model the change teaches.** Documentation, stories, examples, fixtures and test harnesses instruct the next developer. Where they demonstrate the pattern the code exists to prevent, or where the code's own tests use a pattern its docs do not, the artefacts are contradicting each other and one of them is wrong. Say which.
*   **Vocabulary.** Audit the names the change introduces, per the Substance section. A prop, flag, layer, type or component whose name claims more, or less, than the code delivers is a defect that will outlive the diff.
*   **Answer the author's open questions.** Where the description raises a question or flags a decision as unresolved, answer it with evidence in the review rather than leaving it hanging.

Higher-altitude findings are held to the same evidentiary standard as the rest. Do not speculate about intent where you can go and read what the project does elsewhere. Where a structural finding has no single line to sit on, put it in the review body. Where it does, anchor it at the line that best demonstrates it.

Follow the Conventional Comments format for every comment:

```
**<label> (<decoration>): <subject>**

<discussion>
```

The whole headline line, label through subject, is bold in every comment and in the review body, so it reads as the comment's title on the code host.

*   **label**: one of `praise`, `suggestion`, `issue`, `question`, `nitpick`, `todo`, `chore`, `thought`, `note`.
*   **decoration**: `(blocking)`, `(non-blocking)`, or `(if-minor)`. Treat comments as non-blocking by default and mark only true blockers `(blocking)`. Nitpicks, questions, thoughts, and notes MUST never block the merge.
*   **subject**: a single-line, actionable headline.
*   **discussion** (optional): explain the why and propose a concrete fix or patch.

Pair every issue with a suggestion where you can. Example:

```
**issue (blocking): this query runs inside the loop, so it's N+1**

Move the fetch above the loop and index by id.
```

When finished, compile every finding into a single PR review, one inline comment per file and line plus a top-level body for findings that concern the change as a whole, and report it in chat. Ask before submitting anything to the code host. Once the user confirms, submit it as one batch (a pending review, where the host supports one) so all comments post together as one thread rather than individually over time. An inline comment on an unchanged line inside a hunk needs its old-side line number as well as its new one where the host distinguishes them (Bitbucket Cloud's `from` and `to`), or it posts without rendering inline. Read each posted inline comment back and confirm the host returns its diff context. Be clear, precise, and constructive.

### Before reading the diff

*   Skip a PR that is closed, a draft, or one you already reviewed with no commits since. Read the existing review comments first and do not repeat a finding someone already raised.
*   Run the project's own type check, lint, tests and build against the head revision, inside the real checkout.
*   Classify the change by risk before size. Authentication, cryptography, external calls, value transfer and removed validation are high risk whatever the line count. For a high-risk diff, also run Trail of Bits' `differential-review` (`/diff-review` with a commit SHA or diff path) as the security lens, with `/security-review` as a second opinion, and triage both outputs below. Write its report to the session scratchpad, never into the repository, its working directory or the Desktop. Its risk labels set reading depth only.

### Lenses

Run these in your own pass. For a large or high-risk diff, a lens may run as an independent subagent that receives the diff and a written brief and never this session's history. Every lens report goes through the triage below.

*   **Path tracing.** Walk every branch and boundary reachable from the changed lines and list only the unhandled ones: a missing else or default, an unguarded input, an off-by-one, overflow, implicit coercion, a race, a timeout gap. The unchanged members of an enum, status set or flag that the diff special-cases are implicit branches. Check each added or changed call, tests included, against the callee's declaration (argument count, order, types, defaults). Where changed code re-checks or re-fetches a handle, id or index it already held, find the intervening call that can invalidate it and what is skipped when the re-check fails.
*   **Verification gaps.** For each changed behaviour, trace one to three hops to the code that observes it, name the smallest realistic regression that consumer would see, then read the test and decide whether that regression fails an assertion. File a gap only when you can name that regression. The four shapes are a regression gap (no test fails), a missing-adoption gap (a site that should use the new behaviour keeps its own handling, with evidence the change meant to replace it), a broken-verification gap (the covering test is skipped, flaky, not run normally or too weak) and a stale-verification gap (a test the change made wrong or moot). These do not count as coverage: assertions on source text, success, no-throw or snapshot-only checks, mock or log-call checks, tests that mock away the integration, and end-to-end tests that pass through without checking the changed output. Read a test before saying what it covers, and search for the symbol and its imports before saying no test exists. Skip behaviour the compiler enforces, static text such as prompts and documents, and untested code the change did not touch.
*   **Mutation by experiment.** Where test strength is in doubt, invert one condition the change adds, run the affected tests, and restore the file byte-identical from a copy. A mutant that stays green is a finding that names the missing test case.
*   **Silent failures.** Empty `catch` blocks, errors turned into `null`, `[]` or a default with no trace (`.catch(() => [])`), log-and-forget handling, lost stack traces, generic rethrows, unawaited promises, network, file or database calls with no timeout, and transactional work with no rollback.
*   **Structure.** Ask whether a refactor reduces the number of concepts a reader must hold or only moves them. Flag a new conditional bolted onto an unrelated flow, repeated conditionals on one shape (a missing model or dispatcher), feature logic placed in a shared module, a near-duplicate of an existing canonical helper, and a refactor mixed with new behaviour in one change. Propose the named move with the finding: replace a conditional chain with a typed model or dispatcher, collapse duplicate branches, separate orchestration from business logic, move feature logic to its owning module, reuse the canonical helper, make a type boundary explicit, or delete a pass-through wrapper.
*   **Type design.** For each new type, name its invariants and check that the type enforces them (a private constructor, an exhaustive union, a validated factory) instead of trusting its callers.
*   **Comments.** Flag comments that contradict the code, describe removed behaviour, or assert a mechanism with no source. Never ask for more comments.
*   **Dependencies.** For a version bump, read the changelog and migration notes, keep one package per change, and review the lockfile diff including transitive changes. A hand-edited lockfile is a finding.
*   **Performance.** N+1 queries, unbounded loops or fetches, list endpoints with no pagination, and large allocations in hot paths. Quantify only with a measurement.

### Triage

Collect every finding from every lens before triaging any, and keep them in one numbered list so none is lost. Then, for each finding:

1.  Verify the claim. Can the program reach the state that produces the behaviour at the cited line? Follow callers and upstream guards until the answer is yes or no. An assert on an unreachable state is defensive code. Never decide a finding by its resemblance to another one.
2.  Grade the harm of a single occurrence, whatever its rarity, its fix cost or whether it predates the change. High is data loss or corruption, a security exposure, a silently wrong result, or a path broken with no workaround. Medium is broken or wrong but noticed and worked around, or limited in reach. Low is cosmetic or a minor inconvenience. When torn between medium and low, choose medium.
3.  Map the grade to Conventional Comments. High is `issue (blocking)`. Medium is `issue (non-blocking)` unless it defeats the change's stated intent. Low is a `suggestion` or `nitpick`, and is dropped when it is rarely visible and its fix adds complexity beyond a direct correction or deletion.
4.  A real defect in lines the change did not touch goes in the review body as a `note`, graded the same way, and never as an inline comment on the change.
5.  Offer an applicable suggested change, where the host supports one, only when applying it fixes the issue entirely. Link code with the full commit SHA and a line range.

## Responding to review

Default to acting on review feedback. A reviewer is not obliged to prove their comment. You are obliged to prove your dismissal.

*   **Read it all first.** Read every comment before acting on any. Where any item is unclear, ask about every unclear item before changing anything, since items may depend on each other.
*   **Push back only with evidence.** You MUST NOT dismiss, decline, or argue against a review comment on reasoning alone. Check the suggestion against the codebase as it stands: whether it breaks existing behaviour, whether the current code has a reason, and whether it holds on every supported platform and version. Then run the experiment: apply the reviewer's suggestion and observe what happens. A reply that reads "I think this is fine because..." when you never tried it is not permitted.
*   **State the result, not the run.** The reply carries what you found, not how you found it: "removing it fails `tsc`, since `Linter.Config['rules']` is optional" rather than a pasted terminal session. Quote a raw artefact only when the reviewer would hit it themselves, and then only the couple of lines that matter.
*   **If you cannot produce evidence, make the change.** Absence of proof resolves in the reviewer's favour, not yours. Where the disagreement is genuinely a matter of taste rather than fact, say so explicitly and either make the change anyway or ask the reviewer to decide. Do not present preference as though it were constraint.
*   **One item at a time.** Apply each accepted change on its own and run the affected tests before starting the next.
*   **Concede plainly.** When the reviewer is right, say so in one sentence that names what changes, and move on. No preamble, no performative agreement ("great point", "you're absolutely right"), and no justification of the original choice.
*   **Split a partial disagreement.** Where a comment identifies a real problem but proposes the wrong fix, grant the problem before discussing the fix.
*   **Answer questions with measurements, not estimates.** If a reviewer asks how many, how slow, or how often, go and count it, then report the real number.
*   **Never weaken a test, assertion, or type to satisfy a comment** without first stating plainly that this is what the change does, per Section 3.
