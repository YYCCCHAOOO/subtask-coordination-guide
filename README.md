# Subtask Coordination Guide

A platform-neutral protocol for decomposing complex goals into bounded work units, coordinating their execution, and synthesizing trustworthy results.

The protocol treats durable task threads, temporary agents, and serial workstreams as interchangeable executors behind the same task-packet and result-packet contracts. The coordinator remains responsible for the final goal, dependencies, write boundaries, conflicts, decisions, and completion.

## Why this protocol exists

Complex work often fails at the handoff points rather than inside the individual tasks. Common failure modes include:

- two executors editing the same file or record;
- downstream work treating an unverified assumption as fact;
- prompts that omit acceptance criteria or forbidden actions;
- long returns that hide blockers and validation failures;
- declaring the parent goal complete because every executor stopped running.

This guide makes responsibilities and handoffs explicit before work begins.

## When to use it

Use this protocol when a goal needs one or more of the following:

- clean context separation;
- independent research, implementation, or review;
- dependency-aware parallel work;
- strict write ownership;
- explicit user-decision gates;
- compact, auditable result handoffs;
- synthesis across multiple domains or repositories.

For a small, linear task with one owner and one output, a simple checklist is usually enough.

## Required coordination specification

Do not dispatch work until these fields are explicit:

1. `final_goal`: one observable end state.
2. `coordinator_scope`: what the coordinator owns and what it must not execute.
3. `subtasks`: bounded units with responsibility, exclusions, inputs, outputs, acceptance checks, dependencies, and write scope.
4. `ordering`: dependency waves, or a clear reason tasks can run concurrently.
5. `conflict_policy`: rules for shared files, shared identifiers, contradictory findings, and decision gates.
6. `executor_profile`: configured model, reasoning level, tools, and any justified exception.
7. `return_contract`: the exact compact result the coordinator needs to continue.

## Choosing an executor

Choose based on responsibility and lifecycle, not on agent technology:

- **Durable task thread**: recurring work, a large clean context, or user-visible continuation.
- **Temporary agent**: a bounded implementation, audit, search, or independent critique within the current goal.
- **Serial workstream**: work where concurrency adds conflict risk or no separate executor is available.

Do not create a new task merely to avoid reasoning. Create it only when the responsibility and return boundary are meaningful.

## Workflow

### 1. Normalize the goal

Rewrite the request as:

- one final goal;
- explicit non-goals;
- measurable acceptance criteria;
- decision authority;
- stable identifiers and canonical input locations.

### 2. Design bounded subtasks

Give every subtask:

- one accountable responsibility;
- explicit excluded scope;
- only the context needed for that responsibility;
- disjoint write ownership or a read-only role;
- measurable outputs and validation;
- dependencies and a return target.

If two subtasks can modify the same file, registry, database row, or entity identifier, do not run them concurrently. Assign one writer and make the other read-only or dependent.

### 3. Build optimized prompts

Put the final goal first, followed by:

1. the subtask's sole responsibility;
2. minimum background and canonical inputs;
3. required rules or policy files;
4. allowed writes and forbidden actions;
5. dependencies and the current wave;
6. acceptance checks;
7. executor profile and justified exceptions;
8. the exact result schema and return target.

Prefer stable paths and identifiers over copied source content. Require each executor to read its own domain rules rather than pasting an entire project into every prompt.

### 4. Dispatch by dependency waves

Dispatch only subtasks whose dependencies are satisfied. Run a wave concurrently only when write scopes and decision authority are disjoint. Keep later waves pending instead of asking them to guess missing upstream results.

### 5. Review returns

Accept only these result states:

- `done`
- `partial`
- `blocked`
- `needs_decision`

Review evidence, changed assets, validation results, boundary compliance, unresolved risks, and the requested next action.

Do not silently merge contradictions. Record competing claims, their evidence, affected downstream tasks, and the coordinator's resolution or user-decision request.

### 6. Continue or close

Update dependency state and send only newly required context to the next wave. Close the final goal only when all required outputs pass acceptance checks and remaining risks are explicit.

## Conflict locks

| Lock | Rule |
| --- | --- |
| `write_lock` | Only one subtask may own a mutable target at a time. |
| `id_lock` | Only one authority may mint or rename identifiers in a namespace. |
| `decision_lock` | An executor may recommend but may not resolve a coordinator or user decision. |
| `evidence_lock` | A candidate, inference, or unverified path may not be upgraded into a fact. |
| `sequence_lock` | Downstream work may prepare structure but may not apply changes before its dependency is complete. |

## Task-packet contract

Use concise values. Point to canonical inputs rather than copying large source material.

```json
{
  "contract_version": "subtask.packet.v1",
  "coordination_id": "COORD-YYYYMMDD-NNN",
  "subtask_id": "SUB-YYYYMMDD-NNN",
  "title": "bounded action",
  "final_goal": "observable parent outcome",
  "responsibility": "the only result this unit owns",
  "excluded_scope": ["work owned elsewhere"],
  "background": "minimum local context",
  "inputs": [
    {
      "id": "stable-input-id",
      "location": "canonical/input/location",
      "state": "verified"
    }
  ],
  "required_rules": [
    "repository instructions, policy file, or skill reference"
  ],
  "allowed_writes": ["exact target or read_only"],
  "forbidden_actions": ["cross-scope action"],
  "dependencies": [
    {
      "subtask_id": "SUB-YYYYMMDD-NNN",
      "required_state": "done"
    }
  ],
  "acceptance_checks": ["measurable check"],
  "executor_profile": {
    "model": "configured-model",
    "reasoning": "configured-reasoning",
    "override_reason": ""
  },
  "decision_authority": "executor|coordinator|user",
  "return_target": "parent task or coordination ledger",
  "return_contract": "subtask.result.v1"
}
```

## Result-packet contract

```json
{
  "contract_version": "subtask.result.v1",
  "coordination_id": "COORD-YYYYMMDD-NNN",
  "subtask_id": "SUB-YYYYMMDD-NNN",
  "status": "done|partial|blocked|needs_decision",
  "summary": "what changed or was established",
  "outputs": [
    {
      "role": "artifact role",
      "location": "canonical/output/location",
      "validation_state": "passed"
    }
  ],
  "evidence": [
    {
      "claim": "bounded finding",
      "source": "artifact or source reference",
      "confidence": "high|medium|low"
    }
  ],
  "validations": [
    {
      "check": "command or audit",
      "state": "passed|failed|not_run"
    }
  ],
  "boundary_check": {
    "within_scope": true,
    "unexpected_writes": []
  },
  "conflicts": [
    {
      "with": "subtask or rule",
      "issue": "conflict",
      "impact": "downstream effect"
    }
  ],
  "blockers": [
    {
      "reason": "blocker",
      "required_from": "coordinator|user|dependency"
    }
  ],
  "next_recommendation": "one concrete next action"
}
```

## Boundary rules

- The coordinator owns orchestration and synthesis. Specialist implementation remains delegated unless it is explicitly reassigned.
- A subtask may not widen the final goal, create sibling tasks, or modify another subtask's scope without coordinator approval.
- A subtask may not treat a dependency assumption as fact.
- A durable task may coordinate smaller subtasks inside its own domain, but it must return one consolidated result to its parent.
- Keep raw logs and long notes with the executor or canonical project. Return compact findings and original artifact locations.
- Escalate missing authority, high-risk writes, external actions, and scope-changing choices.

## Privacy and publication checklist

Before sharing a task packet, result packet, prompt, screenshot, or repository:

- replace personal names and internal system names with role labels;
- replace absolute local paths with canonical placeholders;
- remove credentials, tokens, one-time codes, account identifiers, and private URLs;
- parameterize provider-specific models and reasoning settings;
- remove private project names, datasets, stable IDs, and unpublished findings;
- inspect the staged diff, not only the working copy;
- scan the full publication scope for secrets and environment-specific markers;
- confirm that every referenced file is included or publicly accessible;
- ensure the repository visibility matches the intended audience.

## Completion standard

The coordinator's final report should state:

- final-goal status;
- completed, partial, and blocked subtasks;
- accepted outputs and their canonical locations;
- validation evidence;
- unresolved conflicts and decision owners;
- boundary compliance;
- the next recommended wave.

Never declare the parent goal complete merely because all executors finished their turns.

## Compact example

Suppose a release needs implementation, documentation, and security review:

1. The coordinator defines the release outcome and owns final integration.
2. The implementation subtask exclusively owns the source-code write scope.
3. The documentation subtask may write only the guide and must depend on the finalized interface.
4. The security subtask is read-only and can run alongside documentation review after implementation.
5. The coordinator compares all result packets, resolves conflicts, runs integration checks, and closes the release only when the acceptance criteria pass.

The value of the protocol is not the number of agents. It is the clarity of responsibility, evidence, sequencing, and handoff.
