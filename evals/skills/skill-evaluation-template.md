# Skill Evaluation: [Skill name]

> Save completed records as `YYYY-MM-DD-<skill-id>-evaluation.md`.
> `config/skill-policy.yaml` is the source of truth for dimensions, weights, thresholds, and gates. If this template differs from that policy, follow the policy.

## Evaluation Metadata

- Skill ID: `[lowercase-kebab-case]`
- Skill name: `[name]`
- Skill version: `[version]`
- Source: `[source or URL]`
- Scope: `[global | project]`
- Current lifecycle stage: `[trial | evaluate | install | distill | maintain]`
- Owner: `[owner]`
- Activation: `[activation method]`
- Evaluator: `[name]`
- Evaluation date: `[YYYY-MM-DD]`
- Related task or requirement: `[reference]`

## Trial Evidence

| Case ID | Goal | Input or context | Expected result | Actual result | Verification method | Evidence reference | Result |
| --- | --- | --- | --- | --- | --- | --- | --- |
| CASE-01 | [goal] | [input or context] | [expected result] | [actual result] | [command, review, or observation] | [repository path, task record, or stable URL] | [pass | fail] |

Each score or decision must be supported by reviewable evidence. Do not include passwords, Tokens, private keys, or sensitive data.

## Scorecard

| Dimension | Weight snapshot | Score (1–5) | Rationale | Evidence reference |
| --- | ---: | ---: | --- | --- |
| relevance | 0.20 | [score] | [reason] | [evidence] |
| reliability | 0.20 | [score] | [reason] | [evidence] |
| verifiability | 0.15 | [score] | [reason] | [evidence] |
| interoperability | 0.15 | [score] | [reason] | [evidence] |
| maintainability | 0.10 | [score] | [reason] | [evidence] |
| safety | 0.15 | [score] | [reason] | [evidence] |
| efficiency | 0.05 | [score] | [reason] | [evidence] |

Weighted total formula: `sum(dimension score * dimension weight)`

- Weighted total: `[1.00–5.00]`
- Adoption threshold snapshot: `4.00`
- Safety gate: a `safety` score below `4` blocks `install` regardless of weighted total.

## Gate Checks

- [ ] At least one representative trial case is complete.
- [ ] Every material score has an evidence reference.
- [ ] Permission, privacy, data, and irreversible-operation risks were reviewed.
- [ ] Compatibility with Codex, AIOS repository rules, and related Skills was reviewed.
- [ ] Source, version, scope, owner, and activation are known, or missing values are documented below.
- [ ] The safety installation gate passes, or the block is explicitly recorded below.

Unmet gates: `[none, or list each unmet gate and its impact]`

## Decision

- Decision: `[install | distill | maintain | continue-evaluation | do-not-adopt]`
- Decision rationale: `[evidence-backed rationale]`
- Rejection reason: `[required for do-not-adopt; otherwise not applicable]`
- Known limitations: `[limitations or none]`
- Required follow-up actions: `[actions or none]`

`continue-evaluation` and `do-not-adopt` are evaluation outcomes, not lifecycle stages. Select exactly one Decision value.

## Review Schedule

- Review date: `[required for install, distill, or maintain; otherwise not applicable]`
- Next evaluation date: `[required for continue-evaluation; otherwise not applicable]`
- Review trigger: `[version, permission, repeated-failure, maintenance, or other trigger]`

Dates use `YYYY-MM-DD`. `do-not-adopt` does not require a review date, but it does require a rejection reason.
