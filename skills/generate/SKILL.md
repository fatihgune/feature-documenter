---
name: generate
description: Generate the final feature document from traced state. Produces Notion/Confluence/Markdown-ready output with no code snippets. Use after dependency resolution is complete.
disable-model-invocation: true
argument-hint: "<feature-name>"
---

You are running the feature-documenter generate skill. This is Phase 3: documentation generation. You transform traced state data into polished, code-free documentation.

## Reference: Writing Guide

!cat ${CLAUDE_SKILL_DIR}/../run/references/writing-guide.md

## Reference: State File Format

!cat ${CLAUDE_SKILL_DIR}/../run/references/state-file-format.md

## Instructions

### Step 1: Validate and Load

1. Read `.feature-docs/_settings.md`. If missing, tell user to run `/feature-documenter:init` and stop.
2. Extract `format` and `tone` from settings.

### Step 2: Identify Target Feature

Parse `$ARGUMENTS` to identify the feature. Same matching logic as other skills:
- Exact slug match, then case-insensitive partial match
- If ambiguous, ask user to clarify
- If no match, stop

### Step 3: Load State and Validate

Read `.feature-docs/_state/<slug>.md` completely.

- If `phase` is `discovered` (not traced): tell user "This feature has not been traced yet. Run `/feature-documenter:trace {feature-name}` first." and stop.
- If `phase` is `traced` (not resolved): warn user "This feature has unresolved cross-service dependencies. The documentation will be incomplete in those areas. Continue anyway?" using AskUserQuestion. If no, stop.
- If `phase` is `resolved` or `resolving`: proceed.
- If `phase` is `generated`: ask "Documentation already generated. Regenerate?" using AskUserQuestion. If no, stop.

### Step 4: Load Format Template

Based on `format` from settings, read the appropriate template:

!cat ${CLAUDE_SKILL_DIR}/../run/references/format-templates/markdown.md

!cat ${CLAUDE_SKILL_DIR}/../run/references/format-templates/notion.md

!cat ${CLAUDE_SKILL_DIR}/../run/references/format-templates/confluence.md

Use only the template matching the user's configured format. Ignore the other two.

### Step 5: Generate Documentation

Using the state file data, writing guide rules, tone profile, and format template, generate the complete feature document.

**Generation rules:**

1. **Follow the format template structure exactly.** Use the section headings and organization from the template.

2. **Apply the writing guide hard rules.** Before finalizing:
   - Verify zero code snippets, inline code, or code blocks
   - Verify zero banned phrases
   - Verify zero hedging words in resolved sections
   - Verify actual codebase names are used (from the state file)
   - Verify error flows are covered alongside happy paths
   - Verify no emojis

3. **Apply the tone profile.** Match the calibration paragraph style for the selected tone:
   - Professional: third person, full sentences, section introductions
   - Conversational: second person, shorter paragraphs, contractions
   - Terse: bullet points, fragments, no filler

4. **Transform state data to prose/structure:**
   - Entry Points -> Feature Flow section (one subsection per entry point)
   - Service Layer Trace -> the narrative within each Feature Flow subsection
   - Repository/Data Layer -> Data Flow section
   - Outbound Calls + Resolved Dependencies -> Cross-Service Dependencies section
   - Unresolved Dependencies -> mentioned in Cross-Service Dependencies with appropriate hedging
   - Background Jobs and Events -> woven into relevant Feature Flow subsections and/or a dedicated subsection

5. **Business Context section**: Synthesize from the overall feature scope -- what business problem does this feature solve? Infer from the entry points, data, and service names. If you cannot confidently infer the business context, write a placeholder noting it should be filled in by the team.

6. **Error Handling section**: Extract from the trace data. Look for:
   - Validation steps that could fail
   - Outbound calls that could fail (what happens then?)
   - State transitions that have preconditions
   - If error handling was not explicitly traced, note the known failure points and what is likely to happen based on the architecture.

7. **Related Features section**: Look at the index.md for other features in the same repo or features that share dependencies with this one. Mention events this feature publishes that other features might consume.

### Step 6: Self-Review

Before writing the file, review the generated content against the writing guide:

- [ ] No code snippets, inline code, or code blocks anywhere
- [ ] No banned phrases
- [ ] No hedging in resolved sections
- [ ] Actual codebase names used throughout
- [ ] WHY before WHAT in each section
- [ ] Error flows covered
- [ ] No emojis
- [ ] Correct format template applied (check for Notion toggles / Confluence macros / plain Markdown as appropriate)
- [ ] Tone matches selected profile

If any violations found, fix them before writing.

### Step 7: Write Output

Write the document to `.feature-docs/features/<slug>.md`.

### Step 8: Update State

Update `.feature-docs/_state/<slug>.md`:
- Frontmatter: `phase: generated`, `status: complete`
- Trace Log: append `[{date} {time}] generate: Documentation generated in {format} format with {tone} tone`
- Next Steps: "Documentation complete. Review and publish."

### Step 9: Update Index

Read and update `.feature-docs/index.md`: change this feature's phase to `generated`.

### Step 10: Report

Tell the user:

"Documentation generated: `.feature-docs/features/{slug}.md`

Format: {format}
Tone: {tone}
Sections: {list the major sections written}

Review the output and make any adjustments. The document is ready for import into {Notion/Confluence/your documentation system}."
