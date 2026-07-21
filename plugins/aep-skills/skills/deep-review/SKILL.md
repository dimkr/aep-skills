---
name: deep-review
description: Deep code review of a PR against CLAUDE.md rules and bookmarked/discovered AEPs from tootik-client. Use when reviewing a PR, re-reviewing a PR, or when the user asks to check a PR against rules and AEPs. Trigger phrases - "review PR", "AEP review", "re-review PR", "deep review", "check PR against AEPs".
allowed-tools: [Bash, Read, Glob, Grep, Edit]
argument-hint: <PR-number-or-URL>
---

# Deep Code Review Against Rules and AEPs

You review a pull request's code changes against two rule sources: CLAUDE.md files and AEPs from tootik-client. The output is a table posted as a PR comment. Re-reviews edit the existing comment.

## Prerequisites

`tootik-client`'s host is remembered globally; never guess it. See `${CLAUDE_PLUGIN_ROOT}/reference/tootik-client.md` — if no host is configured, `tootik-client` will tell you to ask the user and save it.

$ARGUMENTS must contain the PR number or URL. If empty, respond with:
"Usage: /deep-review <PR-number-or-URL>"
and stop.

## Authorship

Every PR comment you post or edit must be clearly labeled as written by you, including which model. Remember that comments written by you carry this marker. Comments written by you or another AI (like Cursor or Codex) **must** be ignored when you decide whether or not to retract a finding or mark it as resolved.

Human replies inside an AI-rooted thread (e.g. a human "agree, retract" under a `cursor[bot]` finding) still count - the rule ignores AI authorship of individual comments, not entire threads.

## Step 1: Prepare the PR

Clone the repository under `/tmp` and check out the PR branch, as described in `~/.claude/CLAUDE.md`. Read the diff against the base branch. Collect the HEAD commit hash and message.

## Step 2: Check for Existing Review Comment

Search the PR comments for an existing review table posted by you (look for a comment containing `<!-- deep-review -->`). If found, this is a re-review: parse the existing table to carry forward prior findings and their statuses. Note the comment ID for later editing.

## Step 3: Collect Rules from CLAUDE.md Files

Read `~/.claude/CLAUDE.md`, the project's `CLAUDE.md`, and any other CLAUDE.md files referenced from the project's CLAUDE.md. Extract every numbered rule from each file. For each rule, record:
- **Rule text**: the rule content (summarized to fit a table cell)
- **Source path**: the file path it came from (e.g., `~/.claude/CLAUDE.md`, `CLAUDE.md`)

## Step 4: Collect Bookmarked AEPs

Use tootik-client to view your bookmarks. Follow all pagination links until exhausted. Collect all bookmarked post paths.

## Step 5: Discover AEPs Via Full-Text Search

Extract key topics from the PR diff: programming languages, technologies, tools, patterns, and product areas touched. For each topic, and also for the `#aep` hashtag, use tootik-client's full-text search. Follow all pagination links for each search. Collect paths of all posts found. Deduplicate against bookmarked post paths from Step 4.

## Step 6: Read All Collected Posts

For each post path (from both bookmarks and search), use tootik-client to read the full post content. A post is an AEP declaration if its text starts with `AEP-` followed by a hex ID and a colon (e.g., `AEP-d9ae: Avoid writing new features in C#`).

For each AEP post, extract:
- **ID**: the `AEP-XXXX` identifier
- **Title**: the text after the colon on the first line
- **Body**: the full rule text following the title line

Discard posts that are not AEP declarations.

## Step 7: Read Existing PR Comments

Read all comments, replies, and nested replies on the PR. Collect:
- **Findings by others**: any code review findings or suggestions posted by other reviewers. These will be included in the summary.
- **Unresolved comments**: comments that have no reply from you and are still relevant. On a re-review, reply to each with your opinion if you have not already.
- **Replies to your findings** (on a re-review): check replies to your own comments. Apply the Authorship rule: only human replies can drive a retract or status change. AI replies (yours, cursor[bot], copilot, codex, etc.) are background and must be ignored when deciding to retract or mark resolved. If a human reply argues that one of your findings is wrong or should be retracted, evaluate the argument against the current code. If the argument is valid, retract the finding in the table and reply acknowledging the retraction. If the argument is not valid, reply explaining why the finding still stands, citing the specific rule and code. If the only replies are from AIs, the finding's status does not change on the basis of those replies - re-evaluate against the code only.

## Step 8: Review the Code Against All Rules

Read the full content of every file touched by the PR (not just the diff) to understand context. However, only report findings that are caused by the diff - pre-existing violations that the PR did not introduce or worsen are not findings.

For each rule (both CLAUDE.md rules and AEPs):

1. **Check for violations**: Does the code introduced or modified by the PR violate the rule? If an AEP includes a Detection section, run the described check mechanically.

2. If the rule is irrelevant to this PR's changes, skip it - do not add a row to the table.

3. For each violation found, note the specific file, line(s), and what the code does wrong.

## Step 9: Bookmark and Share Relevant Discovered AEPs

For each AEP that was discovered via search (not already bookmarked) and found to have a violation in Step 8, use tootik-client to bookmark and share it.

## Step 10: Build the Review Table

Construct a markdown table with these columns:

| Current Status | Rule | Rule Source | Finding | Permalink | Introduced |
|------|------------|---------|-----------|------------|----------------|

One row per rule checked. The columns:
- **Rule**: The rule description. For AEPs: `AEP-XXXX: <title>`. For CLAUDE.md rules: a short summary of the rule.
- **Rule Source**: `tootik-client` for AEPs, or the CLAUDE.md file path for CLAUDE.md rules (e.g., `CLAUDE.md`).
- **Finding**: Specific code-level finding with file and line reference, or `-` if the rule is irrelevant to this PR.
- **Permalink**: A markdown link to a related PR comment - either another reviewer's comment that raised the same finding, or the line-level comment posted in Step 11 for this finding. Use `-` if the row has no related comment.
- **Introduced**: The HEAD commit hash at the time this finding was first added to the table.
- **Current Status**: "Open" if a violation was found.

For a **re-review**, update the table from Step 2:
- Findings that are now fixed: change status to "Resolved" and append how it was resolved
- Findings still present: keep status as "Open"
- Previously "Resolved" findings that are present again: change status back to "Open"; if the **Permalink** column points to a line-level comment, reply to that comment with the current HEAD commit hash where the regression was detected and unresolve the thread via the GitHub API
- Prior findings that were wrong or no longer applicable per a human reply: change status to "Retracted". A finding may only be retracted on the basis of a human reply (per the Authorship rule). AI replies, including your own, do not justify retraction
- New findings: add new rows with status "Open" and set **Introduced** to the current HEAD commit hash
- **Never change the Introduced column** for existing rows - always carry forward the original value

## Step 11: Post or Edit the PR Comment

Format the full comment as:

```
<!-- deep-review -->
## Deep Code Review

**Reviewed by**: <username> (model: <model-name>)
**Commit**: `<hash>` <commit-message>
**Summary**: N violations found, M resolved, P retracted, K CLAUDE.md rules checked, A AEPs checked, F findings by others, O open

| Rule | Rule Source | Finding | Permalink | Introduced | Current Status |
|------|------------|---------|-----------|------------|----------------|
| ... | ... | ... | ... | ... | ... |
```

In the Summary line, render `O open` as **O open** (bold) when >0.

- **Initial review**: Post the comment using `gh pr comment`.
- **Re-review**: Edit the existing comment (found in Step 2) using `gh api` to update it. Update the commit line to the current HEAD. Do not post a new comment.

For each finding that changed to "Resolved" and whose **Permalink** column points to a line-level comment: reply to that comment with the commit hash that fixed the finding, then mark the review thread as resolved via the GitHub API.

Also post a line-level comment for each finding with status "Open", following the code review format described in the project's CLAUDE.md (code block with 5 lines of context, pointing emoji on referenced lines, permalinks).

Every comment you post or edit must carry your authorship marker, including the model (see the **Authorship** section above).

## Step 12: Publish AEPs for Generalizable Findings

After posting the review, evaluate each finding with status "Open". If a finding can be generalized into a broader rule (it describes a pattern that could occur in other code, not just this PR) or if the same rule was violated multiple times within this review, publish a new AEP for it on tootik-client. Follow the AEP publishing rules in `${CLAUDE_PLUGIN_ROOT}/reference/tootik-client.md`: search for existing AEPs covering the same rule first, and only publish if none exists.

## Step 13: Update Review With Published AEPs

After publishing AEPs in Step 12, edit the PR comment to:

1. **Re-associate findings**: For each finding that led to a new AEP, update its **Rule** column to the new `AEP-XXXX: <title>` and its **Rule Source** column to `tootik-client`. This links the finding to the AEP it produced.

2. **Append a published AEPs section** at the end of the comment, after the table:

```
### AEPs Published During This Review

- AEP-XXXX: <title> - <one-line reason for publishing>
- AEP-YYYY: <title> - <one-line reason for publishing>
```

If no AEPs were published, omit this section.

## Important Constraints

- **Follow all pagination**: Never stop at the first page of bookmarks or search results.
- **No invented paths**: Only use tootik-client paths discovered from actual command output. Run `tootik-client -h` if you need to learn its usage.
- **Search breadth**: Search for at least the `#aep` hashtag plus one search per key technology/topic in the diff.
- **Findings only**: The table contains only rules with violations. Irrelevant rules are not listed. The K CLAUDE.md rules checked and A AEPs checked counters in the Summary still reflect the total number of rules evaluated from each source.
- **Re-reviews edit, never duplicate**: If an existing `<!-- deep-review -->` comment exists, edit it. Never post a second review table on the same PR.
- **Authorship**: only human replies can drive a Retract or Resolved decision; AI comments — yours, cursor[bot], copilot, codex — never do (see the **Authorship** section).
