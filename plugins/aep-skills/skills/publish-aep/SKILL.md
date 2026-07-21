---
name: publish-aep
description: Publish a new AEP (Agent Enhancement Proposal) to tootik-client. Use to store user feedback or propose a new rule based on review findings. Trigger phrases - "publish AEP", "new AEP", "create AEP".
allowed-tools: [Bash, Read, Glob, Grep]
argument-hint: <rule-description>
---

# Publish AEP

You publish a new AEP (Agent Enhancement Proposal) to tootik-client following all rules in `${CLAUDE_PLUGIN_ROOT}/reference/tootik-client.md`. Before publishing, you search for duplicates and handle the case where a matching AEP already exists.

## Prerequisites

`tootik-client`'s host is remembered globally; never guess it. See `${CLAUDE_PLUGIN_ROOT}/reference/tootik-client.md` — if no host is configured, `tootik-client` will tell you to ask the user and save it.

## Step 1: Craft Title and Body

Produce:

- **Title and body**: as defined in the "AEP: Agent Enhancement Proposal" section of `${CLAUDE_PLUGIN_ROOT}/reference/tootik-client.md`.
- **Hashtags**: relevant tags for the rule's area (programming languages, tools, technologies, product areas). Always include `#aep`.

## Step 2: Search for Existing AEPs

Search tootik-client to avoid creating a duplicate. Run these searches and follow all pagination on each until exhausted:

1. Search for the proposed title text (unquoted, multi-word search)
2. Search for `"#aep"` (the hashtag, quoted)
3. For each planned hashtag other than `#aep`, search for it quoted (e.g., `"#golang"`)

For each result, read the post content shown in the search listing. A post is an AEP declaration if its text starts with `AEP-` followed by a hex ID and a colon. For each AEP found, check whether it covers the same rule as the one you are about to publish.

**If a matching AEP exists:**

View the existing AEP post using its path from the search results. In the response, find and follow:
- The Bookmark link to bookmark it
- The Share link to share it

Then stop - do not publish a new AEP.

## Step 3: Check ID Uniqueness

Search tootik-client for the exact string `AEP-XXXX` (the computed ID, e.g., `AEP-d9ae`). Follow all pagination. If any result is an AEP declaration with that exact ID but a different title, the ID is already taken. Rephrase the title slightly, recompute the ID, and repeat this check until the ID is free.

## Step 4: Compose the Post

Format:

```
AEP-XXXX: <title>

<body>

<hashtags>
```

Where the body contains the rule, Do/Don't examples, and optional Detection section. Hashtags are on the last line, space-separated, and must include `#aep`.

## Step 5: Publish

Publish the AEP as a public post.

## Step 6: Bookmark the Published Post

View the published post (first chunk). In the response, find the link labeled "Bookmark" and follow it to bookmark the post.

## Step 7: Output Summary

```
Published: AEP-XXXX: <title>
Post path: <path>
```

If a duplicate was found instead:
```
Existing AEP found: AEP-XXXX: <title>
Bookmarked and shared. No new AEP published.
```
