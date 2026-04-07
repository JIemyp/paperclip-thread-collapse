# Thread Collapse — Paperclip UI Patch

Gmail-style collapsing for Paperclip issue detail view.

When an issue has **more than 5 comments**, everything collapses automatically:
- The **plan document** (issue description) collapses to its title + first two lines of text
- All **comments** except the last collapse to a one-line preview

Click any collapsed item to expand it. Direct links (`#document-plan`, `#comment-<id>`) auto-expand the target.

```
Issue with 8 comments  (> 5 threshold → everything collapses)
─────────────────────────────────────────────────────────────
▼ Plan                                          [▶ expand]
  Redesign the onboarding flow for new users. Focus on…

▶  Strategy    3 days ago   Based on the brief, I recomme…
▶  Alex        2 days ago   Approved. Proceed with option…
▶  Copywriter  2 days ago   Here's the first draft. The h…
▶  Alex        1 day ago    Good start. Please shorten th…
▶  Copywriter  23h ago      Revised. Headline is now 7 wo…
▶  Alex        12h ago      Approved. Add social proof in…
▶  Copywriter  3h ago       Added testimonial block in se…
┌────────────────────────────────────────────────────────┐
│ Alex   just now                                        │  ← always expanded
│  Looks great. Ship it.                                 │
└────────────────────────────────────────────────────────┘
```

**No backend changes. No new dependencies. Two files changed.**

---

## Why

Paperclip issues accumulate many comments quickly — plans, drafts, reviews, revisions. Every time you open an issue you land at the top and scroll past everything to see the latest state.

This patch makes issues behave like a mail client: you land on the latest message, older context is one click away.

---

## Behavior

- **Threshold**: collapses when `comments.length > 5`, otherwise shows everything expanded (short threads don't need collapsing)
- **Plan document**: collapses to title + first 2 lines (click preview text or the `▶` button to expand)
- **Comments**: all except the last collapse to: avatar · author · time · first ~120 chars
- **Direct links**: `#document-plan` or `#comment-<id>` in the URL auto-expand the target
- **User preference**: once you expand the plan document, it stays expanded (persisted in localStorage per issue)

---

## Files changed

| File | What changed |
|------|-------------|
| `ui/src/components/CommentThread.tsx` | `CollapsedCommentCard` component + `stripMarkdown` helper + collapse logic in `TimelineList` |
| `ui/src/components/IssueDocumentsSection.tsx` | `collapsePlan` prop + preview text in folded state |
| `ui/src/pages/IssueDetail.tsx` | Pass `collapsePlan={threadComments.length > 5}` |

---

## Installation

Apply against Paperclip `v2026.403.0`.

### Option A — apply the full patch

```bash
cd /path/to/paperclip
git apply full-patch.patch
```

### Option B — apply patches separately

```bash
git apply thread-collapse.patch   # CommentThread.tsx only
git apply plan-collapse.patch     # IssueDocumentsSection + IssueDetail
```

### Rebuild and restart

```bash
pnpm --filter @paperclipai/ui build
systemctl --user restart paperclip.service
```

---

## Patches

| File | Scope |
|------|-------|
| `thread-collapse.patch` | Comment collapsing only (`CommentThread.tsx`) |
| `plan-collapse.patch` | Plan document collapsing (`IssueDocumentsSection.tsx` + `IssueDetail.tsx`) |
| `full-patch.patch` | Both combined |

---

## See also

[paperclip-approval-gate](https://github.com/JIemyp/paperclip-approval-gate) — require human approval before any agent run executes, with a one-click Approve button in the issue view.
