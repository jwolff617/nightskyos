# Night_OS Dictionary
**Address:** `B.Night_OS`
**Version:** 0.3
**Extends:** Core Dictionary v0.6

---

## SECTION 1 — OVERVIEW

Night_OS is a personal note management dictionary on the Sage_DS platform. Its core purpose is to force users to actively sort and resolve daily notes: content created during a day lives in `new_day` and is automatically archived or merged into `context` at end of day (11:59:59 PM), so each new day begins with a fresh workspace.

**Hierarchy:**
```
Night_OS.{contents}.{category}.{note}.{detail}.{fragment}
```

Maximum 10 items per level → 100,000 fragments per workspace area.

**Stack:** React + Supabase + Babel (CDN), single HTML file, deployable to Vercel.

---

## SECTION 2 — CONTENTS AREAS

Night_OS uses 6 of the 10 available contents slots:

| Slot | Name | Type | Description |
|---|---|---|---|
| 1 | `settings` | system | App configuration: theme, view mode, rules, account |
| 2 | `new_day` | workspace | Today's temporary workspace; cleared at end of day |
| 3 | `context` | workspace | Persistent notes; never auto-archived |
| 4 | `journal` | view | Shortcut into new_day.life.journal; entries archived at end of day |
| 5 | `archive` | repository | Flat table of all archived items; max 10,000 rows |
| 6 | `collabs` | workspace | Collaborative projects organized by theme categories |

---

## SECTION 3 — COLOR DOT SYSTEM

Color dots appear on `new_day` items only. They indicate what will happen at end of day.

| Color | Meaning | At 11:59:59 PM |
|---|---|---|
| 🔴 Red | No matching `context` path exists | Item archived |
| 🟢 Green | Matching `context` path exists | Item appended to context, then removed from new_day |
| 🟣 Purple | Journal entry — no context equivalent exists by design | Entry archived |

**Matching rule:** A new_day item matches a context item when both share the identical path from category through the item's level.

Examples:
- `new_day.life.reviews.books` ↔ `context.life.reviews.books` → 🟢 green
- `new_day.life.today.mood` (no context.life.today.mood) → 🔴 red
- Any `new_day.life.journal.*` entry → 🟣 purple

**Dot is set at creation** and updated live as context items are added or removed. When a new_day item is created, the system immediately checks for a context match and assigns the dot, so the user knows before end of day whether action is needed.

---

## SECTION 4 — NEW_DAY WORKSPACE

### 4.1 Behavior
- Content is temporary; exists only during the current calendar day
- At 11:59:59 PM, end-of-day processing runs (see Section 10)
- new_day begins empty at 12:00 AM each day

### 4.2 Life — Default Category
`new_day.life` is a persistent default category that appears automatically for new users. It mirrors `context.life` in structure. Users may disable it in Settings to free a category slot.

**Life sub-structure** (same in both new_day and context):

| Note | Details | Fragments |
|---|---|---|
| `today` | `quotes`, `gratitudes` | Individual quote or gratitude text |
| `journal` | _(entries at detail level, max 10)_ | _(no fragments — journal is one level deep)_ |
| `plans` | `goals`, `people`, `places`, `events`, `things` | Individual plan item text |
| `reviews` | `books`, `movies`, `experiences`, `music` | Individual review text |
| `projects` | `personal`, `work`, `creative` | Individual project item text |
| `portfolio` | `work`, `creative` | Individual portfolio item text |
| `drawing` | `sketches` | Individual drawing data |

Life's notes with matching paths in both new_day and context automatically receive green dots.

### 4.3 User-Defined Categories
Users may create up to 9 additional categories in new_day (slots 2–10; slot 1 is Life). Each follows the full category.note.detail.fragment hierarchy.

### 4.4 Slot Enforcement
When a user attempts to add an 11th item at any level, `slot_limit` fires and the app guides consolidation:

1. **Prompt:** "This level is full (10/10). To add a new item, combine two existing items into a new parent, or move one item down a level."
2. **Offer:** Select two existing items → merge into a new named parent
3. **If parent level also full:** Repeat consolidation guidance up the hierarchy

The consolidation helper should name candidate merges (items with similar names or content).

---

## SECTION 5 — CONTEXT WORKSPACE

### 5.1 Behavior
- Content is permanent; never subject to end-of-day archiving
- Serves as the merge destination for new_day items with green dots
- User may edit context directly at any time
- Merged content is appended to the end of the matching item's content

### 5.2 Life — Default Category
`context.life` is pre-populated with the same structure as new_day.life for new users (see Section 4.2).

### 5.3 Slot Enforcement
Same 10-item cap and consolidation guidance as new_day.

---

## SECTION 6 — JOURNAL

### 6.1 Behavior
- Structurally located at `new_day.life.journal` — entries live at the detail level
- Accessible via the top-level `journal` contents shortcut (does not duplicate data)
- Max 10 journal entries per day
- Each entry = one detail-level node with a title and body; no fragments below
- No context equivalent; journal cannot be merged, only archived
- All entries carry 🟣 purple dots

### 6.2 Entry Structure
| Field | Description |
|---|---|
| `title` | Short label; auto-populated with today's date if left blank |
| `content` | Free-text body |
| `created_at` | Timestamp |

### 6.3 End of Day
All journal entries are archived at 11:59:59 PM. A user who wants to keep an entry must manually move it to a context note before midnight.

### 6.4 When Full (10/10)
Cannot add more entries. Options presented:
- Edit an existing entry to free its slot
- Move an entry to a context note to free the slot

---

## SECTION 7 — ARCHIVE

### 7.1 Structure
Archive is a flat table, not a navigable hierarchy. Each row is one archived item.

| Column | Description |
|---|---|
| `original_path` | Full address at archiving (e.g., `new_day.life.reviews.books.dune_review`) |
| `level` | Original hierarchy level: `category`, `note`, `detail`, or `fragment` |
| `parent_name` | Name of the parent item at time of archiving |
| `content` | Full text content of the item |
| `archived_at` | Date and time archived |

### 7.2 Limits
Maximum 10,000 rows. User is notified at > 9,000 rows. Limit is configurable in future versions.

### 7.3 Search
Full-text search across `original_path`, `parent_name`, and `content`. Results display as a flat list with all columns visible.

### 7.4 Restore
A user selects an archived row and chooses Restore. The system checks that every ancestor level in the target area has at least one open slot. If any level is full, `restore_blocked` fires:

> "Cannot restore: [level] in [path] is full (10/10). Free a slot first."

If all slots are available, the item is placed back at its original path and removed from archive.

---

## SECTION 8 — COLLABS

### 8.1 Structure
`Night_OS.collabs` is a full hierarchy workspace (category → note → detail → fragment).
- Categories group collab projects by theme
- Any node may be marked `is_project = true`
- Multiple nesting levels allow more than 10 total projects (10 × 10 = 100 notes per category)

### 8.2 Projects
- A node marked `is_project = true` is a project
- Projects also exist in `context.life.projects`
- When created anywhere, the `auto_assign_projects_context` rule (Section 10) ensures a matching context entry exists
- Projects are never subject to end-of-day archiving (they live in context)

### 8.3 Collaboration Invites
1. Project owner enters collaborator's email on the project node
2. System sends invitation email with a unique secure token (via Supabase auth)
3. Invitee clicks link → creates or logs into Night_OS account → access granted to project node and all its children
4. Status: `pending` → `accepted` or `declined`
5. Owner sees invite status on the project node

### 8.4 Shared Editing
- Both users see the same project content
- Changes sync within 15 seconds (client-side polling)
- Last-write-wins on concurrent edits

---

## SECTION 9 — SETTINGS

| Setting | Options | Default | Description |
|---|---|---|---|
| `theme` | `night`, `day` | `night` | Visual theme (dark / light) — aesthetic only |
| `view_mode` | `panel`, `standard` | `panel` | Grid cards or column list |
| `life_default` | `on`, `off` | `on` | Show/hide the Life default category in new_day and context |
| `notifications` | `on`, `off` | `on` | Enable end-of-day warnings |
| `account` | — | — | Email, password, display name |

Rules are managed from Settings > rules (see Section 10).

---

## SECTION 10 — RULES SYSTEM

Rules are predefined end-of-day instructions. Users toggle each on/off in Settings. The rules engine is designed for extensibility — new rules are added without modifying existing rule handlers.

| Rule ID | Name | Default | Fires |
|---|---|---|---|
| `notify_before_eod` | Notify Before End of Day | ON | 11:30 PM — show 30-min warning |
| `auto_append_matched` | Auto-Append Matched | ON | 11:59:59 PM — merge green-dot items into context |
| `auto_archive_unmatched` | Auto-Archive Unmatched | ON | 11:59:59 PM — archive red-dot items |
| `auto_assign_projects_context` | Auto-Assign Projects to Context | ON | At creation — create matching context entry for any project |
| `archive_journal_at_eod` | Archive Journal at End of Day | ON | 11:59:59 PM — archive all journal entries |
| `keep_empty_categories` | Keep Empty Categories | OFF | 11:59:59 PM — skip archiving empty new_day categories |

### EOD Execution Order
1. `notify_before_eod` (11:30 PM)
2. `auto_append_matched` (11:59:59 PM)
3. `auto_archive_unmatched` (11:59:59 PM, after append)
4. `archive_journal_at_eod` (11:59:59 PM)
5. `keep_empty_categories` (modifies archive step — empty categories skipped if ON)

### Extensibility
Each rule is a standalone handler function registered in a rules registry. To add a new rule:
1. Add rule definition to the registry (id, name, description, default, fire_time)
2. Add handler function
3. Add toggle to Settings UI

No existing rule handler is modified.

---

## SECTION 11 — PANEL VIEW

Night_OS supports two display modes, toggled in Settings:

| Mode | Description |
|---|---|
| `panel` | Items displayed as square grid cards in a responsive grid |
| `standard` | Items displayed as a single-column list with 4px colored left borders (Sage_DS standard) |

### Panel Navigation
- Clicking a card navigates **into** it: upper level collapses, lower level opens (folder metaphor)
- Breadcrumb at top always shows current path
- Back: breadcrumb click, `Ctrl+↑`, or `..` in command bar

### Node Reordering
- **Drag-and-drop** is the primary reorder method: drag any card or list row to a new position
- **↑/↓ buttons** on each card/row also reorder with a single click
- `Ctrl+Arrow` is a map navigator only — it has no reorder function (see Section 1.6 of Core)

### Ctrl+Arrow — Global Map Navigator
All four `Ctrl+Arrow` shortcuts work everywhere in Night_OS — inside settings, archive, journal, collabs, and any panel:
- `Ctrl+↑` — go up to parent level
- `Ctrl+↓` — go into first child of current position
- `Ctrl+←` — previous sibling (or previous contents area at the top row)
- `Ctrl+→` — next sibling (or next contents area at the top row)

The Map (Module 1) always reflects current position. When in a non-navigable area (archive, journal, settings), the map defaults to showing the `new_day` tree.

### Project Board
- Accessible by navigating to any project node or `Night_OS.collabs`
- Shows **3 levels simultaneously**: category → note → detail
- Maximum **9 items visible per level** (9 × 9 × 9 = 729 items on screen)
- Fragment level: click into a detail to expand fragments below
- Click any item to expand children inline; click out to collapse
- Text scales to fit: categories largest, fragments smallest
- Works in both panel and standard view mode

---

## SECTION 12 — DATABASE SCHEMA (Supabase)

Run the following SQL in the Supabase SQL editor to initialize Night_OS:

```sql
-- nodes: main content hierarchy (new_day, context, collabs)
CREATE TABLE nodes (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  parent_id uuid REFERENCES nodes(id) ON DELETE CASCADE,
  area text NOT NULL CHECK (area IN ('new_day','context','collabs')),
  level text NOT NULL CHECK (level IN ('category','note','detail','fragment')),
  name text NOT NULL DEFAULT '',
  content text DEFAULT '',
  sort_order int DEFAULT 0,
  is_project boolean DEFAULT false,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

-- journal_entries: daily journal (max 10 per user per day)
CREATE TABLE journal_entries (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  title text DEFAULT '',
  content text DEFAULT '',
  sort_order int DEFAULT 0,
  entry_date date DEFAULT current_date,
  archived_at timestamptz,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

-- archive_rows: flat archive table (max 10,000 rows per user)
CREATE TABLE archive_rows (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  original_path text NOT NULL,
  level text NOT NULL,
  parent_name text DEFAULT '',
  content text DEFAULT '',
  archived_at timestamptz DEFAULT now()
);

-- collab_invites: collaboration invitations
CREATE TABLE collab_invites (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  node_id uuid REFERENCES nodes(id) ON DELETE CASCADE NOT NULL,
  owner_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  invitee_email text NOT NULL,
  invitee_id uuid REFERENCES auth.users(id),
  status text DEFAULT 'pending' CHECK (status IN ('pending','accepted','declined')),
  invite_token text UNIQUE DEFAULT encode(gen_random_bytes(32),'hex'),
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

-- user_settings: per-user configuration
CREATE TABLE user_settings (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE UNIQUE NOT NULL,
  theme text DEFAULT 'night',
  view_mode text DEFAULT 'panel',
  life_default boolean DEFAULT true,
  rules jsonb DEFAULT '{
    "notify_before_eod": true,
    "auto_append_matched": true,
    "auto_archive_unmatched": true,
    "auto_assign_projects_context": true,
    "archive_journal_at_eod": true,
    "keep_empty_categories": false
  }'::jsonb,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

-- eod_log: end-of-day processing history
CREATE TABLE eod_log (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  processed_at timestamptz DEFAULT now(),
  merged_count int DEFAULT 0,
  archived_count int DEFAULT 0,
  summary jsonb DEFAULT '{}'::jsonb
);

-- Row Level Security
ALTER TABLE nodes ENABLE ROW LEVEL SECURITY;
ALTER TABLE journal_entries ENABLE ROW LEVEL SECURITY;
ALTER TABLE archive_rows ENABLE ROW LEVEL SECURITY;
ALTER TABLE collab_invites ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_settings ENABLE ROW LEVEL SECURITY;
ALTER TABLE eod_log ENABLE ROW LEVEL SECURITY;

-- RLS: users see only their own rows
CREATE POLICY "own_nodes" ON nodes USING (user_id = auth.uid()) WITH CHECK (user_id = auth.uid());
CREATE POLICY "collab_read" ON nodes FOR SELECT USING (
  user_id = auth.uid()
  OR id IN (SELECT node_id FROM collab_invites WHERE invitee_id = auth.uid() AND status = 'accepted')
  OR parent_id IN (
    SELECT n.id FROM nodes n
    JOIN collab_invites ci ON ci.node_id = n.id
    WHERE ci.invitee_id = auth.uid() AND ci.status = 'accepted'
  )
);
CREATE POLICY "own_journal" ON journal_entries USING (user_id = auth.uid()) WITH CHECK (user_id = auth.uid());
CREATE POLICY "own_archive" ON archive_rows USING (user_id = auth.uid()) WITH CHECK (user_id = auth.uid());
CREATE POLICY "own_settings" ON user_settings USING (user_id = auth.uid()) WITH CHECK (user_id = auth.uid());
CREATE POLICY "own_eod_log" ON eod_log USING (user_id = auth.uid()) WITH CHECK (user_id = auth.uid());
CREATE POLICY "collab_invites_access" ON collab_invites USING (
  owner_id = auth.uid() OR invitee_id = auth.uid() OR invitee_email = (SELECT email FROM auth.users WHERE id = auth.uid())
);

-- Indexes
CREATE INDEX idx_nodes_user_area ON nodes(user_id, area);
CREATE INDEX idx_nodes_parent ON nodes(parent_id);
CREATE INDEX idx_journal_user_date ON journal_entries(user_id, entry_date);
CREATE INDEX idx_archive_user ON archive_rows(user_id);
CREATE INDEX idx_collab_node ON collab_invites(node_id);
CREATE INDEX idx_collab_invitee_email ON collab_invites(invitee_email);
```

---

## SECTION 13 — NIGHT_OS NOUNS

| Word | Meaning |
|---|---|
| `life` | The default pre-built category in new_day and context; contains today, journal, plans, reviews, projects, portfolio, and drawing |
| `today` | A note within life containing daily quotes and gratitudes |
| `plans` | A note within life for goals, people, places, events, and things |
| `reviews` | A note within life for recording reactions to books, movies, experiences, and music |
| `projects` | A note within life for personal, work, and creative projects; also the type for all collab-enabled nodes |
| `portfolio` | A note within life for work and creative portfolio items |
| `drawing` | A note within life for sketch data |
| `collab` | A project node that has one or more accepted collaboration invitees |
| `invite` | A pending or accepted collaboration request sent by email and authenticated via token |
| `project_board` | A multi-level view of a project or collabs area showing category, note, and detail simultaneously (max 9 per level) |
| `eod` | End-of-day processing; the automated merge, archive, and rules execution at 11:59:59 PM |

---

## SECTION 14 — NIGHT_OS ERRORS

These extend Core Dictionary Section 4.

| Error | Meaning |
|---|---|
| `journal_full` | All 10 journal entry slots are occupied; user must free a slot before adding |
| `project_context_required` | A project node was created without a matching context entry (auto_assign_projects_context rule is ON) |
| `collab_not_accepted` | The collaboration invite has not been accepted; edit access is not yet granted |
| `eod_in_progress` | End-of-day processing is currently running; edits are temporarily locked |
| `archive_limit_reached` | The archive has reached 10,000 rows; oldest rows must be deleted before archiving more |

---

## SECTION 15 — MANIFEST

This is the Night_OS manifest as registered with SageDS. It is the source of truth SageDS uses to render all three modules for `B.Night_OS` without Night_OS shipping any UI of its own.

```yaml
manifest:
  address:        B.Night_OS
  type:           B
  extends:        Core
  definitions:
    life:           "Default pre-built category in new_day and context; contains today, journal, plans, reviews, projects, portfolio, drawing"
    today:          "Daily quotes and gratitudes; child of life"
    plans:          "Goals, people, places, events, and things; child of life"
    reviews:        "Recording reactions to books, movies, experiences, music; child of life"
    projects:       "Personal, work, and creative projects; collab-enabled nodes; child of life"
    portfolio:      "Work and creative portfolio items; child of life"
    drawing:        "Sketch data; child of life"
    collab:         "A node accepted into collaboration with another user"
    invite:         "A pending or accepted collaboration request sent by email and authenticated via token"
    project_board:  "A view of a project category showing note, detail, and fragment levels simultaneously (max 9 per level)"
    eod:            "End-of-day processing: automated merge, archive, and rules execution at 11:59:59 PM"
    journal_full:        "All 10 journal entry slots occupied; user must free a slot before adding"
    project_context_required: "Project node created without a matching context entry"
    collab_not_accepted:      "Collaboration invite not yet accepted; edit access not granted"
    eod_in_progress:          "End-of-day processing running; edits temporarily locked"
    archive_limit_reached:    "Archive has reached 10,000 rows; oldest rows must be deleted before archiving more"
  contents:
    - settings
    - new_day
    - context
    - journal
    - archive
    - collabs
  roles:          [owner]
  visibility:
    settings:       [owner]
    new_day:        [owner]
    context:        [owner]
    journal:        [owner]
    archive:        [owner]
    collabs:        [owner]
  data:
    source:       supabase
    endpoint:     <Night_OS Supabase instance — see index.html lines 544-545 for current keys>
  panels:
    new_day:        standard
    context:        standard
    journal:        standard
    archive:        standard
    collabs:        standard
    settings:       standard
```

**Note on user settings and system fragments:** Night_OS `settings` (theme, view_mode, EOD rules, etc.) are user-owned configuration stored in the `user_settings` table. They are content that belongs to the user and is editable by the owner role through normal dictionary functions. They are not SageDS system fragments. SageDS system fragments (manifest `visibility`, `panels`, and `data` tables above) are written at registration and are only changeable through the Core governance process (Section 5.3). No overlap.

---

*Night_OS Dictionary v0.3 — extends Core Dictionary v0.6*
