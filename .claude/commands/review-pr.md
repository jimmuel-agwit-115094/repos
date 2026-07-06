Review an Azure DevOps pull request against NBS company coding standards, security best practices, and risk assessment. Posts approved findings as PR comments.

PR Link: $ARGUMENTS

---

If `$ARGUMENTS` is empty, ask: "Provide a PR link or repo/PR-ID (e.g. `/review-pr https://dev.azure.com/nbsdev/Services/_git/Banking/pullrequest/123456` or `/review-pr Banking/123456`)."

---

## Step 1 — Review

Invoke the `pr-reviewer-agent`:

> "Review this pull request against all NBS standards. Use the pr-diff-reader skill to fetch the full diff and file content. Use the nbs-conventions skill to load all relevant convention docs. Check C#, Angular, tests, security, risk, and CSP-specific rules where applicable. Compile all findings but do not post yet — return them to me.
>
> PR: {$ARGUMENTS}"

Wait for the compiled review findings.

---

## Step 2 — Present to User

Display the full review findings returned by the `pr-reviewer-agent`.

Then ask:

> "Review complete. What would you like to do?
> 1. **Post all** — post every finding as PR comments
> 2. **Post selected** — tell me which finding numbers to post (e.g. "post 1, 3, 5")
> 3. **Edit first** — tell me what to change, I'll revise and re-present
> 4. **Skip posting** — keep the review for reference only"

---

## Step 3 — Post (if approved)

If user selects 1 or 2, invoke the `pr-reviewer-agent` again:

> "Post the following approved findings to the PR using the pr-commenter skill. Post the overall assessment first as a general thread, then post inline file-level findings in severity order (CRITICAL first).
>
> PR: {$ARGUMENTS}
> Findings to post: {all findings / finding numbers {X, Y, Z} as selected by user}
>
> Compiled review:
> {full review from Step 1}"

Report the posting summary (comments count + PR URL) returned by the agent.
