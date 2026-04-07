# Thread Collapse — Paperclip UI Patch

Gmail-style collapsing for Paperclip issue detail view.

When an issue has **more than 5 comments**, the timeline collapses automatically:
- All comments except the last are hidden
- A single **"Show N older messages"** button appears directly above the last comment
- Timeline events (status changes, assignments) and run entries are **always visible**
- The plan document collapses to title + first two lines of text

Click the button to reveal all hidden messages. Click any collapsed plan document to expand it. Direct links (`#document-plan`, `#comment-<id>`) auto-expand the target.

```
Issue with 8 comments  (> 5 threshold → collapses)
─────────────────────────────────────────────────────────────
▼ Plan                                          [▶ expand]
  Redesign the onboarding flow for new users. Focus on…

Strategy   run  3b1ebfeb  succeeded  3 days ago
Alex       updated this task  2 days ago  →  in_progress
Strategy   run  7fca4a3d  succeeded  2 days ago

──────── Show 6 older messages ────────

┌────────────────────────────────────────────────────────┐
│ Alex   just now                                        │  ← always expanded
│  Looks great. Ship it.                                 │
└────────────────────────────────────────────────────────┘
```

Events and runs are always visible regardless of count. Only comment cards collapse.

**No backend changes. No new dependencies. Three files changed.**

---

## Why

Paperclip issues accumulate many comments quickly — plans, drafts, reviews, revisions. Every time you open an issue you land at the top and scroll past everything to see the latest state.

This patch makes issues behave like a mail client: you land on the latest message, older context is one click away. Timeline events and run status remain fully visible so you always have context for when things happened.

---

## Behavior

- **Threshold**: collapses when `comments.length > 5`, otherwise shows everything expanded (short threads don't need collapsing)
- **Button placement**: "Show N older messages" appears immediately above the last comment — always visible at the bottom of the thread, not buried at the top
- **Events and runs**: always visible, never hidden
- **Plan document**: collapses to title + first 2 lines (click preview text or the `▶` button to expand)
- **Direct links**: `#document-plan` or `#comment-<id>` in the URL auto-expand the target and show all messages
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

## Implementation notes

### Button placement

The "Show N older messages" button is inserted as a sibling **above the last comment card**, not at the chronological position of the first hidden comment. This ensures the button is always visible at the bottom of the timeline regardless of how many events and runs are interleaved.

### What is a "comment"

Only items of `kind === "comment"` in the mixed timeline array count toward the threshold and collapse logic. Items of `kind === "event"` (status changes, assignments) and `kind === "run"` are always rendered.

### Plan document collapse timing

`collapsePlan` is passed as a prop to `IssueDocumentsSection`. Because comments load asynchronously, the prop arrives after the first render. A `useEffect` + ref pattern applies the collapse once — but only before the user has manually expanded the plan:

```tsx
const collapsePlanAppliedRef = useRef(false);
useEffect(() => {
  if (collapsePlan && !collapsePlanAppliedRef.current) {
    collapsePlanAppliedRef.current = true;
    setFoldedDocumentKeys((current) =>
      current.includes("plan") ? current : [...current, "plan"],
    );
  }
}, [collapsePlan]);
```

---

## See also

[paperclip-approval-gate](https://github.com/JIemyp/paperclip-approval-gate) — require human approval before any agent run executes, with a one-click Approve button in the issue view.
