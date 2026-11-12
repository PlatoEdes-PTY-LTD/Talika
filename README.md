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
- **Index tab.** Every document starts with an **Index** tab, always the first tab. It lists every other tab in the document, with two columns: **Name** and **Description** (both Text). Rows are added, renamed, reordered, and removed automatically as you add, rename, move, duplicate, and delete tabs. Click a name to open that tab. Edit a row to write its description; the name is read-only and follows the tab's name. You can add more columns to the Index, but Name and Description can't be renamed, retyped, or removed, and the Index tab can't be renamed, moved, duplicated, or deleted. The Index is exported as its own worksheet; when you import a file that has one, its descriptions fill in this document's Index instead of adding a second Index tab.
- **Column tools.** Each column header has a small ▾ menu button. Hover over it with a mouse (or tap it on a touch screen) to open the column menu: **Filter…**, **Change type**, **Rename**, and **Remove column**. Double-clicking the name also renames it. Drag the header to reorder and drag its right edge to resize.
- **Show and hide columns.** The round eye button in the toolbar opens a dialog where you tick the columns to show. When any column is hidden, the button turns red and shows how many are hidden. Hidden columns keep their data, stay in the bulk edit dialog and the Excel export, and are remembered after a reload. The Add/Edit Entry dialog leaves hidden columns out; tick **Show hidden columns** at the top of the dialog to edit them.
- **Sort and filter.** Click a header arrow to sort. Every column has a filter (**Filter…** in the column menu): a range for Number, Date, and Date & Time; a value checklist for Set; "contains" text for Text, Link, List, and ID. A filtered column's header and its ▾ menu button turn amber. The round funnel button in the toolbar turns amber, shows how many filters are active, and clears them all with one click. A global search box covers everything.
- **Tab filters (filter across sheets).** Click **Add Tab Filter** (the funnel with a plus, just after the search box) to filter this sheet's entries by a column from another sheet. A bar opens above the table, and each condition reads as a sentence: *Where* [column in this sheet] [operator] [other sheet › column] *value* [Any, or one value]. The operators depend on the column's type: **contains** and **equals** for Text, Set, Link, List, and ID; **equals**, **is greater than**, and **is less than** for Number; **equals**, **is before**, and **is after** for Date and Date & Time. With **Any**, an entry matches if it meets the condition for at least one value in the other column. Pick a single value from the other column instead to compare against only that value. Click **+ Add condition** for more lines; an entry must meet every condition. A condition with no column chosen yet has a dashed outline and is ignored. Complete conditions count toward the Clear Filters badge, and each sheet keeps its own tab filters when you switch tabs.
- **Format Columns.** The round sliders button in the toolbar, just before the search box, opens a dialog listing the sheet's Date, Date & Time, and Number columns. For a Number column, choose the decimal places (Auto, or 0 to 6) and whether to show a thousands separator. For a Date column, choose a date format (system default, 2026-03-25, 25/03/2026, 03/25/2026, 25 Mar 2026, Mar 25, 2026, or a long date). A Date & Time column also has a time format: system default, 24-hour, or 12-hour. Each column can also have a **default for new entries**: a number for a Number column; Today or a specific date for a Date column; Now or a specific date and time for a Date & Time column. The Add Entry form (and Save & Next's new entry) starts with that value, which you can still change. Each format shows a preview from the column's data. Formats change how values are shown in the table, the totals row, and the tab filter lists; the stored values don't change. Formats are saved with the column, and Excel export uses the decimal places as the cells' number format.
- **Number totals.** A row at the bottom of the table shows the sum of every visible Number column. When search or filters hide some entries, it shows *shown / total*: the sum of the entries in the current list, in amber, then the sum of all entries. Empty cells count as nothing. The row stays in view while the table scrolls. To hide it, click the round Σ button in the toolbar, just before the search box; it is filled while the totals row is shown; each sheet remembers its own choice.
- **Colored pills** for Link and List values; link entries are clickable.
- **Icon-only toolbar.** The toolbar buttons show icons. Hover a button to see its name.
- **Table scroll.** The table scrolls on its own, up and down and left and right. The header row stays at the top. The horizontal scroll bar stays in view.
- **Pinned actions column.** Edit and Delete stay visible at the right edge while the table scrolls horizontally.
- **More-columns dots.** The header of the actions column shows a three-dots button when more columns are hidden to the right. Click it to scroll right. It disappears at the end of the table.
- **Multiple independent instances.** Each browser tab has its own data. The **New Window** button opens a fresh, separate copy.
- **Download App.** The **Download App** link in the header of the [home page](https://platoedes-pty-ltd.github.io/Talika/index.html) downloads the app as `talika.html`, so you can keep it on your disk and run it offline. Your data is not in that file; use Export for your data.
- **Phones, iPad and other touch screens.** Everything works with a finger:
  - Tap a column's ▾ button to open its menu (Filter, Change type, Rename, Remove); tap it again to close it.
  - Long-press a tab for its menu (Rename, Duplicate, Move left, Move right, Delete).
  - Reorder columns with the arrows in the Show/Hide Columns dialog. Drag-and-drop is off on touch-only devices, so a long-press doesn't start a drag; iPads with a trackpad or mouse keep it.
  - Drag the edge of a column header to resize it.
  - Text fields use 16px text on touch screens, so iOS doesn't zoom in when you tap one.
  - Hover tooltips don't exist on touch screens, so the icon-only buttons have no visible names there; screen readers still announce them.
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

The Add/Edit Entry dialog shows each field's type next to its label. **Save & Close** saves the entry and closes the dialog. **Save & Next** saves the entry and opens the next one in the table's current order (search, filters, and sort included); a counter such as "3 of 10" shows where you are. On the last entry, or when adding, **Save & Next** opens a new, empty entry instead; it is added to the table only when you save it. **Close** shuts the dialog without saving.

### Changing a column's type

Open the column menu (the ▾ button in the header) and choose **Change type**. Each value is converted through its text form. The dialog tells you how many values can't convert and will be cleared before you confirm.

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
