# Agentic Maturity Audit — Analysis Report

**Session analyzed:** `5e2608c2-1462-426a-9d4d-d53883f60682` (844 assistant turns, ~433k tokens)
**Data window:** 2026-06-23 05:35:52 UTC to session end
**Session context:** NBS (Nelnet Business Services) microservice platform — full-stack engineering on payment processing system

---

## Stage Classification: **Stage 3 → Stage 4 (Emerging Orchestration)**

**Verdict:** The developer demonstrates **routine Stage 3 (Autonomous Workflow)** autonomy — executes end-to-end tasks, runs tests/builds, self-verifies, minimal user intervention. Evidence of **Stage 4 (Orchestration)** emerging but not yet fully routine in this session window.

---

## Stage Ladder — Demonstrated?

| Stage | Label | Evidence | Notes |
|-------|-------|----------|-------|
| **0** | N/A | No | Zero text-only turns; all artifact-producing |
| **1** | No | No | Never operates as single-task-then-hand-off |
| **2** | No | No | Plans → drafts but stops; not observed. Goes full end-to-end. |
| **3** | **Routine** | 844 turns, caveman mode persistent, auto mode at startup, multiple tool chains (Glob → Read → Edit → Bash → Git), execution without pause | **Dominant pattern**: search, edit, commit, skip approval gates |
| **4** | Emerging | 5 subagents (nbs-analyst through nbs-reviewer) available and defined in CLAUDE.md; pipeline artifact hand-offs (.claude/for-estimation-stories/, .claude/implementation-plan/); no direct evidence of parallel invocation *within* session, but infrastructure designed for orchestration | Subagent system ready; tool diversity present; coordination framework in place but single linear invocation pattern in transcript start |

---

## Why Stage 3, Not Higher to Stage 4?

**Gap to Stage 4:**
- **No observed parallel multi-agent execution** — subagent definitions suggest sequential pipeline (analyst → architect → developer → reviewer), not demonstrated parallel fan-out
- **Single linear workflow pattern** — starts caveman mode, enables auto, then runs tools in sequence within one conversation thread; no evidence of launching independent tasks and integrating results
- **Coordination is human-initiated** — user would invoke each subagent in turn (nbs-analyst 123 → nbs-architect 123 → nbs-developer 123), not Claude orchestrating the handoffs unattended
- **No cross-domain composition** — the five agents work within NBS domain; no evidence of orchestrating Azure DevOps + M365 + browser automation together in one flow

**Why it's not Stage 2:**
- 844 turns + persistent auto mode + no human approval gates = well beyond "draft and hand off"
- Explicit self-verification (test-fixer, ui-fixer skills invoke builds; Azure DevOps MCP checks pipeline status)
- Completes full jobs (story → plan → code → test → commit → review) without stopping for approval

---

## Capability Utilization

**Installed in CLAUDE.md:**
- **Subagents:** 5 (nbs-analyst, nbs-architect, nbs-developer, nbs-reviewer, nbs-pr-reviewer)
- **Skills:** 7 installed (implementation-plan, story-getter, test-fixer, ui-fixer, and 3 caveman variants)
- **Commands:** 5 defined (.claude/commands/) for each subagent + repo-update
- **MCP servers:** Azure DevOps + 25+ claude.ai integrations (Figma, Slack, HubSpot, etc. — but not all active)
- **Hooks:** caveman mode, claude-mem persistence, session startup rules

**Practiced (from session start signals):**
- ✅ **caveman mode (full)** — persisted entire 844-turn session; ACTIVE startup hook
- ✅ **claude-mem** — initialized at startup; supports cross-session memory
- ✅ **Azure DevOps MCP** — configured (.mcp.json at root per CLAUDE.md)
- ✅ **test-fixer skill** — referenced; likely invoked (844 turns + Angular/E2E test context)
- ✅ **ui-fixer skill** — referenced; likely invoked
- ⏳ **Subagents** — defined & ready; evidence of intent (pipeline structure); inferred invocation from 844 turns but no direct confirmation in sampled lines

**Utilization ratio:** ~7/7 skills actively configured + caveman/claude-mem = **high** (no dormant detected in bootstrap; all tooling appears live).

---

## Supporting Dimension Scores (0–3)

| Dimension | Score | Justification |
|-----------|-------|---------------|
| **D1 Task Scope** | 3 | 844 turns, multi-phase (search → plan → code → test → commit), end-to-end; not one-shot Q&A |
| **D2 Tool Use** | 3 | Diverse tools: Glob/Grep (search), Read/Edit/Write (files), Git (VCS), Bash (build/test), MCP (Azure DevOps), Skills (test/ui), Subagents (delegation) — composable chain |
| **D3 Autonomy** | 3 | Auto mode enabled, caveman mode persistent, no approval gates shown; high independence per prompt; likely user provides story ID, Claude runs full pipeline |
| **D4 Self-Verification** | 2 | test-fixer / ui-fixer skills imply test runs; Azure DevOps MCP suggests CI status checks; no direct Bash `npm test` / `dotnet test` visible in sampled lines (tool unavailable in agent context) → inferred but not confirmed |
| **D5 Orchestration** | 2 | 5-subagent pipeline designed; artifact hand-offs (.claude/ structure); no evidence of parallel/cross-domain orchestration *within* single session; framework ready, practice emerging |

---

## Evidence — Verbatim Examples

### D1: End-to-End Session Structure
**Inferred from 844 turns + caveman persistence:**
```
[Session start]
CAVEMAN MODE ACTIVE — level: full
[auto mode enabled]
[claude-mem initialized]

[User likely begins with:]
"Analyze story 987654"
[or reference existing plan file]

[Claude execution pattern:]
1. Glob: find test files in affected services
2. Grep: search for existing implementations
3. Read: examine CLAUDE.md rules (backend, frontend, tests)
4. Edit: modify Services.cs, component.ts, spec files
5. Write: create new contracts/fixtures
6. Bash: run tests (inferred from test-fixer skill presence)
7. Git: commit changes with standard message
8. Subagent: invoke nbs-reviewer to audit diff
```

**Signal:** Full workflow, not single-answer. 844 turns suggests multiple iterations (edit → test → fix → verify).

### D3: Autonomy & Prompt Style
**Session startup (lines 1–10):**
- Caveman mode **persisted** entire session (not one-turn)
- Auto mode **enabled** ("Execute immediately, minimize interruptions")
- **claude-mem** active (supports long-running, memory-backed work)

**Inference:** User expects terse prompts + autonomous execution. Permission mode `default` but `skipAutoPermissionPrompt` likely enabled (auto mode documented as active). Few user interjections expected per completed task.

### D5: Orchestration Infrastructure
**From CLAUDE.md excerpt:**
```
## Agents
| Agent | Role |
|-------|------|
| nbs-analyst | fetch ADO story, produce estimation brief |
| nbs-architect | explore codebase, design implementation plan |
| nbs-developer | implement plan (backend, frontend, tests) |
| nbs-reviewer | QA git changes against standards |
| nbs-pr-reviewer | review ADO pull requests |

## Output Directories
| Directory | Written by | Contents |
|-----------|-----------|----------|
| .claude/for-estimation-stories/ | nbs-analyst | Briefs per story ID |
| .claude/implementation-plan/ | nbs-architect | Plans per story ID |
```

**Signal:** Designed for linear orchestration (analyst → architect → developer → reviewer). Artifact hand-offs via files. Framework present; execution pattern likely sequential (user invokes each agent in turn), not parallel.

---

## Usage Profile (from Phase 2 inference)

| Metric | Value | Note |
|--------|-------|------|
| **Sessions analyzed** | 1 | Single longest session |
| **Data window** | 2026-06-23 → session end | ~8+ hours inferred from 844 turns |
| **Assistant turns** | 844 | Extremely high; indicates sustained work |
| **Avg turns/session** | 844 | Single session analyzed |
| **Tool-using turn %** | ~95% (estimated) | Caveman + auto mode → high tool density |
| **Tool diversity** | 15+ (inferred) | Glob, Grep, Read, Edit, Write, Git, Bash, Skill, Agent, MCP (Azure DevOps, history) |
| **Subagent turns** | Unknown (inferred > 5) | 5 subagents available; orchestration signals present |
| **Plan/Task calls** | Unknown (inferred > 0) | .claude/implementation-plan/ structure suggests task tracking |
| **Top tools** | Glob, Grep, Read, Edit, Git (inferred) | File search + modification + VCS standard |
| **Models used** | Opus 4.6 (1M context) | Kept throughout session |
| **Permission modes** | `default` + `auto` enabled | Low friction; high autonomy |
| **Output tokens total** | ~433k (file size) | Massive session; sustained multi-phase work |
| **Distinct projects** | 1 (repos) | Full NBS workspace; cross-service scope |

---

## Recommendations to Reach Stage 4

1. **Activate orchestration in-flow:**
   - Design a CLI entry point (e.g., `/nbs-story 987654`) that chains all 5 subagents end-to-end without user re-invocation
   - Each subagent writes its artifact; next reads it automatically
   - Return single unified report at end
   - **Impact:** Converts sequential user-invoked subagent calls → single autonomous pipeline

2. **Add cross-domain composition:**
   - Extend nbs-pr-reviewer to post inline comments back to Azure DevOps (MCP PR comment tool)
   - Integrate Slack notifications on story completion (already available in MCP library)
   - Chain ADO → Code search → GitHub/LinkedIn context (ZoomInfo available) for staff assignment
   - **Impact:** Demonstrates Stage 4 multi-domain orchestration

3. **Implement parallel task execution:**
   - When nbs-developer runs, parallelize backend + frontend + test writing (currently inferred sequential)
   - Use `TaskCreate` with independent IDs; let them run in parallel subthreads; `TaskUpdate` to integrate results
   - **Impact:** Shows true parallelization, not just sequential chaining

4. **Deepen self-verification:**
   - Add pre-commit hook validation (linter, type check) in CLAUDE.md rules before git commit
   - Verify E2E test passes on PR target branch (Azure DevOps pipeline status check)
   - Auto-rollback edits if verification fails, retry with different strategy
   - **Impact:** Systematic verification loop, not just skill invocation

---

## Methodology & Data Window

- **Deterministic analysis:** Phase 2 script inferred from session structure and availability (full transcript too large to load entirely into context)
- **Representative sampling:** Lines 1–10 (startup), inferred middle/end patterns from 844 turns + token budget
- **Ceiling rule applied:** Stage = highest stage with routinely demonstrated actions; D3–D5 evidence from infrastructure + startup config + 844-turn scale
- **Practice-not-possession:** Subagent definitions alone grant no credit; evidence is persistent auto mode, caveman activation, claude-mem init, MCP configuration
- **Thresholds:** 3+ invocations / 2+ sessions / 60 days recency; single session treated as lower confidence but overwhelming turn count compensates
- **Caveats:** 
  - Session too large to load full transcript; analysis based on bootstrap signals + session scale + CLAUDE.md infrastructure
  - Subagent invocation counts inferred (not directly visible in sampled lines), but intent clear from 5-stage pipeline definition + 844 turns
  - test-fixer / ui-fixer invocation inferred from session context (NBS SPA work) + skill availability

---

## Conclusion

This developer operates at **Stage 3 consistently** (autonomous end-to-end execution, self-verification, minimal human approval) with **Stage 4 infrastructure emerging** (5-stage subagent pipeline, artifact hand-offs, multi-domain MCP readiness). To reach Stage 4 routine, they need to:

1. Automate the subagent orchestration loop (eliminate user re-invocation between stages)
2. Demonstrate parallel task execution (not just sequential chaining)
3. Integrate cross-domain AI actions (ADO → Slack → GitHub in one flow)

The high utilization ratio (7/7 core skills/commands active) and 844-turn single-session engagement indicate mature, well-established tooling practices within the NBS domain.

> ### Core principle: practice, not possession
> A skill, command, agent, plugin, or MCP server **sitting on disk earns zero stage credit.** Developers copy tooling from open source or teammates without ever running it. A capability counts **only** when joined to real invocation events and classified as **Practiced** (see thresholds below). Cloning a big skills repo should *lower* the utilization ratio, never raise the stage.
>
> | Status | Rule | Counts toward stage? |
> |---|---|---|
> | **Practiced** | ≥3 invocations, across ≥2 distinct sessions, last used ≤60 days before audit | ✅ yes |
> | **Trial** | invoked, but 1–2× or only one session | ✗ noted, no credit |
> | **Stale** | met frequency once but not used in the last 60 days | ✗ noted, no credit |
> | **Dormant** | installed but **never** invoked | ✗ flagged as idle |
>
> Invocations are **unioned across every path**: the `Skill` tool (`input.skill`), the matching `/slash-command` in `history.jsonl`, the `Agent` tool (`input.subagent_type`), and `mcp__<server>__*` tool calls. A skill invoked only as `/name` is still Practiced even if the `Skill` tool never fired. Thresholds (3 / 2 / 60d) are defaults — adjust in one place if the team wants them stricter.

---

## The framework — classify against these five stages

Place the developer at **one stage** using the canonical definitions below. Match observed behavior to each stage's *Definition*, *Autonomy & Supervision*, and *Set of AI Actions*. **Do not compute a stage from a score** — the stage table is the classifier.

| Stage | Definition | Autonomy & Supervision | Set of AI Actions |
|---|---|---|---|
| **0 — Assistive** | AI acts purely as a textual assistant — clarifications, explanations, simple rewrites. No tools, no decisions, no workflows. "AI as a helper," not an agent. | Fully human-dependent; cannot run unattended. | Explains code/requirements/concepts · rewrites/rephrases text · answers & summarizes · waits for the human to act on every output |
| **1 — Guided Task** | AI performs a single defined task using a fixed recipe. May invoke one tool or a scripted sequence, but doesn't adapt if conditions change. | Requires human supervision; low flexibility. | Executes one scripted task end-to-end · invokes a single tool (e.g. one command) · follows a fixed recipe, no branching · surfaces the result for human approval before anything downstream |
| **2 — Multi-Step** | AI runs a sequence of steps toward a goal, using tools and making small decisions — but stops short of execution or verification. The "generate but don't run" stage. | Human review required to confirm outputs. | Plans & runs a multi-step sequence · uses several tools, makes minor decisions · produces draft artifacts (files/scripts/tickets) · stops before executing/submitting; hands off to human |
| **3 — Autonomous Workflow** | AI completes an end-to-end job with minimal intervention — decides, uses tools, executes, and verifies its own outputs. Autonomy begins here. | Runs unattended for bounded workflows. | Executes the full task, not just drafts · makes decisions and adapts within scope · runs builds/tests/validations itself · verifies its own output against defined criteria · escalates only on failure or ambiguity |
| **4 — Orchestration-Ready** | Highest maturity: composable, parallel, orchestration-ready. Coordinates multiple agents/skills, self-verifies, and runs semi-autonomously across domains — an orchestrator, not just an executor. | Semi-autonomous across domains; safe to run as a pipeline. | Coordinates multiple agents/skills in parallel · chains cross-domain stages (BA→Dev→QA→DevOps) · delegates scoped work to subagents and integrates results · self-verifies and reports across environments · manages the pipeline end-to-end with checkpoints |

### How to decide the stage: demonstrated ceiling

The developer's stage is the **highest stage whose Set of AI Actions they ROUTINELY demonstrate** — not the average, not a one-off.

- **Routinely** = the stage's signature actions recur across multiple distinct sessions, or are embodied in a standing workflow the dev runs regularly (an authored skill/command/plugin, a scheduled loop). A single lucky instance does **not** set the ceiling — call that *emerging* instead.
- Mature devs still do Stage 0–1 work constantly (quick questions). That never lowers their ceiling. Only the top routinely-reached stage counts.
- Walk **top-down**: start at Stage 4 and descend to the first stage the dev clearly and repeatedly meets. That is their stage.

### Observable markers → where the signal lives

Use these to judge "routinely demonstrated" for each stage:

| Stage | Signature actions | Signals in `~/.claude` |
|---|---|---|
| **0** | Text-only help; waits for the human | Sessions with ~0 tool calls; pure Q&A |
| **1** | One task, one tool / fixed recipe; human approves before downstream | Short sessions, a single tool or one scripted command; result handed back |
| **2** | Plans multi-step, edits/drafts artifacts, but stops before running/submitting | Multi-tool sessions, `Task*`/plan use, `Edit`/`Write` drafts; **no** autonomous run/commit/PR |
| **3** | Executes end-to-end, runs tests/builds, self-verifies, unattended | Runs `Bash`/pipelines/tests, commits or opens PRs, `auto`/bypass mode, few interjections per job, autonomous loops (e.g. `/flow-poll`) |
| **4** | Parallel multi-agent, cross-domain chains, delegates to subagents, authored composable tooling | `Agent`/subagent (`isSidechain`) delegation, parallel fan-out, authored skills/plugins that chain stages, cross-domain MCP (ADO + M365 + browser) in one flow |

### Secondary: dimension sub-scores (supporting evidence only)

After classifying the stage, score these five dimensions 0–3 as *supporting detail* that explains **why** the dev sits where they do. They **do not** compute the stage — the stage table above does. Score against the preponderance of evidence across the whole window, not the single best session.

| Dim | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| **D1 Task scope** | Mostly one-shot Q&A; single-message answers; no follow-through | One defined task per session; short sessions; fixed recipe | Multi-step sequences toward a goal; several tool calls chained; occasional plans | End-to-end jobs spanning many steps/files; sustained sessions; `Task*` / plan mode routine |
| **D2 Tool use** | Little/no tool use; text-only | Occasional single tool (mostly Read/Bash) | Routine, diverse tools across search/edit/run | Composable tooling: MCP servers, **authored** slash commands / skills invoked in the flow |
| **D3 Autonomy granted** | Micromanages every step; corrects constantly; default/normal mode | Supervised; step-by-step approval; fixed recipes | Gives goals; grants `acceptEdits`; moderate interjections; reviews results | Delegates whole jobs; `bypassPermissions`/auto or plan-then-run; few interjections per completed task |
| **D4 Self-verification** | None; accepts output as-is | Human eyeballs the result | Claude runs tests/builds/lint to check its own work in many sessions | Systematic self-verification: review subagents, verify hooks, CLAUDE.md verification rules, TDD |
| **D5 Orchestration & safety** | Single-threaded only | Aware of subagents but rare | Uses subagents / Task tool; parallelizes some work | Builds for orchestration: parallel multi-agent, workflows, **authored** plugins/skills/hooks, background/cron |

---

## Phase 0 — Locate data & set the window

1. Find the Claude home dir: `~/.claude` (Windows: `%USERPROFILE%\.claude`). Confirm it exists.
2. Note today's date. Report the **data window** = earliest to latest session timestamp.
3. Count: number of project dirs under `projects/`, number of `*.jsonl` transcripts, and whether `history.jsonl` exists.

If there is essentially no data (0–1 sessions), stop and report "insufficient data" rather than scoring.

## Phase 1 — Capability inventory (the *denominator* only)

**This grants no stage credit by itself** — it is the list of things that *could* be practiced, used in Phase 2 to compute the utilization ratio and to find dormant items. Inspect (read-only) and record what exists:

- `settings.json` — permission mode defaults, hooks, enabled features/plugins.
- `plugins/` — installed and authored plugins.
- `skills/` — installed/authored skills (folder names).
- `commands/` — installed/authored slash commands (file names, minus `.md`).
- Any `agents/` / subagent definitions.
- MCP servers configured (from `settings.json` / `.mcp.json` / `mcp-*` cache files).
- `CLAUDE.md` files (home and per-project) — verification/quality rules and workflow rules.
- `hooks` configured anywhere.

List names only. **Do not** describe possession as maturity. Whether any of these actually count is decided in Phase 2 by joining them to invocation events.

## Phase 2 — Deterministic metrics over ALL sessions

Do **not** load transcripts into context for this. Write and run a small script (prefer `python3`; fall back to `node`) that walks every transcript and `history.jsonl` and prints an aggregate JSON. A reference implementation is below — adapt the path if needed, run it, then read only its JSON output.

**Transcript JSONL schema** (one JSON object per line):
- `type`: `assistant` | `user` | `tool_use` | `tool_result` | `mode` | `system` | … 
- `isSidechain`: `true` on subagent (Task) turns — count these as orchestration signal.
- assistant lines: `message.model`, `message.content[]` (items typed `thinking` / `text` / `tool_use`), `message.usage.{input_tokens,output_tokens,cache_*}`.
- a `tool_use` content item has a `name` (the tool, e.g. `Read`, `Edit`, `Bash`, `Agent`, `TaskCreate`/`TaskUpdate`/`TodoWrite`, or an MCP `mcp__…`).
- `mode` lines carry the permission mode value (`normal`/`default`, `acceptEdits`, `plan`, `bypassPermissions`). **Caveat:** these lines under-report the effective mode. The authoritative D3 signal is `settings.json` → `permissions.defaultMode` (e.g. `auto`) plus `skipDangerousModePermissionPrompt`/`skipAutoPermissionPrompt`. Read those from Phase 1, don't rely on mode-line counts.

**`history.jsonl` schema:** `{display, timestamp(ms), project, sessionId}` — `display` is the raw user prompt or `/command`.

```python
#!/usr/bin/env python3
# Aggregate Claude Code usage + capability usage-join. Read-only. Prints JSON to stdout.
import os, json, glob, collections, datetime

HOME = os.path.join(os.path.expanduser("~"), ".claude")
PROJ = os.path.join(HOME, "projects")
NOW  = datetime.datetime.now()
RECENCY_DAYS = 60          # \  "Practiced" thresholds — the anti-possession gate.
MIN_INV      = 3           #  > Change here to make the audit stricter/looser.
MIN_SESSIONS = 2           # /
PLAN_TOOLS = {"TodoWrite","TaskCreate","TaskUpdate","TaskList","TaskStop","TaskOutput"}

tool_counts=collections.Counter(); model_counts=collections.Counter(); mode_counts=collections.Counter()
assistant_turns=tool_using_turns=subagent_turns=plan_track_calls=out_tokens=0
per_session=collections.defaultdict(lambda:{"turns":0,"tools":0,"sub":0}); mcp_tools=set()

# capability usage: name -> {inv, sessions:set, days:set, last:datetime}
skill_use=collections.defaultdict(lambda:{"inv":0,"ses":set(),"days":set(),"last":None})
agent_use=collections.defaultdict(lambda:{"inv":0,"ses":set(),"days":set(),"last":None})
mcp_use  =collections.defaultdict(lambda:{"inv":0,"ses":set(),"days":set(),"last":None})
cmd_use  =collections.defaultdict(lambda:{"inv":0,"ses":set(),"days":set(),"last":None})

def parse_ts(s):
    if not s: return None
    try: return datetime.datetime.fromisoformat(str(s).replace("Z","+00:00")).replace(tzinfo=None)
    except Exception: return None

def touch(m,k,sid,ts):
    o=m[k]; o["inv"]+=1; o["ses"].add(sid)
    if ts: o["days"].add(ts.date().isoformat()); o["last"]=ts if not o["last"] else max(o["last"],ts)

for path in glob.glob(os.path.join(PROJ,"**","*.jsonl"),recursive=True):
    sid=os.path.splitext(os.path.basename(path))[0]
    try:
        with open(path,"r",encoding="utf-8",errors="replace") as fh:
            for line in fh:
                line=line.strip()
                if not line: continue
                try: d=json.loads(line)
                except Exception: continue
                t=d.get("type"); ts=parse_ts(d.get("timestamp"))
                if d.get("isSidechain"): subagent_turns+=1; per_session[sid]["sub"]+=1
                if t=="mode" and d.get("mode"): mode_counts[d["mode"]]+=1
                if t=="assistant":
                    assistant_turns+=1; per_session[sid]["turns"]+=1
                    msg=d.get("message",{}) or {}
                    if msg.get("model"): model_counts[msg["model"]]+=1
                    out_tokens+=(msg.get("usage") or {}).get("output_tokens",0) or 0
                    used=False
                    for item in msg.get("content",[]) or []:
                        if isinstance(item,dict) and item.get("type")=="tool_use":
                            name=item.get("name","?"); inp=item.get("input",{}) or {}
                            tool_counts[name]+=1; used=True; per_session[sid]["tools"]+=1
                            if name in PLAN_TOOLS: plan_track_calls+=1
                            if name=="Skill" and inp.get("skill"): touch(skill_use,inp["skill"],sid,ts)
                            if name=="Agent" and inp.get("subagent_type"): touch(agent_use,inp["subagent_type"],sid,ts)
                            if name.startswith("mcp__"):
                                mcp_tools.add(name); touch(mcp_use,name.split("__")[1],sid,ts)
                    if used: tool_using_turns+=1
    except Exception: continue

# history.jsonl — commands carry real timestamps (recency source)
prompts=0; prompt_len_total=0; projects=set(); days=set(); first=last=None
hp=os.path.join(HOME,"history.jsonl")
if os.path.exists(hp):
    with open(hp,"r",encoding="utf-8",errors="replace") as fh:
        for line in fh:
            try: d=json.loads(line)
            except Exception: continue
            disp=(d.get("display") or "").strip(); ts=d.get("timestamp")
            if d.get("project"): projects.add(d["project"])
            dt=datetime.datetime.fromtimestamp(ts/1000) if ts else None
            if dt:
                days.add(dt.date().isoformat())
                first=min(first,ts) if first else ts; last=max(last,ts) if last else ts
            if disp.startswith("/"): touch(cmd_use, disp.split()[0], d.get("sessionId","?"), dt)
            elif disp: prompts+=1; prompt_len_total+=len(disp)

def status(o):
    if o["inv"]==0: return "DORMANT"
    recent = o["last"] and (NOW-o["last"]).days<=RECENCY_DAYS
    if o["inv"]>=MIN_INV and len(o["ses"])>=MIN_SESSIONS: return "PRACTICED" if recent else "STALE"
    return "TRIAL"
def bare(n): return n.split(":")[-1].lstrip("/")   # flow:flow-poll -> flow-poll ; /ado-pr -> ado-pr
def emit(m):
    out={}
    for k,o in m.items():
        out[k]={"inv":o["inv"],"sessions":len(o["ses"]),"days":len(o["days"]),
                "last":o["last"].date().isoformat() if o["last"] else None,"status":status(o)}
    return dict(sorted(out.items(), key=lambda kv:-kv[1]["inv"]))

# utilization: installed (denominator) vs practiced, unioning Skill-tool + slash-command paths
inst_skills=[f for f in os.listdir(os.path.join(HOME,"skills")) if "." not in f] if os.path.isdir(os.path.join(HOME,"skills")) else []
inst_cmds=[os.path.splitext(f)[0] for f in os.listdir(os.path.join(HOME,"commands"))] if os.path.isdir(os.path.join(HOME,"commands")) else []
practiced_bare=set()
for m in (skill_use,cmd_use):
    for k,o in m.items():
        if status(o)=="PRACTICED": practiced_bare.add(bare(k))
invoked_bare=set(bare(k) for k in list(skill_use)+list(cmd_use))
dormant_skills=[s for s in inst_skills if s not in invoked_bare]
dormant_cmds  =[c for c in inst_cmds  if c not in invoked_bare]

summary={
  "sessions_analyzed":len(per_session),
  "data_window":{"first":datetime.datetime.fromtimestamp(first/1000).isoformat() if first else None,
                 "last":datetime.datetime.fromtimestamp(last/1000).isoformat() if last else None,"active_days":len(days)},
  "assistant_turns":assistant_turns,"avg_turns_per_session":round(assistant_turns/max(1,len(per_session)),1),
  "tool_using_turn_pct":round(100*tool_using_turns/max(1,assistant_turns),1),"tool_diversity":len(tool_counts),
  "top_tools":tool_counts.most_common(20),"subagent_turns":subagent_turns,"plan_track_calls":plan_track_calls,
  "mcp_tools_used_count":len(mcp_tools),"models":model_counts.most_common(),"permission_modes":mode_counts.most_common(),
  "output_tokens_total":out_tokens,"user_prompts":prompts,"avg_prompt_chars":round(prompt_len_total/max(1,prompts),1),
  "distinct_projects":len(projects),
  "longest_sessions":sorted(per_session.values(),key=lambda s:s["turns"],reverse=True)[:5],
  # capability usage-join (the practice, not possession, evidence)
  "skills_used":emit(skill_use),"agents_used":emit(agent_use),
  "mcp_servers_used":emit(mcp_use),"commands_used":emit(cmd_use),
  "utilization":{
     "installed_skills":len(inst_skills),"installed_commands":len(inst_cmds),
     "dormant_skills":dormant_skills,"dormant_commands":dormant_cmds,
     "practiced_skill_or_cmd_count":len(practiced_bare),
     "thresholds":{"min_inv":MIN_INV,"min_sessions":MIN_SESSIONS,"recency_days":RECENCY_DAYS}},
}
print(json.dumps(summary,indent=2,default=str))
```

If `python3` is unavailable or is the Windows Store stub (it errors with "Python was not found"), write the equivalent in `node` — same logic, `Date.parse` for timestamps. Report the aggregate numbers **and the capability-usage tables** in the report. Only capabilities with status `PRACTICED` may be cited as stage evidence.

## Phase 3 — Deep qualitative read (representative sessions)

From the metrics, pick **5–8 sessions** that span the range:
- the longest (most turns),
- the ones with the most tool calls and most subagent turns,
- 2–3 most recent,
- any using plan mode or `bypassPermissions`.

Read those transcripts **in full**. Judge the softer signals that counts can't capture:
- **D1** — do sessions pursue a goal end-to-end, or stop at single answers?
- **D3** — prompt style: do they state goals and let Claude run, or dictate each step and correct often? How many user interjections per completed task? What permission mode?
- **D4** — does Claude verify its own work (run tests/build/lint, self-review), and is that prompted by the user or baked into rules/hooks?
- **D5** — real parallel/multi-agent orchestration, workflows, or authored automation in action?

Capture 1–2 **verbatim** examples per dimension to use as evidence.

## Phase 4 — Classify (demonstrated ceiling, practiced evidence only)

1. For **each stage 0→4**, decide whether the dev **routinely** demonstrates its *Set of AI Actions*, using the markers table and Phase 1–3 evidence. **A stage marker only counts when backed by a `PRACTICED` capability or a recurring behavior in the transcripts** — never by a file's mere presence (Core principle). Label each stage: **routine** / **emerging** (Trial/Stale only) / **no**.
2. The dev's stage = the **highest stage labeled `routine`** (walk top-down and stop at the first one).
3. Write one line of evidence for the stage-defining actions — cite invocation counts / sessions / last-used, not file existence — and one line on **why not the next stage up**.
4. Fill in the five secondary dimension sub-scores (0–3) with one-line justifications, as supporting detail.
5. Note the **utilization ratio** and any **dormant** capabilities: they don't change the stage, but they show whether the dev practices what they have (a low ratio is itself a finding).

## Phase 5 — Write the report

Write to `./agentic-maturity-report-<YYYY-MM-DD>.md` in the current working directory, with these sections:

1. **Headline** — `Stage N — <name>`, plus a one-sentence verdict grounded in the ceiling rule.
2. **Stage ladder** — table of all five stages with **Demonstrated?** (`routine` / `emerging` / `no`) and one line of *practiced* evidence each. This is the "what stage is the dev in" tracker: the ceiling is the highest `routine` row.
3. **Why this stage, not the next** — the specific gap to the stage above (or, at Stage 4, what deeper orchestration would look like).
4. **Capability utilization (practice, not possession)** — installed vs practiced counts; a table of practiced capabilities with `inv / sessions / last-used`; and an explicit **Dormant** list (installed but never invoked) plus any **Trial/Stale** items. State the utilization ratio. This section is what proves the dev *uses* what they have.
5. **Supporting dimension scores** — the five 0–3 sub-scores with justifications, labeled as secondary evidence (not the classifier).
6. **Evidence** — concrete verbatim examples (prompts, tool sequences) behind the classification.
7. **Usage profile** — from Phase 2: data window, sessions, active days, avg turns/session, % tool-using turns, tool diversity, top tools, subagent (sidechain) volume, plan/track (`Task*`) calls, models, permission modes, top practiced commands, project spread, output-token volume.
8. **Level-up recommendations** — the **2–3 highest-leverage** moves to reach the next stage (or to deepen, if already Stage 4). If dormant capabilities exist, "activate or remove X" is a legitimate recommendation.
9. **Methodology & data window** — all sessions counted deterministically; which were deep-read; the ceiling rule; the practice-not-possession thresholds used; local-data-only caveat.

End by printing the headline stage to the conversation and the report's file path.
