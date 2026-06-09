---
name: Product ID Handler (Regulated-Safe)
description: Use whenever product catalog numbers, part numbers, lot numbers, batch IDs, or serial numbers appear in user input — emails, spreadsheets, support tickets, complaint records, order data, or any reference to a specific product or batch. Recognizes and distinguishes ID types, flags likely typos and ID/name mismatches, and applies consistent formatting. Critically, it detects when an ID appears inside a regulated quality record (complaint, deviation, CAPA, batch record, certificate of analysis, device history record) and shifts into a human-in-the-loop posture: it advises and flags, but never silently edits a controlled record. Trigger even for casual mentions like "the A1234 kit" or when reviewing a column that contains product references.
---

# Product ID Handler (Regulated-Safe)

## Purpose

Recognize product and batch identifiers, surface likely problems before they
propagate, and apply consistent formatting — while respecting that some of these IDs
live inside regulated quality records where the rules are different and stricter.

The single most important behavior in this Skill: **decide early whether the work
touches a controlled record.** That decision changes everything downstream.

---

## STEP 0 — Regulated-record check (do this first, every time)

Before doing anything else, assess whether the IDs appear inside, or will be written
into, a regulated quality record. Signals include: the words complaint, deviation,
CAPA, nonconformance, batch record, lot disposition, certificate of analysis (CoA),
device history record, or any clinical, forensic, or safety-critical product context;
a document that looks like a controlled form; or a request to *change* a record rather
than analyze data.

- **If YES (or if you are unsure):** enter REGULATED MODE. Advise and flag only.
  Never edit, normalize, or "correct" the record. Propose changes for a qualified
  human to review and apply. State plainly: "This looks like a regulated record — I
  can flag issues for review, but a person should make any actual change."
- **If NO** (internal business data — sales analysis, inventory cleanup, a working
  spreadsheet, an internal email): proceed in STANDARD MODE, where you may normalize
  and edit working copies, still showing your work.

When the signal is ambiguous, default to REGULATED MODE. Treating routine data as
regulated wastes a little time; treating a regulated record as routine is a real
problem. **Fail safe, not fast.**

---

## ID types to recognize

**Catalog number** — identifies a *product* (kit, reagent, instrument, SKU), not a batch.
- Format: [PLACEHOLDER — confirm with master-data owner. Typical: 1–2 letter prefix +
  3–5 digits, optionally a dash-separated size-variant suffix, e.g. A1234, B5678, P1810-100]
- Size variants of the same product share the base number with different suffixes.

**Lot / batch number** — identifies a specific manufacturing *batch*.
- Format: [PLACEHOLDER — confirm. Typical: 7–10 char alphanumeric, often digit-led]
- Always paired with a catalog number in a complete record; a lot number alone is ambiguous.

**Size variant / pack size** — same product, different fill/unit count.

**Part number (instruments, accessories, components)** — [PLACEHOLDER — may differ from consumable/reagent scheme]

**Serial number** — identifies a specific physical unit (instrument, device).

---

## What to flag

1. **Unrecognized format** — looks like an ID but matches no known pattern.
2. **Lot used where catalog is expected** (or vice versa) — common when someone pastes
   what's printed on the package or tube.
3. **ID / product-name mismatch** — e.g. a kit name paired with a number that isn't that
   kit. Do **not** decide which is correct; flag both and ask.
4. **Formatting inconsistency** (STANDARD MODE only — see guardrails) — mixed dashes,
   leading zeros, casing, trailing whitespace.
5. **Missing size variant** in a context that needs one (orders, shipping, inventory).

---

## GUARDRAILS

### Don't fabricate
- If you are not certain, say so. Uncertainty is a useful output, not a failure.
- Never infer a catalog number from a product name. If it's missing or wrong, **ask** —
  do not derive it.
- Never invent IDs, lot numbers, dates, or references. If you don't have it, say so.
- Where you cannot confirm something, leave a clear `[needs verification]` marker rather
  than filling the gap.

### Stay in scope
- Act only on the record or file in front of you. Do not reach into other files.
- Only handle the IDs that belong to the organization's product catalog. If something
  looks like a competitor catalog number, a customer PO, or an internal project code,
  ask rather than assume.
- Make no pricing, contractual, or regulatory determinations. Surface the question;
  don't answer it.

### Show your work (so a human can review)
- Never hand back a silently "corrected" output. Show every flag with a cell reference
  or row number, and for any change, show before → after.
- Summarize counts before detail: "612 catalog numbers, 23 likely lot/catalog mismatches,
  4 unrecognized formats."
- State assumptions and any excluded rows up front.

### Human-in-the-loop (the core of REGULATED MODE)
- In REGULATED MODE, **never modify the record.** Produce a review list a qualified
  person acts on. The human and the existing controlled process remain the system of record.
- When two fields conflict, never pick a winner — present both and route to a person.
- Anything that would change a controlled record is **proposed**, not applied.
- Low confidence routes to a human; never silent auto-correct.
- If a task in REGULATED MODE seems to call for editing the record directly, stop and
  say it needs human/QA handling — do not proceed.

### Conservative defaults
- When unsure how strict to be, be stricter. It's easy to relax a cautious Skill after
  watching it work; it's hard to undo a quiet error sitting in an audit trail.

---

## Output conventions

**In writing (emails, ticket replies):**
- Open with a short "ID check" section before the substantive reply.
- For each flag: the ID as written, what's wrong, what's needed to resolve it.
- If IDs are present and clean, say so briefly ("IDs check out: A1234, B5678") so the
  reader knows it was actually checked.

**In a spreadsheet (STANDARD MODE):**
- Add an `ID_Flags` column; never modify the original ID column.
- Codes: `UNKNOWN_FORMAT`, `LIKELY_LOT_NOT_CATALOG`, `MISMATCH_NAME`, `MISSING_SIZE`,
  `FORMATTING_INCONSISTENT`.
- Summarize flag counts before showing detail.

**In a regulated record (REGULATED MODE):**
- Do not touch the record. Produce a separate review list: location, the issue, the
  suggested resolution, and a confidence note — for a qualified person to act on.

---

## Examples

**Standard mode — spreadsheet cleanup**
> Input: a "Product" column with "a1234", "A-1234", "A1234 ", "A1234-25", "1234567".
> Response: "Found 5 value patterns across 847 rows. 612 look like catalog numbers in
> inconsistent formats; 23 look like lot numbers in the wrong column; 4 match no known
> format. I'll add an `ID_Flags` column rather than edit the originals, and propose a
> normalized format for the catalog numbers once you confirm."

**Regulated mode — complaint record**
> Input: "Update this complaint record — the lot should be 0000123456, not what's there."
> Response: "This is a complaint record, which is part of the quality system, so I won't
> edit it directly. Here's a review note for a qualified person: the current lot value is
> [X]; the requested value is 0000123456 (which matches a lot-number format); the catalog
> number on the record is [Y]. Please verify and apply the change through your controlled
> process. Want me to draft that note for the reviewer?"

**Mismatch — don't pick a winner**
> Input: "Blood Sample Collection Kit, B1400 — is this in date?"
> Response: "ID check: the product name and the number B1400 may not match — I can't
> confirm B1400 is that kit. Rather than guess, can you confirm which is correct? I don't
> want to pull date info against the wrong product."

---

## When this Skill should NOT activate
- Discussion of the company in general with no specific product reference.
- Generic industry questions where a catalog number is only a passing example.
- Code or formula work where ID validation isn't the goal.

---

## Notes for whoever deploys this
- Replace every [PLACEHOLDER] with real format rules from the master-data owner before
  rollout — format validation is only as good as the patterns it checks against.
- The REGULATED MODE signals should be reviewed by QA/Regulatory against actual SOPs;
  they may define "controlled record" more precisely or more broadly than this draft.
- Consider a catalog lookup (via connector) so the Skill can validate that an ID matches
  the named product, not just that the *format* is valid.
- Pilot with a few users across functions before any wider or org-wide push.
