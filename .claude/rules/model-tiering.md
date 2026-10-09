# Model Tiering Rule

## Core Rule

Pick the model for each subagent and workflow agent by the work it does. Lower tiers
are the default for bounded work, and a higher tier validates what they produce. Any
model above the session model needs the user's approval first, with the escalation
criterion it meets stated in the ask.

## The Tiers

Highest to lowest: Fable, Opus, Sonnet, Haiku.

- **Fable**: escalation only. Use it only when an escalation criterion below is met and
  the user has approved it.
- **Opus**: judgment where an error becomes trusted output. Adversarial challengers,
  rules-accuracy checks, auditors who must spot subtle wording or errata changes or copy
  exact `find` strings, and validators of lower-tier work.
- **Sonnet**: bounded work with a checker behind it. Revisers applying a challenger's
  spelled-out fixes, guide or summary writers, routine triage (mostly noise
  classification), research lookups, routine code changes covered by tests.
- **Haiku**: mechanical, low-stakes work only. Listing files, counting, reformatting,
  extracting fields into a fixed shape. Not for exact quoting or page checks against
  source documents.

## At or Below the Session Model: Proceed

Any tier at or below the session model is allowed without asking. Prefer the lowest
tier that can do the job reliably. Do not let every agent inherit the session model by
default.

## Validation Layer

Lower-tier output that will be trusted gets checked by a higher tier before it is
treated as done.

- **Validate** when the output is written to a file that will be committed, quotes or
  cites a source, feeds a decision, or is handed to the user as a finding.
- **Validator**: at least one tier above the producer. Either the session model reviews
  it directly, or a separate validator agent does. Mixing tiers between a writer and its
  checker is a plus: different models catch different mistakes.
- **Skip validation** for mechanical Haiku work whose result is checked by a
  deterministic step anyway (a test, a linter, a count, a schema parse).
- **Opus work on an Opus session**: validate with a separate Opus agent (fresh context)
  or by the session itself. Do not escalate to Fable just to validate.

## Above the Session Model: Ask First

Using a tier above the session model (Fable on an Opus session, or Opus on a Sonnet
session) requires the user's approval before the agent runs. The ask states:

1. Which escalation criterion below is met, and the evidence for it.
2. Which agents would run on the higher tier, and how many.
3. The cheaper alternative and why it falls short.

### Escalation criteria

A higher tier is justified only when at least one of these holds:

- **Repeated failure at the current tier**: the task failed validation twice at the
  session tier, or two session-tier agents reached contradictory conclusions that review
  could not settle.
- **High consequence, no downstream check**: the output will be acted on without a
  feasible later check, and an error is costly or hard to reverse. Examples: legal, tax,
  or financial interpretation that drives a real decision; content published under the
  user's name; a migration or deletion plan for data with no backup.
- **Whole-corpus reasoning**: the task needs one coherent analysis across a large body of
  material, and splitting it across agents would lose the cross-references that make it
  correct. Examples: cross-examining a full research archive, an architecture decision
  spanning several repositories.
- **Final adjudication**: the last-word reviewer on a run where session-tier challengers
  disagree on a point that matters.

### What does not qualify

Volume, speed, "to be safe", routine coding, summarization, research lookups, and
validating work the session tier can already check. None of these justify a higher tier.

## Applying It

- Set `model` on every Agent tool call and every `agent()` call in a workflow script.
- State in one line which tiers a run uses before it starts, for example "Sonnet writers,
  Opus validators, Haiku file listing."
- Do not change models mid-run on a resumable workflow; it invalidates the cache. Apply
  the change to the next run.
- An approved escalation covers the run it was asked for, not later runs.

## Why

A 2026-10-09 SRD import audit in Campaign-Manager ran about 120 agents, all on Opus.
Danny wants cost matched to the work: lower tiers are fine when a higher tier checks
the result, and the most expensive tier is used only when the work demonstrably needs
it and he has agreed.
