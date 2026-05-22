---
name: oumeiao-weekly-report
description: Generate 欧美澳商务 weekly reports from the user's current BI session and user-provided private runtime inputs. Use when the user asks to write 欧美澳周报, diagnose weak channels, summarize progress, or propose channel-level and owner-level actions. Prefer the current visible BI tree and private runtime config over hardcoded internal routes, and never commit real data, traces, logs, or credentials.
---

# Oumeiao Weekly Report

## Overview

Use this skill to produce a data-backed 欧美澳商务周报 from the user's real BI environment, not from guessed table names or partial metric lists.

Before doing any analysis, this skill must verify that the BI session is actually usable. It must not assume that the user is already logged in, that the current account has the needed permissions, or that the required reports are visible.

Public-safe rule:
- Do not hardcode real BI URLs, credentials, cookies, private export paths, or browser traces in the repository.
- Read any real route mapping or export file locations only from a user-provided private file at runtime.
- Treat the repository as reusable logic plus templates, not as a storage place for business data.

If the user's BI uses `非港澳商务` as the visible path name and 欧美澳 as the business meaning, use that mapping at runtime instead of storing it as a mandatory constant.

## Private Runtime Inputs

At runtime, prefer a private config file that is not committed to Git. A suggested shape is provided in `references/private-runtime.template.json`.

The private runtime file can contain:
- BI base URL
- visible route names for the current account
- which report corresponds to chain achievement, channel split, owner split, daily trend, and detail drill-down
- optional private export directory
- optional business aliases such as visible path name vs business wording

If no private config is provided, learn from the current in-app browser tree and the current logged-in BI session.

## Business Scope Parameter

This skill must not assume that every user is working on 欧美澳商务.

At runtime, support a user-selectable business scope parameter:
- 欧美澳商务
- 港澳商务
- 台湾商务
- Local商务

Behavior rule:
- first determine which business scope the user wants
- then scan the visible BI tree under `海外商务` for that scope
- then learn the visible reports under that scope before diagnosing anything

Default mapping rule:
- 欧美澳商务 usually maps to the visible BI path `非港澳商务`
- 港澳商务 usually maps to `港澳商务`
- 台湾商务 usually maps to `台湾商务`
- Local商务 usually maps to `Local商务`

Do not hardcode these as absolute truth.
If the visible BI tree uses different names, prefer the current visible tree over historical assumptions.

## Session Scope Map

Before diagnosing any business scope, build a current-session scope map from the visible BI tree and current search results.

The scope map is a temporary runtime structure, not a committed file.

For the selected scope, capture at least:
- visible scope name in the current BI tree
- user-facing business label
- visible child report names
- which visible report best matches:
  - chain achievement
  - channel split
  - owner split
  - daily trend
  - detail drill-down
- which expected reports are missing in the current account

This scope map must be refreshed:
- at the start of a new reporting task
- after the user logs in with a different account
- after the user says permissions changed
- after the visible BI tree looks different from earlier in the session

## BI Access Preflight

Run this check before reading any report, drafting any report, or claiming that a report is unavailable.

1. Confirm whether the current in-app browser is already on the user's BI.
2. If BI is not open, open the BI URL provided by the user or the private runtime config.
3. Inspect the visible page state:
   - If the page is a login page, explicitly tell the user to log in and wait.
   - If the page is logged in but the report tree is not loaded yet, wait for the tree or re-open the analysis view.
   - If the page is logged in and the tree is visible, continue.
4. Confirm report permissions from the visible tree or search results, not from historical notes.
5. If a required report is not visible:
   - say that the current account may not have permission
   - list the missing report name
   - ask the user to switch account, grant permission, or provide an alternative visible report

Default user-facing prompt when blocked by access:
- `我需要你先登录 BI，或者把 BI 切到已登录状态，我再继续读取报表。`
- `我当前账号下看不到这个报表，可能是权限问题；你可以给我换到有权限的账号，或者告诉我当前可见的替代表。`

This preflight is mandatory for:
- half-week report
- weekly report
- monthly review
- any channel diagnosis based on BI

After login and permission are confirmed, this preflight must also verify:
- the selected business scope is visible in the current BI tree
- the child reports under that scope can be found by tree scan or search

The preflight must end with one of these states:
- ready: scope found and enough reports available
- partial: scope found but some supporting reports are missing
- blocked: login or permission is insufficient

## Workflow

1. Run the BI access preflight.
2. Determine the business scope parameter.
3. Build or refresh the session scope map for that business scope.
4. Scan the visible BI subtree under that business scope.
5. Confirm the route from the current BI tree or private runtime config, not from memory.
6. Read `references/metric-taxonomy.md` before diagnosing performance.
7. Determine the report mode: half-week, weekly, or monthly review.
8. Start from the result metrics and identify the actual problem type first.
9. Dynamically choose which report to open next based on that problem type.
10. Diagnose by layer: result, process, efficiency, distribution, owner, and drill-down evidence when needed.
11. Write the report with concrete channels, numbers, and actions.

## Scope Map Build Rules

When building the session scope map, follow this order:

1. Try scanning the visible scope subtree directly.
2. If the subtree is collapsed or hard to enumerate, use the BI search box with scope-specific report keywords.
3. Normalize the visible report list into functional roles.
4. Mark each functional role as:
   - found
   - missing
   - ambiguous

If ambiguous:
- prefer the report whose name and visible fields most closely match the intended role
- note the ambiguity internally and proceed

If missing:
- continue with available reports when possible
- explicitly lower confidence for conclusions that depend on the missing role

## Functional Role Matching

Map visible reports into these functional roles:
- chain achievement report
- channel split report
- owner split report
- daily trend report
- detail drill-down report
- optional monthly / cross-month support report

Examples of matching clues:
- `链路达成` usually indicates chain achievement
- `各渠道主辅投数据` usually indicates channel split
- `负责人` usually indicates owner split
- `日例子GMV` usually indicates daily trend
- `销售明细` usually indicates detail drill-down
- `跨月` usually indicates cross-month support

Treat these as clues, not absolute rules.

## Report Modes

This skill must support three standard modes.

### 1. Half-Week Report

Use when the user asks for:
- 半周会
- 半周报
- 周五进展会

Default objective:
- focus on current-month progress up to the day before the meeting
- also explain what changed from Tuesday to Thursday of the current week

Typical route:
- start with chain-achievement if available
- then branch dynamically to channel-split, owner-split, or daily-trend based on what is wrong

Default date logic:
- chain-achievement: month start to meeting minus one day
- channel-split: month start to meeting minus one day
- daily-trend: Tuesday to Thursday of the current week

Default output sections:
- overall progress
- this half-week's visible changes
- weak channels and bad links
- next actions before week close

### 2. Weekly Report

Use when the user asks for:
- 周报
- 周会
- 欧美澳周报

Default objective:
- summarize current weekly progress and month-to-date target completion
- identify good channels, bad channels, and accountable follow-ups

Typical route:
- start with chain-achievement
- branch to channel-split for structure diagnosis
- branch to owner-split when the issue needs ownership
- branch to daily-trend or detail drill-down only when the first two levels do not explain the issue

Default date logic:
- meeting label uses Tuesday to next Monday
- chain-achievement uses month start to the week-end date
- channel/owner/daily trend uses the week window unless the user says otherwise

Default output sections:
- overall progress
- channel structure
- weak channels
- stronger channels
- next-week actions

### 3. Monthly Review

Use when the user asks for:
- 月报
- 月复盘
- 月报复盘
- 欧美澳月复盘

Default objective:
- review full-month result completion
- diagnose which channels created the result and which wasted budget
- summarize structural lessons, not only weekly fluctuations

Typical route:
- start with chain-achievement
- then use channel-split and owner-split to identify structural gains and losses
- use daily-trend and detail drill-down only for representative channels that explain the month

## Dynamic Diagnosis Route

Do not treat reporting as a fixed table checklist.
The skill must first discover what is wrong, then decide which report to open next.

Base principle:
- issue discovery first
- report selection second
- output formatting last

### Step 1. Start from Result Metrics

If available, begin with the chain-achievement report and decide which result layer is off:
- example achievement
- appointment achievement
- GMV achievement
- ROI2 achievement
- spend progress versus achievement progress

### Step 2. Classify the Problem Type

Map the first visible issue into one of these buckets:
- result shortfall
- process bottleneck
- efficiency problem
- structural channel problem
- ownership problem
- unresolved drill-down case

### Step 3. Route to the Next Report

Use this dynamic routing logic:
- If result shortfall is visible at the top level, open channel-split next.
- If channel-split shows the issue is concentrated in a few channels, open owner-split next.
- If a channel has spend and example volume but weak appointments, treat it as an example-to-appointment problem.
- If a channel has appointments or attendance but zero GMV, treat it as a registration / attendance-to-deal / rolling-conversion problem.
- If a channel has GMV but weak ROI2, treat it as an efficiency problem and inspect cost and ASP before calling it healthy.
- If weekly or half-week change needs to be explained, open daily-trend next.
- If the issue still cannot be explained after owner-split, open detail drill-down for confirmation.

### Step 4. Decide What Tables to Output

Tables are not fixed in advance.
Only output tables that help explain the actual discovered problem.

When confidence is partial because the scope map is incomplete:
- prefer fewer, higher-confidence tables
- explicitly note which view could not be checked

Default date logic:
- use the full selected month unless the user asks for cross-month context
- use cross-month reports only when trend comparison is explicitly needed

Default output sections:
- month result summary
- structural gains
- structural losses
- inefficient budget use
- next-month optimization directions

## Standard Report Call Order

Do not improvise the call order when the standard route is available.

Base route:
- 1. chain-achievement report
- 2. channel-split report
- 3. owner-split report
- 4. daily-trend report
- 5. detail drill-down report

Use this decision rule:
- If the user only wants a fast status summary, stop after 1 and 2.
- If the user asks for weak channels and ownership, include 3.
- If the user asks for what changed this week or half-week, include 4.
- If the user asks why a channel is bad or asks for precise user-flow evidence, include 5.

## Date Templates

When the user does not specify custom logic, apply these templates.

### Half-Week

- Meeting day: Friday
- MTD window: month start to Thursday
- progress-change window: Tuesday to Thursday

### Weekly

- Meeting cycle: Tuesday to next Monday
- report title window: Tuesday to Monday
- chain-achievement window: month start to Monday

### Monthly

- review window: first day to last day of target month

Always state the exact date ranges used in the final output.

## Report Construction Rules

Always do the following:
- start from result metrics first
- then explain which process metrics are dragging the result
- then explain which channels are causing the issue
- then bind next actions to channel names and accountable follow-up
- scan the selected business scope subtree before trusting any historical route
- let the discovered issue decide which supporting tables appear in the output
- rebuild the session scope map whenever account context may have changed

Avoid these mistakes:
- do not write only from the user's example metrics
- do not stop at example count if GMV and ROI are the real problem
- do not treat all process metrics as equally important; explain which link is currently broken
- do not write generic action items without naming the channel and the metric gap
- do not force the same tables into every half-week, weekly, or monthly report
- do not assume 欧美澳商务 if the user may be working on 港澳、台湾、或 Local
- do not reuse stale scope learning after login, permission, or visible-tree changes

## Date Rules

- Weekly meeting cycle is Tuesday to next Monday.
- For a report labeled `5.12-5.18`:
  - Use `5.1-5.18` on the chain-achievement report
  - Use `5.12-5.18` on channel, owner, and daily trend views unless the user says otherwise
- Always state the actual date ranges used in the final report.

## BI Route

### 1. Chain Achievement

Use the chain-achievement report as the main board for overall progress.

Defaults:
- Only keep `主投学科 = 思维`
- Exclude `主投学科 = 美术`
- Keep the current user's visible BI scope

Answer:
- Is total progress lagging time progress
- Is the issue in examples, appointments, attendance, conversion, GMV, or ROI2
- Which channel groups are dragging the result

### 2. Channel Split

Use the channel-split report to compare channel groups and named channels.

Answer:
- Which channel group contributes most
- Which channel group is falling behind
- Which channels are high-spend low-output
- Which channels are efficient and can scale

### 3. Owner Split

Use the owner-split report to point to accountable owners and named channels.

Answer:
- Which specific channels have poor ROI or poor close rates
- Which channels have attendance but no deals
- Which owners need to review specific lanes first

### 4. Daily and Detail Support

Use these only when needed:
- the daily-trend report for day-level fluctuations and channel quality
- the detail drill-down report for detailed user-flow verification

## Diagnosis Rules

Do not diagnose from a partial metric list. Use the taxonomy in `references/metric-taxonomy.md`.

Apply this logic:
- If spend progress is high but example achievement is low, suspect acquisition quality or top-of-funnel efficiency.
- If examples are present but appointments are weak, prioritize example-to-appointment issues.
- If appointments are present but attendance is weak, prioritize attendance issues.
- If attendance is present but GMV is zero, prioritize registration, rolling conversion, and attendance-to-deal problems.
- If GMV exists but ROI2 is weak, check cost and ASP before calling the channel healthy.
- If main-invest metrics are acceptable but total GMV is weak, inspect secondary-invest contribution.

## Output Standard

Always include these sections:

1. Overall progress
- Time progress
- Spend progress
- Example achievement
- GMV achievement
- ROI2 status

2. Bad channels
- High spend, low output
- High attendance, zero deals
- Low ROI2
- Weak example-to-appointment rate

3. Better channels
- Higher ROI2
- Better GMV efficiency
- Evidence that scaling is reasonable

4. Actions
- Each action must bind to a channel name, a metric, and a concrete next move
- Avoid empty phrases like “improve conversion” without evidence

If analysis is blocked by BI access or permissions, do not fake a report.
Instead output:
- what was successfully opened
- which report is blocked
- whether the blocker is login, permission, or browser-download limitation
- the shortest next action the user needs to take

## Output Templates

Use concise Feishu-friendly prose plus tables when useful.

### Half-Week Output

- title
- data scope
- overall progress table
- changed channels this half-week
- weak channels table
- next actions

### Weekly Output

- title
- data scope
- overall progress table
- channel performance table
- weak channels table
- stronger channels table
- next-week actions

### Monthly Output

- title
- data scope
- month result table
- structural channel review table
- budget waste / low-efficiency table
- next-month actions

## Scope Scan Rule

Before using any report inside `海外商务`, scan the selected scope subtree and build a current-session list of visible child reports.

Minimum learning target:
- visible child report names
- which report looks like chain achievement
- which report looks like channel split
- which report looks like owner split
- which report looks like daily trend
- which report looks like detail drill-down

If the visible reports differ from historical learning notes:
- prefer the current session
- mention the mismatch only if it affects analysis quality

## Confidence Rule

Every report should implicitly track confidence from the scope map and actual reports opened.

High confidence:
- scope found
- chain achievement found
- channel split found
- owner split or daily trend found when needed

Medium confidence:
- scope found
- only chain achievement plus one supporting report found

Low confidence:
- scope found but key supporting roles missing
- conclusions rely on partial evidence only

If confidence is not high, do not present the report as complete certainty.

## Writing Style

- Use 欧美澳 in the report title and business wording.
- Mention a visible BI path name only when explaining the source route.
- Prefer concise weekly-report prose that can be pasted into Feishu.
- Put findings before generic summaries.
- Do not invent unavailable tables or metrics.
- Do not claim that BI was fully learned if the current session has not passed the BI access preflight.

## Git Safety

Never commit:
- real BI export files such as `.xlsx`, `.csv`, `.tsv`
- browser traces, screenshots, cookies, or session logs
- private config files with real URLs, account-specific route names, or local private paths
- generated weekly reports that include real business data unless the user explicitly wants a private repository

Safe to commit:
- this skill definition
- metric taxonomy without real figures
- private config templates with placeholder values
- scripts that read private inputs only at runtime

## References

- Read `references/metric-taxonomy.md` when preparing a report or diagnosing weak channels.
- Use `references/private-runtime.template.json` as the public template for private runtime wiring.
- Read `references/report-modes.md` for standard reporting-mode behavior.
- Read `references/output-templates.md` for Feishu-friendly output structure.
