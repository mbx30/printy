# Print Job Schema & Rules

## Notion Database

- **Print Jobs database ID**: `32c9cb079ddb807eba29dd54fee53aac`
- **Clients database ID**: `2ee9cb079ddb809d81f2fa9a9c2a35d3`
- Always verify the live schema before writing — option sets (Size, Paper Type, Done) can change.
- **With Notion API access**: fetch the database schema at session start to get the current option sets. Inject the live values as the authoritative list, superseding anything in this document.
- **Without API access**: if a value you're about to write is not listed in this document, ask Michael: "Is '[value]' a valid [Size/Paper Type] option in Notion right now?"

## Data Model

Write only these properties using exact live schema names (case-sensitive):

| Property | Type | Required | Rules |
|----------|------|----------|-------|
| Job | Title | Yes | Job description — see Job Title Rules below |
| Client | Relation (limit 1) | Yes | Resolve or create — see Client Resolution |
| Paper Type | Multi-select | Yes | Resolve via decision tree below |
| Size | Select | Yes | Must match an existing option exactly |
| Quantity | Number | Yes | Plain integer |
| Cost per | Number (dollar) | No | Per-unit only; if given as batch total, confirm per-unit before writing |
| Rcvd | Date | No | Defaults to today (America/Los_Angeles), date only |
| Promised | Date | Yes | Defaults to next business day 4:00 PM (America/Los_Angeles) |
| Done | Status | Yes | Default: Not started |
| File | Files | No | Attach or note link in job page body |

⛔ **THESE PROPERTIES DO NOT EXIST — never write to them**: Notes, Total, RCVD ON, Instructions, Comments.
- Operational notes → job page **body** (not a property)
- Batch total → confirm per-unit with Michael, then write to "Cost per"
- Received date → "Rcvd" (not "RCVD ON")

## Job Title Rules

The Job title carries every spec not covered by another column.

**Must include**: rounded corners, lamination, mounts, any finishing/premium option that changes price, any production/billing detail without its own column.

**Must exclude (hard ban)**: Size, Quantity, Client name. Write "Regular Posters", not "11x17 Regular Posters" and not "200 Regular Posters".

**Always Title Case.**

**IRON RULE: If you catch yourself typing a number or a dimension in the Job field, stop and remove it.**

**Examples from real jobs**:
- "Laminated Menus" ✅ (lamination is a premium add-on)
- "DS Laminated Food Menu" ✅ (double-sided + laminated)
- "Menus Doublesided" ✅
- "Menu Print; Lamination" ✅ (billing add-on belongs in title, not a Notes property)
- "Framed Poster Print; Sent to ARC" ✅ (ARC = outsourced vendor note)
- "All Day Menus; Laminated" ✅
- "8.5x14 All Day Menus (35)" ❌ (size AND quantity in title)
- "200 11x17 Venue Posters" ❌ (quantity AND size in title)
- "Loma Club Laminated Menus" ❌ (client name in title)
- "11x17 Menus" ❌ (size in title)
- "50 Menus" ❌ (quantity in title)

### Music Box Naming Convention

| Job Type | Size | Paper Type |
|----------|------|------------|
| Venue Poster | 11x17 | Text |
| Signing Poster | 11x17 | Cardstock |
| Pole Poster | 15x30 | (in Paper Type, never title) |
| Bookmark | 4x9 | Cardstock |
| Handbill | 4x6 | Text |
| Signage | one-off | as specified |

- Title format: `[Job Type] — [Month]` for monthly batches; `[Job Type] — Add-on` for mid-cycle; `[Job Type] — [Descriptor]` for one-offs.
- 15x30 jobs: title is "Pole Poster" — never "15x30 Poster".
- "Translucent plastic" / "backlit" goes in Paper Type, never the title.
- Designs received at end of a month are usually for the next month.

## Date Rules

### Rcvd (received date)
- Default: today (America/Los_Angeles). No confirmation needed.
- Date-only value (no time).

### Promised (required every time)
- Default: next business day at 4:00 PM (America/Los_Angeles).
- Never same-day by default.
- Business days: Monday–Friday. Skip Saturday and Sunday.
- If the email states a specific pickup time, use that.
- Always a datetime: date + 4:00 PM unless user states a specific time.

**Ambiguous deadline decision table** — apply before writing Promised:

| User says… | Promised default |
|------------|-----------------|
| "no rush" / "whenever" | Next business day at 4:00 PM |
| "for today" / urgency language | Next business day at 4:00 PM (NEVER same day) |
| "for the weekend" | Monday at 4:00 PM |
| Job placed on Friday | Monday at 4:00 PM (skip weekend) |
| Specific date given | That date at 4:00 PM |

**IRON RULE: Never set Promised = today by default, regardless of urgency language.**

## Paper Type Decision Tree

1. Explicitly stated in email → use it.
2. Client page has a Default Paper Type → propose it and require Michael's confirmation before writing.
3. Neither → ask Michael.

**Synonym normalization** — accept these common variants:
- "textweight" / "text weight" / "text stock" → Text
- "card stock" / "cardstock" → Cardstock

**Never invent a Paper Type not in the live schema. If unsure → ask Michael.**

**Known mappings from real jobs**:
- "Regular posters" → Text
- "Translucent plastic" / "backlit" → Backlit display
- Handbills → Text (default)
- Church bulletins / prayer sheets → Smooth
- Polyester = waterproof plastic menus (common for restaurants)
- Laid ivory = specialty paper for Barra Oliba, Vinarius, Loré Experiential
- 0.040 Styrene = outdoor rigid board (A-frames, outdoor signs)
- Coroplast = corrugated plastic (outdoor A-frames)
- Foamcore = foam board (event signage)
- Satin / Gloss = Remedy Pharmacy fliers and inserts
- OnlineLabels = Pappalecco label sheets
- Cardstock = default for cards, flyers, postcards

## Size Normalization

The live Size option set has inconsistent spacing — match exactly:
- `5 x 8` and `5.5 x 8.5` **include spaces**
- `8.5x11`, `8.5x14`, `4.25x11`, `4x6`, `11x17` do **not**

⚠️ **Critical spacing quirks — these exact strings must be used**:

| User types | Canonical value |
|------------|----------------|
| `5x8` / `5 x 8` | `5 x 8` (spaces required) |
| `5.5x8.5` / `5.5 x 8.5` | `5.5 x 8.5` (spaces required) |
| `4x6` / `4 x 6` | `4x6` (no spaces) |
| `11x17` / `11 x 17` / `11×17` | `11x17` |
| `8.5x11` / `letter` | `8.5x11` |
| `A5` | `A5` |
| `A4` | `A4` |

**If no exact option matches after normalization → ask Michael. Never invent a new size option.**

## Client Resolution

- Client exists in Notion → relate it.
- Client does not exist → create a new Client page immediately (no confirmation), then relate it.
- After creating a new client: remind Michael to add a logo and cover image.

### Client Page Structure (canonical layout)

Every client page uses this exact structure (H2 headings):
1. "Table of Contents" + table of contents block
2. "Mapping / Defaults" + 4-column table: Job Type / Size / Paper Type / Notes. New clients get one placeholder row: TBD / TBD / TBD / "Placeholder — update when first job is placed."
3. "Print History" heading
4. "Print Recipes" heading

### Known Client List (current as of June 2026)

Siamo Napoli, Kettner Exchange, Whaling Bar, Loma Club, Queenstown Public House, Barra Oliba, Prohibition Bar, Firehouse, Vinarius, Hob Nob Hill, Music Box, Coco Maya, Crudo, Harbor City Church, La Jolla Tennis Lessons, Bandar, Remedy Pharmacy, Belly Up (MBX), Grass Skirt, Queenstown Bistro, Vin De Syrah, Captain's Quarters, Devil's Dozen, The Waverly, Pappalecco, Coastlands Community Church, Dunedin, Carté, Civico 1845, Isola, Maison Amani, Buona Forchetta, MEZE, Rubicon Deli, Clean Edge Technologies, Adelman Fine Art, Havana 1920, FMS Solutions, City Life Church, Rancho Del Mar Association, Dropkick Coffee, Go Postal, El Chingon, Pecoraro Painting, Loré Experiential, Harbor Breakfast, Colrich, Nolita Hall, Bencotto, Park 101, Bri Rahoy (Grind & Prosper), Carnitas Snack Shop, Pazza Market & Cucina, Jeevan Productions

## Confidence Model

Tag each extracted value before writing:

- **Certain** — explicitly stated and unambiguous → use it
- **Probable** — strongly implied but could be wrong → confirm with Michael before writing
- **Unknown** — not present → apply default rule or ask

Never promote Probable → Certain without confirmation. "Same as last time" with no stated specs is Unknown — ask, do not copy a prior job.

**These phrases ALWAYS = Probable, never Certain. Always confirm before writing:**
- "same as always" / "same as last time" / "usual"
- "standard" / "regular stock"
- "you know what we use"
- Paper omitted + client has a default → Probable (propose it and require confirmation)

Confirmation format: "I see [Client] usually uses [Paper Type] — confirm?"
Do NOT write the value until Michael replies yes.

## Duplicate Check

Before creating **any** row, scan recent jobs (~last 30 days) for the same Client + Size + Quantity + Paper Type + Promised. If a likely duplicate exists: ask Michael, "A similar job exists (received [date]). Create a new one anyway?"

**Extra vigilance for recurring clients** — do not skip the scan even if Michael says "same as last week":
- Harbor City Church (weekly bulletins)
- Music Box (monthly poster batches)
- Any client + job type combination you've seen in the last 30 days

If a match exists → ask before creating. Never silently create a second row for a recurring job.

## Forbidden Patterns

- Creating a row while any required field is missing
- Guessing a value not in the allowed option set
- Putting Size, Quantity, or Client name in the Job title
- Treating "same as last time" as license to copy specs without confirmation
- Setting a same-day Promised date by default
- Writing to any database other than Print Jobs (`32c9cb079ddb807eba29dd54fee53aac`)
- Writing to a property name not in the live schema
