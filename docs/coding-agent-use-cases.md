# Claude Fable 5.1 Coding Agent Use Cases

Use Claude Fable 5.1 when the coding task benefits from long-context review, high-level planning, hard-to-reproduce debugging, or an agent that can work for hours with verification loops. Route routine implementation and cheap iteration to lower-cost models when the plan is already stable.

## Best-fit workflows

| Workflow | Why Fable 5.1 fits | Suggested output |
|---|---|---|
| Architecture review | It can connect interfaces and trade-offs across a large codebase before edits begin. | Decision-complete implementation plan |
| Large PR review | It can inspect cross-module correctness risks and follow evidence into dependencies. | Findings ordered by severity with file references |
| Agent relay planning | It can define handoff boundaries for cheaper execution models. | Task graph plus acceptance criteria |
| Debugging triage | It can reason from logs, diffs, traces, core dumps, and reproduction notes together. | Root-cause hypothesis and verification checklist |
| Migration planning | It can map compatibility, rollout, fallback, and test coverage before edits. | Stepwise migration plan |
| Long-running implementation | It can work through multi-file features, refactors, and visual verification over extended sessions. | Tested changes plus a concise evidence report |

## First prompt template

```text
You are reviewing a production codebase change.

Goal:
<what needs to ship>

Context:
<repo/module summary, constraints, interfaces, and known risks>

Artifacts:
<paste relevant diffs, logs, stack traces, screenshots, or design notes>

Operating rules:
- Inspect the repository and identify the real root cause before editing.
- Keep independent reads and checks parallel where the tools allow it.
- Run focused tests after each meaningful change and report evidence.
- Keep the conversation history append-only; do not rewrite earlier turns.
- Do not add unrelated fixes or speculative features.

Output:
1. Findings that could break production, ordered by severity.
2. The smallest implementation plan that resolves them.
3. Tests that prove the change works.
4. Any assumptions that must be verified before shipping.
```

## Fable 5.1 migration checklist

When moving an existing Claude Fable 5 coding agent to Fable 5.1:

- Change the model ID to `claude-fable-5-1`.
- Remove `tool_choice` values of `any` and `tool`; use `auto` with strict tool schemas or structured outputs.
- Treat adaptive thinking as always on. Do not send a legacy thinking budget or disable thinking.
- Append assistant responses, including thinking blocks, exactly as returned.
- Keep the `system`, `tools`, and earlier messages unchanged once a thinking block is in the conversation.
- Re-run effort sweeps across `low`, `medium`, `high`, `xhigh`, and `max` against your own task set.
- Check refusal handling before reading content; a refusal can be HTTP 200 with `stop_reason: "refusal"`.
- Configure fallback handling for the documented Opus 4.8 and Opus 5 fallback targets.

## Useful Fable 5.1 prompt nudges

For implied independent reads in a coding loop:

```text
First privately list what you need next; then request every item that doesn't depend on another's result in this one response.
```

For user-facing status in long runs:

```text
Before you start, say in one line what you are about to do. Give brief updates when you finish a meaningful phase, and close with what you found, what you changed, what you verified, and what remains.
```

For writing that becomes too dense:

```text
Please remove mannered prose. Prefer direct literal language, short paragraphs, and concrete nouns whenever they communicate the idea accurately.
```

## API example

```python
import anthropic

client = anthropic.Anthropic()
response = client.messages.create(
    model="claude-fable-5-1",
    max_tokens=4096,
    output_config={"effort": "high"},
    messages=[
        {
            "role": "user",
            "content": "Review this repository migration plan and list the highest-risk gaps.",
        }
    ],
)

if response.stop_reason == "refusal":
    # Inspect stop_details and invoke your configured fallback path.
    raise RuntimeError("Fable 5.1 declined the request")

print(next(block.text for block in response.content if block.type == "text"))
```

## Official references

- [What's new in Claude Fable 5.1](https://platform.claude.com/docs/en/models/fable-5-1/whats-new-fable-5-1)
- [Prompting Claude Fable 5.1](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1)
- [Migration guide](https://platform.claude.com/docs/en/models/fable-5-1/migration-guide)
- [Anthropic launch announcement](https://www.anthropic.com/claude-fable-and-mythos-5-1)

## MuAPI status

MuAPI support for Claude Fable 5.1 is planned. Add the verified MuAPI model route and pricing here when the model is live in the catalog; until then, use the official Claude API example above rather than guessing an endpoint.
