# 01 · Design (before pixels)

WHEN: any screen, layout, colour, type, spacing, icon, empty/loading/error state, or generating UI.
LOAD: this file first on this stack. Where the file lives: [03-file-structure.md](03-file-structure.md). Tokens: [11-ui.md](11-ui.md).
RELATED: 06 (metadata) · 15 (focus, names, XSS) — open only if the task is also that topic · product art direction in `docs/design/` when it exists ([agents/09-docs.md](../../agents/09-docs.md) *docs/design/*) — wins on composition and typography.
SCOPE: how a human reads and acts. Not React trees. Not a brand book for one company.

A screen that looks "modern" and cannot be used is a defect. Decoration after the five questions below are answered.

---

## Five questions (every view)

Stop if any answer is missing. Then type and colour, not before.

1. Where am I?
2. What am I looking at?
3. What matters on this screen?
4. What is the one next action?
5. What happens after that action?

MUST: one primary heading. MUST: one primary action (or none — a read-only page).
MUST NOT: two `h1`. MUST NOT: two equal CTAs ("Get started" + "Learn more" as twins).
MUST NOT: internal names on the glass (`stage_type`, `tenant_id`, `error_code` as the headline). The API `error_code` is for the client; the human sees `message`.

---

## AI-slop (reject the screen)

If several of these are true, start over. This is the default look of an image model, not a product.

- Inter / Roboto / "system-ui" as the only type, or a different font per section
- Purple / indigo gradient, glow, glass, aurora, mesh, floating orbs
- Three equal feature cards, coloured icon circles, lucide-everywhere
- Hero: oversized line + two sentences of filler + two buttons
- `rounded-3xl` / pill on every card; mix of 4px and 24px radii
- Fake social proof (4.9 stars, "10k teams", stock avatars)
- Template dashboard: dark sidebar, four stat cards, area chart, "recent activity"
- Every control is primary; every block has a shadow
- Emoji as the icon set; lorem; "Unlock your potential"
- Dark mode = invert + keep the purple

MUST NOT: ship a screen that would fit a generic "SaaS landing" or "AI dashboard" template.
MUST NOT: add a second accent (gradient, thick border, *and* drop shadow) on the same element — one emphasis per emphasis.

What **does** catch the eye, if the five questions are already yes:

- A short hierarchy: display → title → body → meta. You can squint and still see it
- Alignment to a grid; leftover floating islands are slop
- One accent used sparingly (the primary button, one highlight, not the whole page)
- Density that matches the job: a form breathes; a table is tight; marketing may be large
- Real states (below), not a spinner in an empty viewport

---

## Ratios and scale

MUST: spacing from a 4px or 8px scale (`4 8 12 16 24 32 48 64`). MUST NOT: `13px` / `15px` gaps "to look tight."

Type: one family for UI (plus optional mono for ids, dates, numbers — not for body). Modular scale (e.g. 1.25). Body ~16px. Line-height ~1.45–1.6 for reading, ~1.15–1.25 for display. Measure ~45–75 characters.

MUST: a content column `max-width`. Full-bleed only for media or a marketing band.
MUST: control target ≥ 24px; ≥ 44px on the main mobile action.
MUST: one radius token for controls and cards (about 8–12px). MUST NOT: pill inputs next to sharp tables without a reason.

Media: pick 1:1, 4:3, 3:2, or 16:9 and crop. MUST NOT: stretch. MUST NOT: a different ratio every card.

Space between **sections** > space between **heading and body** > space between **label and field**. If those three are equal, the page is a stack of same-sized boxes (slop).

---

## Colour

Hex lives in tokens (a later file). Components see names (`bg-surface`, `text-primary`, `border-subtle`). MUST NOT: hex in a component.

**60 / 30 / 10** of the view, not of the brand PDF:

- ~60% page surface
- ~30% muted surface, borders, secondary text
- ~10% accent **and** semantic (success / warning / danger)

MUST NOT: "make it ours" by washing the whole UI in purple, teal, or a gradient.
MUST NOT: a new hue for hover. Hover / focus / active / disabled are the **same** token, darker, lighter, or quieter.
MUST: body text vs surface strong enough to read in daylight (WCAG AA as the floor). Grey-on-grey is a defect.
MUST: status is colour **plus** text or icon. Colour alone fails.

Accent is the one thing that should move. If four buttons and a chart and a badge are all accent, none of them is.

---

## Four states

Every list, form, and panel has all four. Same skeleton; different content.

- **Loading** — skeleton that matches the layout. A spinner only for a short in-place wait. Long work: progress words, not a blank page.
- **Empty** — what is missing, what to do next. Illustration optional; type + one action is enough.
- **Error** — on the field when it is a field; a banner when it is the page. Pair icon/text. MUST NOT: a left rainbow stripe as the only language.
- **Success** — inline, or a confirmation block after a large write. MUST NOT: a toast stack for every tiny save.

MUST NOT: a page that is only a centered spinner for more than a moment.
MUST NOT: invent a metric the product does not have.

---

## Universal UX (short)

1. The user can say the job of this screen in one sentence.
2. The next action is findable without decoration.
3. Destroying data names the noun (`Delete order`), asks, and is reversible when the product allows.
4. Do not make them remember: show which record, which step, which tenant label the product already has.
5. Keyboard: visible focus, tab order, Enter submits the intended control.
6. A modal has a close. A wait has an error path. MUST NOT: a trap.
7. Labels are visible. Placeholder is not a label. MUST NOT: "click here."
8. Same control looks the same on every screen (one primary button).
9. MUST NOT: disable paste on passwords. MUST NOT: autoplay sound.
10. Engineering strings stay off the glass.

Copy: product language. MUST NOT: `lorem`, `TODO`, `Submit`, `An error occurred`. A button is a verb; add the object when the object is not obvious (`Save draft`).

---

## Done

- [ ] Five questions answered before colour
- [ ] Not a generic AI landing or dashboard template
- [ ] Spacing and type on a scale; one radius; one accent
- [ ] 60/30/10; no hex in the component; status is not colour-only
- [ ] Loading / empty / error / success exist
- [ ] One `h1`, one primary action, visible focus, human copy
