---
name: war-gaming-plans
description: Use when an implementation plan is written but not yet executed — before dispatching builders — or when asked to "war game", "poke holes in", "stress test", or "anticipate what could go wrong with" a plan or spec. Also use after writing any plan whose execution touches production, migrations, external services, or money.
---

# War-Gaming Plans

Attack your own plan before the codebase does. Two passes — assumption verification and scenario sweep — then **fold every fix back into the plan document**. A war game that ends in a review memo is a failed war game: the executor reads the plan, not the memo.

## Why a plain review isn't enough

A capable reviewer with repo access will verify code-level claims (interfaces, exports, return shapes) on their own. Where reviews reliably fail — observed in baseline testing — is:

1. **Ordering disasters** live between tasks, not inside them (reviews go task-by-task).
2. **Failure-policy decisions** hide inside error handlers as accidents instead of decisions.
3. Findings land as a memo the executor never sees.

The two-pass structure exists to force exactly those.

## Pass 1 — Verify every claim against ground truth

List every assumption the plan makes about the codebase, then verify each by reading the actual code — quote file:line in your finding. Typical claims: function signatures and **return-value field names**, module exports, dependency presence in package.json, migration numbering, middleware behavior, what existing helpers actually do.

- A claim you can't verify is a finding.
- **Mocked tests hide wrong interfaces**: if the plan's tests mock module X, the plan's assumptions about X's real interface are UNTESTED by those tests — verify them here or they ship wrong.
- Also verify the plan's own internal consistency: do tests assert things the design makes impossible? Do "Interfaces: consumes" lists match what the code snippets actually call?

## Pass 2 — Scenario sweep (the categories reviews skip)

Walk EVERY category. "N/A" is an acceptable answer; skipping the question is not.

| Category | The question to force |
|---|---|
| **Deploy/rollout ordering** | For each pair (migration, code) and (PR N, PR N+1): what breaks if they land in the wrong order or one deploys without the other? Auto-deploy-on-merge makes this a merge-button question, not an ops question. |
| **External-dependency failure policy** | For each NEW call to an external system (secrets store, LLM API, cache, third-party API): when it's down, does this fail open or closed? Decide EXPLICITLY and state the consequence of the wrong choice in money, data, or security — "fail open" that silently moves cost onto your bill is a leak, not availability. |
| **Abuse economics** | For each new public or semi-public surface: who pays when it's abused, what does the attacker control (can they rotate around your rate key?), and what's the true backstop vs. the UX guard? |
| **Malformed input from integrators** | For each input a third party constructs (histories, callbacks, webhooks): what does the strictest downstream consumer reject, and do you repair or 4xx? |
| **Mid-flight collisions** | Who else is editing the files this plan touches? What's the rule when their commit lands under you? |
| **Revert path per PR** | If the post-deploy probe fails, is the revert one clean commit? What state (migrations, secrets, IAM) does a revert NOT undo? |
| **Error distinguishability** | Can the caller tell every new failure apart? Two failures with different fixes must not share a code. |

## Fold back — edits, not memos

For each finding: edit the plan at the point of failure (mark it, e.g. "war-game:"), add hard sequencing rules to the plan's global constraints, and append a **failure-modes playbook** (scenario → built-in response) so the executor inherits the reasoning. Findings you decide to accept go in the playbook as "accepted residual" with the named backstop — silent acceptance is indistinguishable from never having asked.

## Discipline

- Every defect claim is either **verified** (file:line quoted) or labeled **scenario/judgment** — never assert a code fact you didn't read.
- Report what you checked that HELD, not just what failed — the executor needs to know which assumptions are load-bearing-and-confirmed.
- Fresh eyes beat author eyes: if the plan's author is you, verify your own citations again; if you can dispatch a subagent reviewer, do both and merge.

## Common mistakes

| Mistake | Fix |
|---|---|
| Reviewing tasks one-by-one only | Ordering bugs live BETWEEN tasks — run the Pass 2 table |
| "Handle errors" treated as done because a catch block exists | The catch block IS the fail-open/closed decision — make it explicit |
| Findings list delivered as a reply | Fold into the plan doc; the memo dies with the conversation |
| Skipping categories that "obviously don't apply" | Write the N/A — the 10 seconds beats the outage |
| Trusting the plan's tests to catch interface errors | Mocks reproduce the plan's assumptions, including the wrong ones |
