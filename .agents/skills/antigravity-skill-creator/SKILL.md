---
name: antigravity-skill-creator
description: Creates, updates, and validates reusable Google Antigravity skills. Use when asked to turn a repeatable task into a skill, author a new SKILL.md, or improve an existing skill's instructions and supporting resources.
---

# Create Antigravity skills

## Establish the task

Identify the capability being packaged, requests that should activate it, required inputs, expected outputs, and how success can be checked. Infer routine choices from the request. Ask only when missing information would materially change the skill's scope or behavior.

For an update, read the existing skill and its referenced resources before editing. Preserve useful behavior and unrelated work. Check for an existing skill with the same purpose before creating another one.

Use the user's requirements and authoritative domain guidance to define quality. Existing repository skills can reveal packaging constraints, but are not automatically good examples to copy. Distinguish domain requirements from optional recommendations and repository conventions.

## Use the Antigravity format

Check [Google's official skills documentation](https://antigravity.google/docs/skills/) when creating or changing structure. If it is unavailable, use the baseline below and disclose that current compatibility could not be verified.

- In this repository, use `.agents/skills/<skill-name>/SKILL.md`.
- Give the folder a unique lowercase name with hyphens between words. Include a matching `name` in YAML frontmatter; this repository requires it even though Google makes it optional.
- Include a required `description` explaining the capability and when to use it, written in third person.
- Put Markdown instructions after the frontmatter.
- Add optional `scripts/` for helpers, `examples/` for reference implementations, and `resources/` for templates or supporting material only when useful.

Start with this shape, replacing the illustrative content with the actual task:

```markdown
---
name: summarize-build-failures
description: Explains build failures from compiler logs and identifies actionable next steps. Use when diagnosing a failed build from supplied logs.
---

# Summarize build failures

## Workflow

Identify the first causal error, explain the affected dependency or source location, and distinguish follow-on failures from the likely cause.

## Verification

Tie each recommendation to evidence in the supplied logs. State when additional information is needed to identify the cause.
```

Do not add another agent platform's metadata, installation paths, or tool names unless cross-platform support was requested. The created skill must work without access to the environment that authored it.

## Write instructions that change decisions

Write the shortest instructions that make the task repeatable and reliable. Assume the consuming agent understands general programming, research, and file operations.

- Define a concrete outcome and the evidence needed to establish it.
- Explain the important decisions: which approach applies, what inputs are necessary, and when an alternative is appropriate.
- Describe operational steps in an order the agent can execute. Include exact commands only where they improve reliability; document prerequisites and how task-specific arguments are obtained.
- Address likely failure modes with a recovery step or clear stopping condition. Avoid unlimited retries and unsupported success claims.
- Preserve the requested scope. Creating a skill does not authorize installing it globally, modifying unrelated settings, or publishing its output. Follow explicit authorization already present in the conversation.
- Keep version-sensitive claims linked to official domain documentation. Check compatibility instead of blindly recommending the newest dependency.
- Remove generic reminders, repeated instructions, arbitrary process gates, and rules that do not affect the task.

Do not embed secrets, personal paths, session IDs, assumed credentials, or dependencies on private authoring tools. Describe tools by their capabilities unless a particular product is essential to the skill.

## Add resources selectively

Keep the essential workflow in `SKILL.md`. Move substantial conditional detail into a named file under `resources/`, and link it where the agent should read it. Explain which situation requires each resource; do not require every resource to be loaded for every task.

Create a helper script only when it avoids repeated fragile work or provides useful deterministic checks. Document its runtime, dependencies, inputs, outputs, and side effects. Provide `--help` where practical, and direct consumers to inspect its usage before invoking it. Prefer relative paths and explicit output destinations.

Include an example when it demonstrates a consequential decision or a non-obvious output contract. Mark illustrative values clearly. Do not add empty folders, unfinished placeholders, duplicated manuals, or an entire sample project without a concrete use.

## Validate before delivery

Check both packaging and usefulness:

1. Parse the YAML frontmatter with an available YAML parser. Verify nonempty string fields, matching folder/name, and lowercase hyphenated naming. Quote descriptions containing YAML-sensitive punctuation. A Markdown heading or visual inspection alone does not validate YAML.
2. Check that linked local files exist, filename casing matches, and bundled resources use portable paths. Remove unfinished scaffolding and accidental private data.
3. Review activation against a realistic matching request and a nearby request outside the skill's purpose. Tighten the description if it attracts unrelated work or misses the intended task.
4. Walk through a representative task using only the finished skill and its declared inputs. Check that the instructions lead to a concrete result without undocumented tools or hidden context. For complex workflows, exercise the relevant steps in an isolated workspace when feasible.
5. Run every added or changed helper's usage command and a representative successful case. Exercise invalid input or failure handling where consequential. Do not report a walkthrough as an executed end-to-end test.
6. For updates, review the diff for unintended behavior changes and stale resource links. Use an available skill validator as an additional check, but do not treat schema validation as proof of behavior or Antigravity runtime compatibility.

Report the skill's path, purpose, validation performed, and remaining limitations. Commit and push when requested, staging only the intended files and verifying the push result. Otherwise leave the completed files ready for review.
