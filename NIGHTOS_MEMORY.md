---
name: nightos-session-memory
description: Night_OS full build state — features, decisions, architecture, file paths, deployed URL
metadata:
  type: project
---

# Night_OS — Session Memory

**Also called:** nightskyos (interchangeable)
**Deployed:** https://nightds.vercel.app
**Repo:** https://github.com/jwolff617/nightskyos
**Local file:** `C:\Users\sagep\Downloads\NightOS\index.html`
**Dictionary:** `C:\Users\sagep\Downloads\NightOS\NightOS_Dictionary.md`
**Supabase:** https://gcsfupdmyubwpoxbvnru.supabase.co

**Why:** Single-file React 18 + Supabase JS v2 + Babel CDN, no build step. Deploy = push to GitHub, Vercel auto-deploys.

---

## Current Build State (as of 2026-06-13)

All phases 1–4 are effectively complete. The app is live and functional.

### Features built and deployed

**Navigation & Map**
- Full persistent map always visible — shows node tree even in settings/journal/archive
- Map highlights current position across all 4 levels (area → category → note → detail)
- `Ctrl+↑` = go up one level (works everywhere)
- `Ctrl+↓` = go into first child
- `Ctrl+←` = previous sibling (or previous area at top row)
- `Ctrl+→` = next sibling (or next area at top row)
- Command bar: type path (`.` or `›` separated), `..` = up, `home` = root
- Breadcrumb prompt is clickable — each segment navigates to that level
- Header: ≡/⊞ toggle for panel/list view; ☀/🌙 theme toggle; ⚙ settings

**Panel Ranking & Reorder**
- `sort_order` field (already in DB schema) used as rank
- Rank badge `#1–#10` on every card
- `↑`/`↓` buttons on each card for click reorder
- Drag-and-drop reorder using native HTML5 drag API (no library)
  - Drag card onto another → swap positions; dashed accent border shows drop target
- `⇄` swap button: click one card, click another to swap
- Rank field in AddNodeModal (default = last slot; choose 1–N to set initial position)
- EOD Top-N Only rule: process only top N ranked new_day categories at EOD

**Color Dot System (new_day only)**
- 🔴 Red — no context match at any ancestor level → archived at EOD
- 🟢 Solid green — exact context match at this node's path → merges INTO existing context node
- 🟢 Green ring (hollow) — parent aligned, this node doesn't exist in context yet, slot available → NEW panel will be created in context at EOD
- 🟠 Orange — parent aligned but parent context node is full (10/10) → skipped at EOD (warning)
- 🟣 Purple — journal entry → always archived

Dots update live. Tooltips explain exactly what will happen at EOD.

**EOD Processing (11:59:59 PM)**
- `computeDot` checks alignment:
  - Exact full-path match → green
  - Ancestor match (up to note/depth-2) + slot available → green-new
  - Ancestor match + parent full → orange
  - No match → red
- `appendFragments` now CREATES missing intermediate nodes in context (not fall back to parent)
  - If node doesn't exist in context but parent does + slot available → created at EOD
  - If slot full → subtree skipped (orange dot warned user)
- Empty seed life framework nodes (with no content) are DELETED not archived at EOD
- Top-N Only rule: filter new_day tree to top N by sort_order before processing

**Archive**
- Per-row delete (✕ button)
- Checkbox bulk select + Select All
- Confirm before bulk delete
- Search, restore, flat table view

**Collabs**
- Project dropdown shows nodes under any "projects" ancestor in both new_day AND context
- Active Collaborations section shows accepted invites with "→ Go to" button
- Invite link generation (no email required — shareable URL with token)
- Pending invite modal on login when ?invite=TOKEN in URL
- `invitee_email` field (not `invitee_label` — that column doesn't exist)

**Settings**
- Theme (night/day), view mode (panel/standard), life_default toggle
- All EOD rules with toggles
- Top-N Only: when enabled, shows number input (1–10) for how many categories to process
- Auto-save with debounce

---

## Architecture Decisions

**Sort order = rank.** sort_order already existed in DB. Rank 1 = sort_order 0 (first shown). Displayed as #1, #2... to user.

**Dot alignment check stops at note level (depth 2).** Details and fragments don't need exact context matches — they inherit green from ancestor. Solid green = exact match, ring green = new node will be created.

**appendFragments creates nodes.** When a note/detail doesn't exist in context, EOD creates it (if slot available). Context grows naturally from new_day usage.

**Global keyboard handler in App.** Ctrl+Arrow registered once in App via refs, never goes stale. WorkspaceView uses `window.__nightos_focusedNode` ref for "navigate into focused" shortcut.

**Drag-and-drop = primary reorder UX.** ↑/↓ buttons still exist as secondary. Shift+Arrow removed (redundant).

**Map always visible.** MapModule never hides — when not in new_day/context, defaults to showing new_day tree.

---

## DB Schema (current)

Tables: `nodes`, `journal_entries`, `archive_rows`, `collab_invites`, `user_settings`, `eod_log`

Key fields on `nodes`:
- `sort_order int DEFAULT 0` — used for rank/ordering
- `is_project boolean DEFAULT false`
- `area` CHECK IN ('new_day','context','collabs')
- `level` CHECK IN ('category','note','detail','fragment')

`user_settings.rules` jsonb includes:
```json
{
  "notify_before_eod": true,
  "auto_append_matched": true,
  "auto_archive_unmatched": true,
  "auto_assign_projects_context": true,
  "archive_journal_at_eod": true,
  "keep_empty_categories": false,
  "eod_top_n_only": false,
  "eod_top_n": 3
}
```

---

## Key Functions (index.html)

- `buildTree(flatNodes)` — flat array → adjacency tree, sorted by sort_order
- `findNodeByPath(tree, pathSegs)` — walk tree by name segments
- `computeDot(node, pathFromCategory, contextTree)` — returns 'red'|'green'|'green-new'|'orange'|'purple'
- `runEODProcessing(userId, rules, nodes, journalEntries)` — full EOD logic
- `appendFragments(ndNode, ctxNode)` — recursive merge/create into context
- `DB.swapSortOrders(id1, order1, id2, order2)` — swap rank between two nodes
- `DB.deleteArchiveRow(id)` / `DB.deleteArchiveRows(ids)` — archive delete
- `MapModule` — always-visible tree map with Ctrl+Arrow navigation
- `WorkspaceView` — drag-and-drop, rank badges, swap, color dots, keyboard focus

---

## LIFE_SEED structure (seeded for new users)

Notes: today, journal, plans, reviews, projects, portfolio, drawing
Details under plans: goals, people, places, events, things
Details under reviews: books, movies, experiences, music
Details under projects: personal, work, creative
Details under portfolio: work, creative
Details under drawing: sketches

Seed nodes with no content are DELETED (not archived) at EOD.

---

## Recent git log (last 10 commits)

- feat: drag-and-drop reordering for panels and list rows
- feat: differentiate green vs green-new dot for EOD merge behavior
- fix: EOD creates new context panels, warns when parent is full
- fix: computeDot checks alignment at note level, not full path
- fix: EOD merge correctly handles unmatched notes/details
- refactor: Ctrl+Arrow is global map navigator, remove Shift+Arrow reorder
- feat: full persistent map + proper Ctrl+Arrow tree navigation
- feat: panel ranking, archive delete, header view toggle
- fix: EOD skips archiving empty seed framework nodes

---

## What is NOT yet built (from original spec)

- Project Board (9×9 simultaneous view) — mentioned in dictionary Section 11
- Slot consolidation helper (merge two items when a level hits 10/10) — currently just shows "slots full" modal
- Collab shared editing / live sync (last-write-wins, 15s polling) — invites work but shared editing not built
- Archive limit notification at >9,000 rows
