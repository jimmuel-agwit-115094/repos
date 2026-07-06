Run the full story implementation pipeline autonomously. Fetches the ADO story, analyzes it, designs the implementation, writes production code and tests, audits the result, and creates a Draft PR — all without user intervention.

Story ID: $ARGUMENTS

---

If `$ARGUMENTS` is empty, ask: "Provide a story number (e.g. /craft 12345)."

---

## Pipeline

Run each step in sequence. Pass the previous step's full output as context to the next step. Do not pause between steps.

### Step 1 — Story Analysis

Invoke the `story-analyst-agent`:

> "Analyze ADO story {$ARGUMENTS}. Use the ado-story-reader skill to fetch it and the story-analyzer skill to produce the full STORY BRIEF. Return the complete structured brief including the exact verbatim story title."

Wait for the brief. Carry it forward.

---

### Step 2 — Architecture & Implementation Plan

Invoke the `architect-agent`:

> "Design the implementation plan for the following story brief. Use the codebase-explorer skill to find reference patterns in the actual codebase before designing anything.
>
> {STORY BRIEF from Step 1}"

Wait for the plan. Carry it forward.

---

### Step 3 — Implementation

Invoke the `developer-agent`:

> "Implement the following plan. Write all production code and tests. Self-audit before returning.
>
> Story: {id} — {title}
>
> {IMPLEMENTATION PLAN from Step 2}"

Wait for the implementation report. Carry it forward.

---

### Step 4 — QA Audit

Invoke the `qa-agent`:

> "Audit the following implementation against the plan. Read every file that was created or modified. Fetch ADO test cases linked to story {id}. Return a SHIP / BLOCK verdict.
>
> Plan:
> {IMPLEMENTATION PLAN from Step 2}
>
> Developer report:
> {IMPLEMENTATION REPORT from Step 3}"

Wait for the verdict. Carry it forward.

---

### Step 5 — Draft PR

Invoke the `draft-pr-agent`:

> "Create a DRAFT pull request for story {id}.
>
> Story title (use this exactly as the PR title): {exact verbatim title from Step 1}
> Story ID: {id}
> QA verdict: {verdict from Step 4}
>
> Detect the current branch and create the PR. Never create a non-draft PR."

Wait for the PR URL.

---

## Final Summary

After all steps complete, display:

```
================================================================
CRAFT COMPLETE — Story {id}: {title}
================================================================

Story analyzed:   {AC count} acceptance criteria
Plan designed:    {N} implementation steps across {M} files
Code written:     {list of created/modified files}
QA verdict:       {SHIP / BLOCK / CONDITIONAL SHIP}
Draft PR:         {PR URL}

{If BLOCK or CONDITIONAL SHIP:}
Required actions:
{list from qa-agent verdict}
================================================================
```
