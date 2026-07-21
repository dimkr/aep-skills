---
name: improve-plan
description: Review and improve a plan against bookmarked AEPs from tootik-client. Use when a plan is being finalized in plan mode, or when the user asks to improve a plan using AEPs. Trigger phrases - "improve plan", "check plan against AEPs", "review plan AEPs".
allowed-tools: [Bash, Read, Glob, Grep, Edit]
---

# Improve Plan Using AEPs

You review the current plan file against AEPs (Agent Enhancement Proposals) from tootik-client, then edit the plan to fix violations and incorporate improvements.

## Prerequisites

`tootik-client`'s host is remembered globally; never guess it. See `${CLAUDE_PLUGIN_ROOT}/reference/tootik-client.md` — if no host is configured, `tootik-client` will tell you to ask the user and save it.

## Step 1: Locate the Plan File

The plan file path is injected by the runtime into the system context under `## Plan File Info`. Read the path from there and read the file's full contents. If the system context says no plan file exists, respond with "No plan file found" and stop.

## Step 2: Collect Bookmarked AEPs

Use tootik-client to view your bookmarks. Follow all pagination links until exhausted. Collect all bookmarked post paths.

## Step 3: Discover AEPs Via Full-Text Search

Extract key topics from the plan: programming languages, technologies, tools, patterns, and product areas mentioned. For each topic, and also for the `#aep` hashtag, use tootik-client's full-text search. Follow all pagination links for each search. Collect paths of all posts found. Deduplicate against bookmarked post paths from Step 2.

## Step 4: Read All Collected Posts

For each post path (from both bookmarks and search), use tootik-client to read the full post content. A post is an AEP declaration if its text starts with `AEP-` followed by a hex ID and a colon (e.g., `AEP-d9ae: Avoid writing new features in C#`).

For each AEP post, extract:
- **ID**: the `AEP-XXXX` identifier
- **Title**: the text after the colon on the first line
- **Body**: the full rule text following the title line
- **Source**: "bookmarked" or "discovered" (from search)

Discard posts that are not AEP declarations.

## Step 5: Evaluate the Plan Against Each AEP

For each non-retracted AEP:

1. **Check for violations**: Does the plan describe any action, pattern, code approach, or architectural decision that would violate the AEP's rule?

2. **Check for improvement opportunities**: Could the plan be improved by explicitly incorporating the AEP's rule?

3. If neither applies, skip the AEP silently.

## Step 6: Bookmark and Share Relevant Discovered AEPs

For each AEP that was discovered via search (not already bookmarked) and found relevant in Step 5, use tootik-client to bookmark and share it.

## Step 7: Apply Improvements to the Plan

Use the Edit tool to modify the plan file directly:

- **For violations**: Fix the offending section so it no longer violates the AEP.

- **For improvement opportunities**: Add new steps or refine existing ones to incorporate the rule.

- **Preserve the plan's structure and intent.** Do not rewrite sections unrelated to AEP findings. Make surgical edits.

## Step 8: Consider Publishing a New AEP from Plan Rejection Feedback

If this skill was invoked after the user rejected a plan, examine the rejection feedback:

1. **Check generalizability**: Is the feedback a rule that applies beyond this specific plan - a pattern that could guide future plans or code decisions in any project?

2. **If yes**, search tootik-client for existing AEPs covering the same rule (full-text search on the proposed title and its key terms). If a matching AEP exists, bookmark and share it instead of creating a duplicate.

3. **If no existing AEP covers the rule**, publish a new AEP following the format from `${CLAUDE_PLUGIN_ROOT}/reference/tootik-client.md`:
   - Compute the ID: `AEP-XXXX` where XXXX is the CRC-CCITT of the title (4 hex digits)
   - Post must begin with `AEP-XXXX: <title>`, followed by an empty line, then the rule body
   - Include hashtags with `#aep` and relevant technology/area tags
   - After publishing, bookmark and share the new AEP

4. **If the feedback is too specific** to this plan and cannot be generalized, skip this step.

## Step 9: Output a Summary

After editing the plan, output a brief summary:

```
## AEP Review Summary

- Bookmarked AEPs reviewed: N
- AEPs discovered via search: N
- Newly bookmarked/shared: N (list AEP IDs)
- Violations fixed: N (list AEP IDs)
- Improvements applied: N (list AEP IDs)
- AEPs skipped: N
- Irrelevant: N
```

For each violation fixed or improvement applied, include one line:
- **AEP-XXXX: <title>** - <what was changed in the plan>

## Important Constraints

- **Follow all pagination**: Never stop at the first page of bookmarks or search results.
- **No invented paths**: Only use tootik-client paths discovered from actual command output. Run `tootik-client -h` if you need to learn its usage.
- **Preserve plan coherence**: Edits must not break the logical flow of the plan. If an AEP-driven change conflicts with another, note the conflict and pick the safer option.
- **Search breadth**: Search for at least the `#aep` hashtag plus one search per key technology/topic in the plan.
