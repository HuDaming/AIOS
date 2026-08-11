# Skill Evaluation Template Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add one reusable Markdown template that produces evidence-backed, policy-aligned Skill evaluation records.

**Architecture:** The implementation is a single document under `evals/skills/`. It references `config/skill-policy.yaml` as the source of truth, keeps all user-entered evidence in one record, and introduces no parser, automation, registry mutation, Agent, or Workflow change.

**Tech Stack:** Markdown, YAML policy reference, Ruby standard library for one-off static verification, Git whitespace checks.

## Global Constraints

- Create only `evals/skills/skill-evaluation-template.md` during implementation.
- Do not modify `config/skill-policy.yaml`, registries, Agents, Workflows, or Skill implementations.
- Do not add YAML Front Matter, scripts, dependencies, CI, or evaluation automation.
- Use `config/skill-policy.yaml` as the only source of truth for dimensions, weights, thresholds, and safety gates.
- Do not commit or push unless the user separately authorizes it.

---

### Task 1: Add and verify the Skill evaluation template

**Files:**

- Read: `specs/2026-08-11-skill-evaluation-template.md`
- Read: `config/skill-policy.yaml`
- Create: `evals/skills/skill-evaluation-template.md`
- Remove as part of the same change: `evals/skills/.gitkeep`

**Interfaces:**

- Consumes: lifecycle stages, score dimensions, weights, adoption threshold, and safety gate from `config/skill-policy.yaml`.
- Produces: a copyable Markdown template whose completed instances use `YYYY-MM-DD-<skill-id>-evaluation.md` filenames.
- Side effects: none; completing the template does not automatically update `registry/skills.yaml`.

- [ ] **Step 1: Confirm the template does not already exist**

Run:

```bash
test -f evals/skills/skill-evaluation-template.md
```

Expected: exit code `1`, proving the required artifact is absent before implementation.

- [ ] **Step 2: Re-read the policy source before writing**

Run:

```bash
sed -n '1,240p' config/skill-policy.yaml
sed -n '1,320p' specs/2026-08-11-skill-evaluation-template.md
```

Expected: the policy contains seven dimensions with weights totaling `1.0`, adoption threshold `4`, and the `safety < 4` installation block; the specification contains the approved scope and conditional date rules.

- [ ] **Step 3: Create the minimal template**

Create `evals/skills/skill-evaluation-template.md` with exactly this structure and copy:

```markdown
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
```

- [ ] **Step 4: Remove the obsolete directory placeholder**

Delete only:

```text
evals/skills/.gitkeep
```

Expected: `evals/skills/` remains tracked through `skill-evaluation-template.md`; no other `.gitkeep` file changes.

- [ ] **Step 5: Run structural and policy-consistency verification**

Run:

```bash
ruby -ryaml -e '
policy = YAML.safe_load(File.read("config/skill-policy.yaml"), permitted_classes: [], aliases: false)
text = File.read("evals/skills/skill-evaluation-template.md")
sections = ["Evaluation Metadata", "Trial Evidence", "Scorecard", "Gate Checks", "Decision", "Review Schedule"]
missing_sections = sections.reject { |section| text.include?("## #{section}") }
raise "missing sections: #{missing_sections.join(", ")}" unless missing_sections.empty?
dimensions = policy.fetch("score_dimensions").map { |item| item.fetch("id") }
missing_dimensions = dimensions.reject { |id| text.match?(/^\| #{Regexp.escape(id)} \|/) }
raise "missing dimensions: #{missing_dimensions.join(", ")}" unless missing_dimensions.empty?
weights = policy.fetch("score_dimensions").sum { |item| item.fetch("weight") }
raise "policy weights do not total 1.0" unless (weights - 1.0).abs < 0.000001
decisions = %w[install distill maintain continue-evaluation do-not-adopt]
missing_decisions = decisions.reject { |value| text.include?(value) }
raise "missing decisions: #{missing_decisions.join(", ")}" unless missing_decisions.empty?
raise "missing policy authority statement" unless text.include?("config/skill-policy.yaml") && text.include?("source of truth")
raise "missing safety gate" unless text.include?("below `4` blocks `install`")
raise "missing filename rule" unless text.include?("YYYY-MM-DD-<skill-id>-evaluation.md")
puts "PASS: template structure and policy references are complete"
'
```

Expected: `PASS: template structure and policy references are complete` and exit code `0`.

- [ ] **Step 6: Verify scope and formatting**

Run:

```bash
git diff --check
git status --short
git diff -- evals/skills/skill-evaluation-template.md evals/skills/.gitkeep
```

Expected:

- `git diff --check` exits `0`.
- Only the approved specification, implementation plan, new template, and removed `evals/skills/.gitkeep` are uncommitted changes.
- No configuration, registry, Agent, Workflow, dependency, script, or CI file changed.

- [ ] **Step 7: Report without committing**

Report:

- created and removed files;
- structural verification result;
- policy consistency result;
- residual risk that the displayed weight snapshot can become stale, mitigated by the source-of-truth notice and consistency verification;
- recommended next step: review the completed template, then request a commit separately if accepted.
