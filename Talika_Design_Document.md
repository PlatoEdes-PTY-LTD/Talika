# DynaDoc — Design Document

Oct 9, 2026 · @Debdeep Banerjee

## Overview

DynaDoc is a single-file, offline HTML app for tracking any flat dataset the user defines — there is no fixed schema. Every column either comes from an imported Excel file's own headers or is added by hand, and a row is just a set of values keyed to those columns.

It grew out of an earlier "Use Case Tracker" that hard-coded ID, Status, Description and similar fields; DynaDoc generalizes that into eight typed column kinds so the same file can track use cases, contacts, inventory, or any other tabular data.

It runs entirely in the browser: no server, no account, no network calls beyond loading the page. Opening the HTML file is the whole installation.

## Architecture

&#91;embedded content: architecture · import, state, UI, storage, export\]

DynaDoc is one HTML file with three `<script>` blocks: the Vue 3 runtime, the SheetJS (xlsx) library, and the app's own code — all bundled inline so the file works fully offline, with no build step and no server. Vue's reactive `columns` and `rows` arrays are the single source of truth: the table and the add/edit dialog render from them, Excel import writes into them, Excel export reads from them, and every change is mirrored to this tab's `localStorage` keys.

## Data model

Two reactive arrays hold everything; nothing else is hard-coded.

**`columns`** — one entry per column, in display order:

| Field | Meaning |
| --- | --- |
| `id` | Internal key, slugified from the name on creation |
| `name` | Column header shown in the UI |
| `type` | One of the seven types (see Column types) |
| `width` | Pixel width, set by dragging the column's resize handle |
| `hidden` | Present and `true` only when the column is hidden from the table |

**`rows`** — one entry per record:

| Field | Meaning |
| --- | --- |
| `_key` | Internal row identity, assigned on creation; not shown or exported |
| `values` | Object mapping each column `id` to that cell's value, shaped by the column's type |

A row has no required fields — remove every column and a row is just `{ values: {} }`.

## Column types

| Type | Stored as | Shown in the table | Edited with |
| --- | --- | --- | --- |
| Text | string | up to 3 lines, truncated | textarea |
| Date | `YYYY-MM-DD` string | localized date | date picker |
| Date & Time | `YYYY-MM-DDTHH:mm` string | localized date and time | date-time picker |
| Number | number or empty | localized number | number input |
| Set | string | colored pill | combo box offering values already used in that column, or a new one |
| Link | `{ label, url }` | clickable colored pill | label field + URL field |
| List | array of `{ label, url }` | colored pills, one per entry; link entries are clickable | repeatable label + URL rows, added and removed freely |

Set and List both let the user type a brand-new value — there's no fixed option set to maintain.

## Sheets (tabs)

Like an Excel workbook, one document holds several sheets, shown as tabs under the table. Each sheet has its own columns, entries, search, sort and filters. The **+** button adds a sheet; double-clicking a tab renames it (Excel's rules apply: at most 31 characters, none of `: \ / ? * [ ]`, and no two sheets share a name); tabs can be dragged to reorder; and a right-click menu offers Rename, Duplicate and Delete.

The shown sheet's data lives in the reactive `columns` and `rows` arrays, so every feature works on it directly. The other sheets keep their data on their own entry in `sheets` until they are switched to. Switching moves the old sheet's data and view state (search, sort, column filters, tab filters) onto its entry and loads the new sheet's.

### Tab filters

A sheet can filter its entries by the data in another sheet. The **Add Tab Filter** button, just after the search box, opens a bar above the table where each condition reads as a sentence:

*Where* [column in this sheet] [operator] [other sheet › column] *value* [Any, or one value from that column]

| Column type | Operators |
| --- | --- |
| Number | equals, is greater than, is less than |
| Date, Date & Time | equals, is before, is after |
| Text, Set, Link, List, ID | contains, equals (case-insensitive) |

With **Any**, an entry matches when it meets the condition for at least one value in the other column — so "greater than" means greater than the smallest of them. Picking one value compares against that value only. Conditions combine, so an entry must meet all of them; an unfinished condition is outlined and ignored. Complete conditions count toward the Clear Filters badge. Removing or hiding a column drops the conditions on it, and changing its type picks an operator that fits. Like other filters, tab filters stay with their sheet while switching tabs but are not saved.

## Excel import

Importing a file builds the schema from its own header row; there is no template to match. Each column's type is inferred from its cells, checked in this order:

1. **Link** — any cell in the column carries a real Excel hyperlink.
2. **List** — any non-empty cell contains a semicolon; it's split on semicolons into entries, and any entry that looks like a URL becomes a link entry.
3. **Date / Date & Time** — every non-empty cell is a native Excel date; Date & Time is used if any value carries a non-midnight time.
4. **Number** — every non-empty cell parses as a number.
5. **Set** — few distinct values relative to the row count (at most `min(8, ⌈rows ÷ 2⌉)`, with at least 3 populated rows).
6. **Text** — the fallback.

Every sheet in the file is imported as its own tab. Importing into an empty document applies immediately. Importing on top of existing data asks the user to **Replace All** (discard everything and use the file's sheets), **Add as New Sheets** (append each imported sheet as a new tab), or **Merge** (match imported sheets to existing ones by name, then columns by name, adding any new columns and backfilling blanks on existing rows; sheets with new names become new tabs).

## Excel export

Export writes one worksheet per tab, in tab order and named after the tab, with exactly the columns currently defined, in their current order, using each column's `name` as the header. Text, Number, Date and Date & Time cells are written as plain values; Link cells are written with a real Excel hyperlink; List cells are flattened to a semicolon-separated line of labels, so round-tripping an exported file re-infers it as a List column on the next import.

Export first opens a small dialog asking for the file name, pre-filled with the name of the last imported file (without its extension) or, if nothing was imported, `use-cases-<today's date>`. Characters that are invalid in file names are replaced, and `.xlsx` is added automatically.

## Multiple independent instances

Each browser tab or window gets its own data, so two files can be open and edited side by side without interfering. On load, the app resolves an instance id — from a `?instance=` URL parameter if one is present, otherwise from `sessionStorage` (private to that tab, and kept across a reload, but never shared with a new tab). All storage keys are namespaced with that id.

The toolbar's **New Window** button opens a second tab with a freshly generated instance id, guaranteeing independence even when a browser would otherwise copy session storage into a duplicated tab. The first instance to ever load on a given browser migrates in any data saved before multi-instance support existed, so existing work isn't lost; later instances start blank.

## Interface

The toolbar carries Add Entry, Add Column, Show/Hide Columns, Clear Filters, search, and Add Tab Filter; Import Excel, Export Excel, Reset All, and New Window sit at the right of the header.

Show/Hide Columns is a round eye button. It opens a dialog with a checkbox for each column; changes apply immediately, and Show All restores every column. When any column is hidden the button turns red, its icon becomes a crossed-out eye, and a badge shows how many columns are hidden, so a narrowed view is never mistaken for missing data. Hiding only affects the table: hidden columns keep their values, stay searchable, appear in the bulk-edit dialog, and are still exported. The add and edit dialog leaves them out by default, with a "Show hidden columns" checkbox to bring them back; their values are kept on save either way. Hiding a column clears any filter or sort on it, so rows are never filtered or ordered by something the user can't see.

The table supports:

- dragging a column's right edge to resize it (the width is remembered)
- dragging a column header to reorder columns
- clicking a header's arrow to sort ascending, descending, or clear
- a filter on every column, opened from its funnel icon: a range for Number, Date and Date & Time (min/max or from/to), a checklist of values (including Unset) for Set, and a case-insensitive "contains" match for Text, Link, List and ID. Filters combine, so a row must pass all of them. A filtered column's header turns amber, and the round Clear Filters button in the toolbar turns amber, shows how many filters are active, and clears them all. When filters leave no rows, the empty state says so and offers the same button.
- hover icons on a column header to change its data type, rename it (double-clicking the name also works), or remove it
- List and Link values drawn as colored pills, with link entries clickable
- edit and delete icons on each row, in a column pinned to the right edge so it stays visible while the table scrolls horizontally
- a table that fills the page width; when the columns are wider than the page, only the table area scrolls horizontally

Add and edit use the same dialog, rendering one field per column based on its type, with a small badge next to each label naming that type. Save Entry saves without closing the dialog; after adding, the dialog switches to editing the new entry so a second save updates it rather than adding a duplicate. Save & Next saves and moves to the next entry in the table's current order, so the user can walk through records one after another; a counter shows the position. Close dismisses the dialog without saving. Delete entry, remove column, reset all, and a conflicting import are each confirmed with the app's own modal — the browser's native `confirm()` dialog is not used, since it proved unreliable in some embedded preview contexts. Brief toast messages confirm actions like saving, deleting, or exporting.

Changing a column's type converts each stored value through its text form: Number keeps only text that parses as a number, Date and Date & Time keep only text that reads as a date, a Link becomes a one-entry List, and a List becomes a Link from its first entry. The change dialog counts the values that can't convert and will be cleared before the user confirms, and the column's filter is reset.

## Persistence

Everything is stored in the browser via `localStorage` and `sessionStorage`; nothing leaves the machine.

| Key | Scope | Holds |
| --- | --- | --- |
| `dynadoc-sheets-v1::<instance>` | per tab | the whole document: every sheet's name, columns and rows, and which sheet is shown |
| `use-case-tracker-columns-v2::<instance>` | per tab | column definitions saved before sheets existed; loaded as the first sheet |
| `use-case-tracker-rows-v2::<instance>` | per tab | row data saved before sheets existed; loaded as the first sheet |
| `use-case-tracker-filename-v2::<instance>` | per tab | last imported file's name |
| `dynadoc_instance_id` | per tab (sessionStorage) | that tab's instance id |
| `use-case-tracker-migrated-v2` | browser-wide | one-time flag marking legacy data as migrated |

Closing the tab doesn't clear anything — `sessionStorage` only drops the instance pointer, not the data itself, which stays in `localStorage` under that instance's keys.

## Limitations and possible next steps

- Type inference is a heuristic: a text column containing semicolons is read as a List, and a free-text column with few distinct values is read as Set. The type can be changed afterwards, but a conversion clears any value that doesn't fit the new type.
- No cross-device sync — data lives in one browser's storage, not shared automatically between machines.
- Filters, including tab filters, are not saved; they reset when the page reloads.
- No undo beyond the confirmation prompts on delete, remove, and reset actions.

Reasonable next steps: CSV import alongside Excel, and an explicit export/import of the whole workspace (columns, rows, and settings together) for moving data between browsers.
