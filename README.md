<p align="center"><img src="webtalika-icon.svg" alt="WebTalika icon" width="96" height="96"></p>

# WebTalika

*The List*

**WebTalika**, also known as **Talika**, is a free, open-source web app for editing spreadsheets in the browser. It is a dynamic tabular document creator for easy maintenance of Excel files. Import a spreadsheet or define your own columns, edit the data in a clean table, and export it back to Excel. There is no fixed schema, so you can track anything: use cases, contacts, inventory, tasks. Everything runs in the browser from a single file, with no server, no account, and no build step.

[Open WebTalika](https://platoedes-pty-ltd.github.io/Talika/index.html)

## Features

- **Schema-free tables.** Columns come from an imported Excel file's headers or are added by hand. Rows are just values keyed to those columns.
- **Eight column types.** Text, Date, Date & Time, Number, Set, Link, List, and ID.
- **ID columns.** You set a prefix when you create the column. Values look like `UC-1`, `UC-2`. A new entry gets the next number automatically.
- **Multi-select and bulk edit.** Tick entries (Shift-click selects a range), click **Edit selected**, then choose which columns to update.
- **Excel import and export.** Column types are inferred on import. Export asks for a file name, pre-filled from the file you imported.
- **Sheets (tabs).** Like Excel, one document holds several sheets, shown as tabs under the table. Each sheet has its own columns, entries, search, sort, and filters. Click **+** to add a sheet, double-click a tab to rename it, drag tabs to reorder them, and right-click a tab to rename, duplicate, or delete it. Import reads every sheet in the file, and export writes every sheet as its own Excel worksheet. Tab filters let one sheet filter its entries by the data in another (see below).
- **Column tools.** Resize, drag to reorder, rename (double-click the name), change data type, and remove.
- **Show and hide columns.** The round eye button in the toolbar opens a dialog where you tick the columns to show. When any column is hidden, the button turns red and shows how many are hidden. Hidden columns keep their data, stay in the bulk edit dialog and the Excel export, and are remembered after a reload. The Add/Edit Entry dialog leaves hidden columns out; tick **Show hidden columns** at the top of the dialog to edit them.
- **Sort and filter.** Click a header arrow to sort. Every column has a filter (funnel icon): a range for Number, Date, and Date & Time; a value checklist for Set; "contains" text for Text, Link, List, and ID. A filtered column's header turns amber. The round funnel button in the toolbar turns amber, shows how many filters are active, and clears them all with one click. A global search box covers everything.
- **Tab filters (filter across sheets).** Click **Add Tab Filter** (the funnel with a plus, just after the search box) to filter this sheet's entries by a column from another sheet. A bar opens above the table, and each condition reads as a sentence: *Where* [column in this sheet] [operator] [other sheet › column] *value* [Any, or one value]. The operators depend on the column's type: **contains** and **equals** for Text, Set, Link, List, and ID; **equals**, **is greater than**, and **is less than** for Number; **equals**, **is before**, and **is after** for Date and Date & Time. With **Any**, an entry matches if it meets the condition for at least one value in the other column. Pick a single value from the other column instead to compare against only that value. Click **+ Add condition** for more lines; an entry must meet every condition. A condition with no column chosen yet has a dashed outline and is ignored. Complete conditions count toward the Clear Filters badge, and each sheet keeps its own tab filters when you switch tabs.
- **Colored pills** for Link and List values; link entries are clickable.
- **Icon-only toolbar.** The toolbar buttons show icons. Hover a button to see its name.
- **Table scroll.** The table scrolls on its own, up and down and left and right. The header row stays at the top. The horizontal scroll bar stays in view.
- **Pinned actions column.** Edit and Delete stay visible at the right edge while the table scrolls horizontally.
- **More-columns dots.** The header of the actions column shows a three-dots button when more columns are hidden to the right. Click it to scroll right. It disappears at the end of the table.
- **Multiple independent instances.** Each browser tab has its own data. The **New Window** button opens a fresh, separate copy.
- **Download App.** The **Download App** link in the header of the [home page](https://platoedes-pty-ltd.github.io/Talika/index.html) downloads the app as `talika.html`, so you can keep it on your disk and run it offline. Your data is not in that file; use Export for your data.
- **Fully offline.** Vue 3 and SheetJS are bundled inline in the one HTML file.

## Getting started

1. Download `talika.html`.
2. Open it in Chrome, Edge, Firefox, or Safari.
3. Click **Import Excel** (up-arrow icon) to build a table from a spreadsheet, or **Add Column** to start from scratch.
4. Click **Add Entry** (plus icon) to add rows.

## Column types

| Type | Stored as | Notes |
| --- | --- | --- |
| Text | string | Multi-line, truncated in the table |
| Date | `YYYY-MM-DD` | Date picker |
| Date & Time | `YYYY-MM-DDTHH:mm` | Date-time picker |
| Number | number or empty | Localized display |
| Set | string | Combo box offering values already used in the column, or a new one |
| Link | `{ label, url }` | Clickable pill |
| List | array of `{ label, url }` | One pill per entry; add or remove entries freely |
| ID | `PREFIX-number` string | Prefix set when the column is created; next number fills in automatically |

The Add/Edit Entry dialog shows each field's type next to its label. **Save Entry** saves and keeps the dialog open, so you can keep editing the same entry. When editing, **Save & Next** saves the entry and opens the next one in the table's current order (search, filters, and sort included); a counter such as "3 of 10" shows where you are. **Close** shuts the dialog without saving.

### Changing a column's type

Hover a column header and click the swap-arrows icon. Each value is converted through its text form. The dialog tells you how many values can't convert and will be cleared before you confirm.

## Excel import and export

On import, each column's type is inferred in this order:

1. **Link**: any cell has a real Excel hyperlink.
2. **List**: any non-empty cell contains a semicolon. Cells are split on `;`, and URL-like entries become links.
3. **ID**: every non-empty cell looks like `PREFIX-number` with one shared prefix that starts with a letter, such as `UC-12`.
4. **Date / Date & Time**: every non-empty cell is a native Excel date.
5. **Number**: every non-empty cell parses as a number.
6. **Set**: few distinct values relative to the row count.
7. **Text**: the fallback.

Every sheet in the file is imported as its own tab. Importing into a non-empty document lets you **Replace All** sheets, **Add as New Sheets**, or **Merge** (sheets are matched by name, then columns by name; sheets with new names become new tabs).

On export, each tab becomes a worksheet with the tab's name, in tab order. Columns are written in their current order. Link cells keep their hyperlinks, and List cells are flattened to a `; `-separated string, so they round-trip back to a List on import.

## Data and privacy

All data is stored in your browser's `localStorage`, namespaced per instance. Nothing is sent anywhere. Clearing site data for the page removes your tables, so export to Excel to keep a copy.

## Tech

- Vue 3 (Composition API), bundled inline
- SheetJS (`xlsx`), bundled inline
- Plain HTML and CSS, no build tooling

## Limitations

- No cross-device sync. Data lives in one browser's storage.
- Type inference is a heuristic, so a column may be guessed wrong. Use the change-type tool to fix it.
- Filters, including tab filters, are not saved. They reset when the page reloads.
- No undo beyond the confirmation prompts on delete, remove, and reset.

## Ideas for next steps

- CSV import alongside Excel
- Export and import of the whole workspace (columns, rows, settings) for moving data between browsers

## License

[MIT](LICENSE). The bundled Vue 3 (MIT) and SheetJS (Apache-2.0) libraries keep their own licenses.
