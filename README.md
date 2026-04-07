# Thread Collapse — Paperclip UI Patch

Gmail-style comment thread collapsing for Paperclip issue detail view.

When you open an issue with many comments, only the **latest comment is expanded**. All earlier comments are collapsed to a one-line preview showing the author, timestamp, and the first ~120 characters of the message. Click any collapsed comment to expand it.

```
Issue with 12 comments
─────────────────────────────────────────────
▶  Alex Chen   3 days ago   Set up project structure and…
▶  Strategy    2 days ago   Based on the brief, I recomme…
▶  Alex Chen   2 days ago   Approved. Proceed with option…
▶  Copywriter  1 day ago    [code] Here's the first draft…
▶  Alex Chen   23 hours ago  Looks good overall. Please re…
   ┌─────────────────────────────────────────┐
   │ Strategy    just now                    │  ← expanded
   │                                         │
   │  Revision complete. Key changes:        │
   │  - Shortened headline to 8 words        │
   │  - Added social proof in paragraph 2    │
   │  - CTA moved above the fold             │
   └──────────────────────────���──────────────┘
────────────────────���────────────────────────
```

**No backend changes. No new dependencies. 164 lines of diff.**

---

## Why

Paperclip issues with active agent work accumulate dozens of comments — plans, outputs, reviews, revisions. Every time you open an issue you had to scroll past all of them to see what happened last.

This patch makes the issue detail behave like a mail client: you land on the latest message, and older context is one click away.

---

## Behavior

- **Only the last comment is expanded** on load
- **All earlier comments** (including the first) are collapsed to a preview row
- **Collapsed row shows:** avatar initials · author name · relative time · first ~120 characters (markdown stripped)
- **Click any collapsed row** to expand it inline
- **Direct links** to a comment (e.g. `#comment-abc123`) auto-expand the target
- **Timeline events and run entries** are unaffected — only comments are collapsed

---

## Installation

This is a source patch against `v2026.403.0`. Apply it to your Paperclip fork.

### Step 1 — Apply the patch

```bash
cd /path/to/paperclip
git apply thread-collapse.patch
```

Or apply manually — all changes are in one file: `ui/src/components/CommentThread.tsx`

### Step 2 — Rebuild the UI

```bash
pnpm --filter @paperclipai/ui build
# or full rebuild:
pnpm build
```

### Step 3 — Restart

```bash
systemctl --user restart paperclip.service
# or however you run Paperclip
```

---

## What the patch does

**New component: `CollapsedCommentCard`**

A compact button that shows author initials, name, timestamp, and a plain-text preview of the comment body. Markdown syntax is stripped before truncation so you see readable text, not `**bold**` syntax.

**New helper: `stripMarkdown`**

Strips code blocks, inline code, images, links, headings, bold, italic, strikethrough, and list markers before producing the preview text.

**Modified: `TimelineList`**

- Adds `expandedIds` state (Set of comment IDs the user has manually expanded)
- On render, finds the index of the last comment in the timeline
- Comments that are not the last and not in `expandedIds` render as `CollapsedCommentCard`
- When `highlightCommentId` prop changes, that comment is automatically added to `expandedIds`

---

## Compatibility

Tested against Paperclip `v2026.403.0`. The patch touches only `ui/src/components/CommentThread.tsx` and should apply cleanly to any recent version.

---

## See also

[paperclip-approval-gate](https://github.com/JIemyp/paperclip-approval-gate) — require human approval before any agent run executes, with a one-click Approve button in the issue view.
