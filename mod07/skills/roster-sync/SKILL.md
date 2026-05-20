---
name: roster-sync
description: >
  Syncs formatting from the Master Template sheet to all period roster sheets in
  Rosters Updated.xlsx, preserving student names and assignment data. Use this skill
  whenever the user says things like "sync the template", "update all rosters from
  the template", "push my formatting changes to all periods", "the master template
  changed", or any time formatting in one sheet should propagate to the others.
  Also trigger when the user asks to change colors, fonts, borders, legend text,
  or assignment labels across all sheets at once.
---

# Roster Sync Skill

## What this skill does

Reads formatting from the **Master Template** sheet in `Rosters Updated.xlsx` and
stamps it onto every period sheet (1st through 6th), while leaving each sheet's
student names and any observation data already entered completely untouched.

## File location

```
/Users/user/Documents/Claude/Projects/6.00 | Narrative Medicine/Rosters Updated.xlsx
```

## Sheet structure

- **Master Template** — leftmost tab. Contains all formatting, assignment labels,
  and legend rows. Student rows are placeholders (`[Student Name A]`, etc.).
- **Period sheets** — `1st - Unit 6`, `3rd - Unit 6`, `4th - Unit 6`,
  `5th - Unit 6`, `6th - Unit 6`. Each has real student names starting at row 3.
- **No Pass List** — leave this sheet untouched.

## How to sync

Use Python + openpyxl. Follow these steps precisely:

### Step 1 — Read the Master Template

Extract the following from the Master Template:

- **Row 1** (title row): fill color, font (size, bold, color), alignment, border, merge state
- **Row 2** (assignment row): fill color, font, alignment, border, and the assignment
  label values (e.g. `6.01`–`6.10`) in columns 2–11
- **Student row formatting**: read the first two non-placeholder student rows (rows 3 and 4)
  to get the alternating fill colors and border style. These two rows define the
  even/odd pattern for ALL student rows.
- **Legend rows**: starting from the first row after the blank gap below the
  placeholder students, collect all rows whose column-A value contains ` | `.
  Capture: the text, fill color, font size, and whether italic/bold.
- **Column widths**: capture `column_dimensions` for columns A–K.

### Step 2 — Apply to each period sheet

For each period sheet (skip Master Template and No Pass List):

1. **Row 1 — Title**: Apply fill, font, alignment, border from template.
   Update the text to match the period: e.g. `1st Period Unit 6`.
   Re-apply the merge across columns 1–11.

2. **Row 2 — Assignments**: Apply fill, font, alignment, border.
   Write the assignment labels from the template into columns 2–11.
   Keep "Assignment" in column 1.

3. **Student rows** (row 3 through the last non-empty name row):
   - Collect all student names first by scanning column A for non-None values
     that don't contain ` | ` (so legend rows aren't accidentally included).
   - Apply alternating fill (even index = color A, odd index = color B) and border
     to all 11 columns of each student row.
   - Do NOT change column A values (names) or any observation data already in
     columns B–K.

4. **Legend rows**: 
   - Find the first blank row after the last student name. Write a blank separator.
   - Write the legend entries from the template into column A starting one row below.
   - Apply fill, font. No cell borders on legend rows.
   - Clear any old legend rows that extend beyond the new legend length.

5. **Column widths**: Apply from template.

6. **Print settings** (preserve these):
   - `print_title_rows = '1:2'`
   - Footer: `oddFooter.center.text` = compact legend key (auto-build from legend entries:
     take the part before ` — ` in each entry, join with `  |  `)

### Step 3 — Save

Save back to the same file path. Confirm to the user which sheets were updated
and note any sheets skipped.

## What NOT to touch

- Student names in column A
- Any observation codes already entered in columns B–K
- The No Pass List sheet
- The Master Template sheet itself

## When the user wants to change formatting

Direct them to edit the Master Template sheet first, then trigger this sync.
Common changes they might want:
- Different colors → change fills in rows 1, 2, or the placeholder student rows
- Different assignment labels → change values in row 2 of the template
- Updated legend entries → edit the legend rows in the template
- Font size changes → edit fonts in rows 1 or 2

After any such change, run this sync to push it everywhere.

## Error handling

- If Master Template sheet is missing: tell the user and stop. Don't guess.
- If a period sheet is missing: skip it and note it in the summary.
- If the file is open in Excel/Numbers: openpyxl will likely error on save.
  Tell the user to close the file first and re-trigger.
