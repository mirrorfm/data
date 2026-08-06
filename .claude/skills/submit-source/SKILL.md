---
name: submit-source
description: Submit one or more YouTube channels and/or Discogs labels to the mirror.fm data repo. Use when someone wants to add/submit channels or labels (usually gives YouTube URLs/@handles or Discogs label URLs) so they become synced Spotify playlists. Resolves IDs, runs the submission checks, appends the CSV rows, and opens a pull request — one commit per source type.
---

# Submit channels / labels to mirror.fm

Mirror.fm turns a **YouTube channel** or a **Discogs label** into an auto-synced Spotify
playlist. A submission is one row appended to a CSV in this repo, sent as a pull request. Once
merged, the sync automation ingests it and the playlist appears on the mirror.fm Spotify
profile — nothing to run by hand.

Two sources, two files (repo default branch: `master`):

| Source | File | ID column | ID format |
|---|---|---|---|
| YouTube channel | `youtube-channels.csv` | Channel ID (`UC…`) | 24-char `UC…` string |
| Discogs label | `discogs-labels.csv` | Label ID | numeric only |

Row format (note the space after each comma — match it exactly):
```
<ID>, <Name>, <Link>
```

## Batching rule

The user may pass many items at once, mixing channels and labels. Process them all, but keep
**channels and labels in separate commits** — e.g. 10 channels + 5 labels → one `[channel]`
commit touching `youtube-channels.csv` and one `[label]` commit touching `discogs-labels.csv`.
Never mix both files in one commit. A single PR carrying both commits is fine.

## Core flow

### 1. Sort the inputs by source type
For each item:
- A `youtube.com/...` URL, an `@handle`, or a bare `UC…` ID → YouTube channel.
- A `discogs.com/label/...` URL or a bare number → Discogs label.
- If ambiguous, ask.

### 2. Resolve ID + name for each item

**YouTube** — fetch the page and extract the canonical channel ID and title (works for
`@handle`, `/channel/UC…`, `/c/…`, `/user/…`):
```bash
URL="https://www.youtube.com/@StressComplex"
curl -sL "$URL" -A "Mozilla/5.0" | grep -o '"externalId":"[^"]*"' | head -1
curl -sL "$URL" -A "Mozilla/5.0" | grep -o '"channelMetadataRenderer":{"title":"[^"]*"' | head -1
```
- ID = the `externalId` (`UC…`). If the user already gave a `UC…` ID, use it directly.
- **Normalize the name to plain readable ASCII.** YouTube titles often come back as stylized
  Unicode (e.g. `𝚃𝚑𝚎 𝙳𝚒𝚐𝚐𝚒𝚗 𝙸𝚜𝚜𝚞𝚎` → `The Diggin Issue`). Submit the plain form.
- Link = the clean canonical URL the user gave (`https://www.youtube.com/@handle`).

**Discogs** — numeric ID is the number in the URL
(`discogs.com/label/985666-Sketches-Records` → `985666`). Name = label name; Link = full URL.

### 3. Run the submission checks (per the README criteria)

**Duplicate check (both types) — always.** Skip any item whose ID is already in the CSV; CI
(`csv-contribution-action`) hard-fails on duplicate IDs. Report which were skipped.
```bash
grep -iF "<ID>" youtube-channels.csv   # or discogs-labels.csv
```

**YouTube channel criteria (README).** Flag any channel that clearly fails — quality gates, not
enforced by CI:
- tracks formatted like `Artist – Track`;
- mostly single tracks, not "full album" / "full EP" / mixes;
- a decent number of tracks (~50+);
- not a single artist's own channel;
- **no matching Discogs label** — see next check.

**Does a Discogs label already exist for this channel?** If the channel is really a record label
that has a Discogs page, submit it as a **Discogs label instead of a YouTube channel** (README
rule). Search Discogs by name:
```bash
curl -sL "https://www.discogs.com/search/?type=label&q=<name>" -A "Mozilla/5.0" | grep -o 'label/[0-9]*-[^"]*' | head
```
If a clear match exists, tell the user and move that item to the label batch. If unsure, ask.

**Discogs labels** — community-standard, generally just add. Only numeric-ID + duplicate checks
apply.

### 4. Append the rows
Append surviving items to the **end** of the correct file, matching the `, ` (comma-space)
style. Keep a single trailing newline; each row must have exactly 3 columns.

## Submit as a pull request

### 5. Commit — one commit per type
Message convention (see `git log`):
- Channels: `[channel] <Name>[, <Name>…]`
- Labels: `[label] <Name>[, <Name>…]`

```bash
git checkout -b add-<short-desc>

git add youtube-channels.csv
git commit -m "[channel] The Diggin Issue, ballacid"    # only if channels were added

git add discogs-labels.csv
git commit -m "[label] Sketches Records"                # only if labels were added
```

### 6. Open the PR
```bash
git push -u origin HEAD
gh pr create --fill
```
CI runs `csv-contribution-action` on the PR (validates CSV shape, numeric Discogs IDs, no
duplicate IDs). Once approved and merged, the playlist syncs automatically.

## Notes
- This repo is public — add nothing but the intended CSV rows.
- Report a summary at the end: what was added, what was skipped (duplicates), and what was
  reclassified (channel → label) or flagged as failing the README criteria.
