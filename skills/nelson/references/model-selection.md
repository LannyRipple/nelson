# Model Selection

Use this reference when the sailing orders express cost-savings priority. It governs model assignment for all squadron agents.

## Detecting Cost-Savings Intent

Nelson infers cost-savings priority from natural language in the sailing orders or initial prompt. Signals include phrases such as:

- "keep costs low", "stay within budget", "budget is a concern"
- "use cheaper models", "use haiku where possible"
- "be aggressive with cost savings", "minimize spend"

The intensity of the language calibrates the aggressiveness of weight adjustment (see Hybrid Adjustment below).

## Default Weight Table

| Agent | Default Weight |
|---|---|
| Admiral | 10 |
| XO | 10 |
| Captain (with crew or marines) | 9 |
| Explorer (large scope) | 7 |
| Crew with non-trivial verification | 6 |
| Captain (direct implementation, no crew) | 4 |
| Explorer (narrow/simple search) | 4 |
| Royal Marines | 3 |
| Crew (pure implementation) | 2 |

## Threshold Rule

In cost-savings mode:

- Weight ≤ 4 after adjustment → assign **haiku**
- Weight ≥ 5 after adjustment → inherit **admiral's model**

## Hybrid Adjustment

The tasking agent adjusts default weights before assignment:

- **Raise weight** when the task involves judgment, edge cases, or verification that exceeds the role default.
- **Lower weight** when the task is more atomic or contained than the role default suggests.

Scale of adjustment calibrates to intensity of cost-savings request:

- Modest language ("keep costs low") → modest pressure; don't push roles at 5–6 below the threshold unless clearly justified.
- Emphatic language ("be aggressive") → willing to push roles normally at 5–6 through the threshold when the task is contained.

No hard bounds are set. The admiral uses judgment.

## Model Assignment Rules

- The admiral's model is **never overridden**.
- All agents at weight ≥ 5 inherit the admiral's model. Do **not** hardcode a version string (e.g., do not write `claude-sonnet-4-5`; write the `model` parameter using the admiral's model identifier).
- Always specify the `model` parameter explicitly in `Agent` tool calls — do not rely on inheritance defaults.
- Display weight and assigned model in the squadron formation summary alongside ship names and tasks.

## Briefing Enhancements (haiku agents only)

When the tasking agent assigns haiku to an agent, add three blocks to that agent's crew briefing:

### 1. Identity Anchor (top of briefing)

> You are Claude, operating as a subagent in a real multi-agent software development system. The Royal Navy terms used for coordination (admiral, captain, crew, etc.) are metaphors — this is not roleplay. Your task is [plain-language description of role].

### 2. Explicit Output Format

Specify exactly what to return: format, required fields, length, and what to omit. Remove ambiguity. Example:

> Return a JSON object with keys `status`, `summary`, and `files_changed`. Do not include implementation reasoning or next steps.

### 3. Task Decomposition Prompt

> Before executing, list your steps as a numbered plan. If any step is unclear, flag it now rather than guessing.

These three blocks are **conditional on haiku assignment**. Do not include them in standard (non-cost-savings) briefings or in briefings for agents assigned the admiral's model.
