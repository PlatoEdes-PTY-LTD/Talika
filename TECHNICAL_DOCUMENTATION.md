# Talika Technical Documentation

**Style:** This document follows the ASD-STE100 Simplified Technical English rules. The sentences are short. Each term has one meaning. It is not checked against the official STE dictionary.

**Reader:** A future Claude session that must maintain or extend Talika (formerly DynaDoc).

**Read this first.** Section 1 gives the rules for a session. Section 12 gives the work procedure. The other sections are reference.

---

## 1. Rules for a Claude session

1. Edit only one file: `talika.html`. It holds all the code.
2. Do not edit the first two `<script>` blocks. They contain the Vue and SheetJS libraries.
3. Edit only the third `<script>` block, the HTML template, and the CSS.
4. Keep the app offline. Do not add a network request. Do not add a CDN link.
5. Keep the data model generic. Do not add a hard-coded column name.
6. Do not use `window.confirm()` or `window.alert()`. Use the custom modals.
7. Do not rename the storage keys. They start with `use-case-tracker`. A rename hides the data that users saved.
8. Test each change in a headless browser before you deliver the file (see Section 12).
9. Copy the final file to `/mnt/user-data/outputs/talika.html`. Send it to the user with `SendUserFile`.
10. When you change a feature, also update `README.md` and the design document (see Section 14).

---

## 2. Product summary

Talika (tagline: *The List*) is a dynamic tabular document creator. It was called DynaDoc before step 26. The app file is now `talika.html`. The storage keys and the GitHub Pages URL keep the old name on purpose. It helps a user keep an Excel file up to date.

- The user imports an Excel file or adds columns by hand.
- The user edits the data in a table.
- One document holds several sheets (tabs), like an Excel workbook. A sheet can filter its entries by the data in another sheet (tab filters).
- The user exports the table to an Excel file.

The app has no fixed schema. It runs in the browser from one HTML file. It works offline. It uses no server.

---

## 3. Files

| File | Purpose |
| --- | --- |
| `talika.html` | The complete app. This is the main deliverable. |
| `README.md` | The GitHub readme. |
| `webtalika-icon.svg` | The app icon shown at the top of the readme. A copy of the SVG favicon in `talika.html`; update both together. |
| `LICENSE` | MIT license. |
| `TECHNICAL_DOCUMENTATION.md` | This document. |
| `dynadoc-icon.svg`, `dynadoc-icon.ico`, `dynadoc-icon-512.png` | App icons. The HTML file also holds inline copies. |

**Working paths in a Claude session:**

- Working copy: `/home/claude/talika.html`
- Delivery copy: `/mnt/user-data/outputs/talika.html`
- Scratch scripts: the session scratchpad directory.

If the working copy is missing, copy it from the delivery copy. If the delivery copy is also missing, read `talika.html` from the Project with `project_read`. Large files are saved to a local path. After a change, write the file back with `project_write` and `local_path`.

**Design document:** A Claude Docs document holds the design description. Its ID is `3bae4ae0-91b0-4bde-85a2-f4e0b4dc3f2f`. Its tab ID is `792f0385-61f9`. Its body node ID is `e53e82e2-5ccd`.

---

## 4. Definitions

| Term | Meaning |
| --- | --- |
| Column | One field of the table. It has an `id`, a `name`, a `type`, and a `width`. |
| Row | One record. It has a `_key` and a `values` object. |
| Entry | The user-facing word for a row. |
| Type | The kind of data in a column. There are eight types. |
| Sheet | One tab of the document, like an Excel worksheet. It has its own columns, rows, search, sort, and filters. |
| Tab filter | A filter condition that compares a column of the shown sheet with a column (or one value of a column) of another sheet. |
| Instance | One independent copy of the app data. Each browser tab has one instance. |
| Instance ID | The text that names an instance. It is part of each storage key. |
| Pill | A small colored rounded label. The app uses pills for Link and List values. |
| Actions column | The last table column. It holds the Edit and Delete buttons. |
| More-columns button | The three-dots button in the header cell of the actions column. It shows when columns are hidden to the right. |
| Icon-only button | A button that shows an icon and no text. Its text is in the `title` and `aria-label` attributes. |
| Overlay | The dark layer behind a modal. |

---

## 5. Architecture

### 5.1 File layout

`talika.html` has one `<style>` block, one `<div id="app">` template, and three `<script>` blocks.

| Block | Content | Edit? |
| --- | --- | --- |
| Script 1 | Vue 3.4.31 global build. | No. |
| Script 2 | SheetJS xlsx 0.18.5 full build. | No. |
| Script 3 | App code. It starts with `const { createApp, ref, reactive, computed, nextTick } = Vue;`. It ends with `}).mount('#app');`. | Yes. |

The libraries are inline. This makes the file work offline.

### 5.2 Order of code in script 3

1. Storage keys and instance functions (`resolveInstanceId`, `makeInstanceId`, `migrateLegacyDataIfNeeded`).
2. Helper functions (`slugify`, `nextKey`, `nextColId`, `convertValue`, `valueToText`, `emptyValueFor`, `isUrlLike`, `normalizeUrl`, `parseListEntries`, date helpers, `inferColumn`).
3. The `ComboBox` component.
4. `createApp({ ... setup() { ... } })`. The `setup` function holds the state and the functions. It ends with a `return { ... }` object.

Search for function names. Do not use line numbers. The line numbers change.

### 5.3 Vue rules

- The app uses the Vue 3 Composition API.
- The template is in the HTML. Vue compiles it in the browser.
- Each name that the template uses must be in the `return { ... }` object of `setup`. If you add a function or a ref for the template, add it to that object.
- The `ComboBox` component is a plain object. It uses `<teleport to="body">` for its dropdown.

---

## 6. Data model

### 6.1 State

| Name | Type | Meaning |
| --- | --- | --- |
| `columns` | `reactive([])` | The column list of the shown sheet, in display order. |
| `rows` | `reactive([])` | The row list of the shown sheet. |
| `sheets` | `reactive([])` | The sheets (tabs) in tab order. Each is `{ id, name, columns, rows, view }` from `makeSheet()`. The shown sheet's `columns` and `rows` are empty; its data lives in the top-level `columns` and `rows`. Use `sheetContent(sheet)` to read any sheet's data. |
| `activeSheetId`, `activeSheet` | ref, computed | The shown sheet. |
| `currentFileName` | `ref('')` | The name of the last imported file. |
| `sortColId`, `sortDir` | refs | The sort state. Not saved. |
| `columnFilters` | `reactive({})` | The filter values by column ID. Not saved. |
| `openFilterColId`, `openFilterCol` | ref, computed | The column whose filter popover is open. |
| `tabFilters` | `reactive([])` | The tab filter conditions of the shown sheet: `{ id, colId, op, sheetId, srcColId, srcValue }`. Not saved. See 9.10. |
| `tabFilterTests` | computed | One test function per condition, in the same order as `tabFilters`; `null` for an incomplete condition. |
| `tabFilterSources` | computed | The other sheets that have columns, as `{ sheet, columns }`. Feeds the source dropdown. |
| `activeFilterCount` | computed | The number of columns with an active filter plus the number of complete tab filter conditions. |
| `renamingColId`, `renameDraft` | refs | The rename state. |
| `dragColId`, `dragOverColId` | refs | The drag state for column reorder. |
| `columnVisOpen` | ref | The open flag of the Show / Hide Columns dialog. |
| `visibleColumns`, `hiddenColumnCount` | computed | The columns without `hidden`, and the count of hidden columns. |
| `showHiddenInDialog` | `ref(false)` | The **Show hidden columns** tick in the Add or Edit dialog. Kept while the page is open. Not saved. |
| `dialogColumns` | computed | The fields of the Add or Edit dialog: `columns` when `showHiddenInDialog` is on, else `visibleColumns`. |
| `editingPosition`, `hasNextRecord` | computed | The 1-based place of the edited row in `filteredRows` (0 when adding), and whether a next row exists. |
| `changeTypeCol`, `changeTypeValue`, `changeTypePrefix` | refs | The state of the change-type dialog. |
| `selected` | `reactive(Set)` | The `_key` values of the selected rows. Not saved. |
| `bulkOpen`, `bulk`, `bulkCols`, `bulkIdKeep` | refs | The state of the bulk edit dialog. `bulkCols` holds the ticked column IDs. |
| `exportDialogOpen`, `exportFileName` | refs | The state of the export dialog. |
| `tableWrap` | `ref(null)` | The `.table-wrap` element. |
| `hasMoreRight` | `ref(false)` | `true` when columns are hidden to the right. Not saved. |

### 6.2 Column object

```
{ id: string, name: string, type: string, width: number, prefix?: string, hidden?: true }
```

`prefix` exists only on a column of type `id`.

`hidden` exists only on a hidden column. Showing a column deletes the key. It is saved with the columns.

- `id` is a slug of the name. It must be unique.
- A manual column gets `slug`, then `slug_1`, `slug_2`, and so on.
- An imported column gets `slug_N`, where N is a counter (`nextColId`).
- `width` is in pixels. The template uses `col.width || 200`. The default is 200.

### 6.3 Row object

```
{ _key: number, values: { [columnId]: value } }
```

- `_key` is a runtime ID from `nextKey()`. The app does not save it.
- A row can have no values. This happens when no column exists.
- Always use `emptyValues()` to build a new values object.

### 6.4 Value shapes

| Type | Internal name | Stored value | Empty value |
| --- | --- | --- | --- |
| Text | `text` | string | `''` |
| Date | `date` | `'YYYY-MM-DD'` | `''` |
| Date & Time | `datetime` | `'YYYY-MM-DDTHH:mm'` | `''` |
| Number | `numerical` | number | `null` |
| Set | `select` | string | `''` |
| Link | `hyperlink` | `{ label, url }` | `{ label:'', url:'' }` |
| List | `list` | array of `{ label, url }` | `[]` |
| ID | `id` | `'PREFIX-number'` string | `''` |

The labels in the UI come from the `typeLabels` object: Text, Date, Date & Time, Number, Set, Link, List, ID. The internal name for Set is `select`. The internal name for Number is `numerical`. The internal name for Link is `hyperlink`.

---

## 7. Persistence and instances

### 7.1 Storage keys

| Key | Storage | Content |
| --- | --- | --- |
| `dynadoc-sheets-v1::<instance>` | localStorage | The whole document as JSON: `{ active, sheets: [{ name, columns, rows }] }`. `active` is the index of the shown sheet. Each row has only `values`. |
| `use-case-tracker-columns-v2::<instance>` | localStorage | Pre-sheets columns. Read only when the sheets key is absent, as the first sheet. |
| `use-case-tracker-rows-v2::<instance>` | localStorage | Pre-sheets rows. Read only when the sheets key is absent, as the first sheet. |
| `use-case-tracker-filename-v2::<instance>` | localStorage | The name of the last imported file. |
| `dynadoc_instance_id` | sessionStorage | The instance ID of this tab. |
| `use-case-tracker-migrated-v2` | localStorage | A flag. It shows that old data was moved. |
| `use-case-tracker-columns-v2`, `...-filename-v2`, and the old rows key | localStorage | Legacy keys from before multi-instance support. |

**Warning.** Do not rename these keys. The prefix `use-case-tracker` is a legacy name. A rename makes saved data invisible.

### 7.2 Save and load

- `persist()` saves every sheet and the file name. Call it after any change to columns, rows, or sheets.
- It catches storage errors and ignores them.
- `loadPersisted()` runs once at the end of `setup`, after every ref it touches exists. If the sheets key is absent, it loads the pre-sheets keys as one sheet named `Sheet1`.
- Sort and filter state is not saved. Column order is saved because it is the order of `columns`. Column visibility is saved because `hidden` is on the column object.

### 7.3 Instances

- `resolveInstanceId()` reads the `?instance=` URL parameter first. If it is absent, it reads `sessionStorage`. If that is empty, it makes a new ID.
- The **New Window** button runs `openNewInstance()`. It opens a new tab with a new `?instance=` value.
- `migrateLegacyDataIfNeeded()` copies old data into the first instance one time. It uses the migrated flag.

---

## 8. Excel import and export

### 8.1 Import (`handleImport`, `applyImport`)

1. The app reads every sheet with `parseWorksheet(ws)`. Row 1 is the header row. A sheet with no header row is skipped. A sheet with headers but no rows becomes a sheet with Text columns.
2. It skips data rows that are fully empty. Hyperlinks are read from each row's real position in the sheet, so empty rows do not shift them.
3. A header with no text becomes `Column N`.
4. `inferColumn(name, values, linkTargets)` picks the type. The order is:
   1. **Link**: any cell has an Excel hyperlink.
   2. **List**: any non-empty cell has a semicolon (`;`).
   3. **ID**: every non-empty cell matches `^[A-Za-z][A-Za-z0-9_]*-\d+$` and all cells share one prefix (`detectIdPrefix`). The column gets that `prefix`. A prefix that starts with a digit does not match. This keeps ISO date text such as `2026-10-20` as Text.
   4. **Date or Date & Time**: all non-empty cells are Excel dates. Date & Time applies if any time is not midnight.
   5. **Number**: all non-empty cells are numeric.
   6. **Set**: at least 3 non-empty cells, and the count of distinct values is at most `min(8, ceil(n / 2))`.
   7. **Text**: the fallback.
5. If the document is not blank (`isBlankWorkbook`), the app opens the **Import** choice modal.
   - **Replace All** deletes all sheets and loads the imported ones.
   - **Add as New Sheets** appends each imported sheet as a new tab. Names that clash get a ` (2)` suffix.
   - **Merge** matches each imported sheet to an existing sheet by name, then columns by name (`mergeRowsInto`). Both matches ignore case. The app adds new columns and backfills empty values. An imported sheet with no match becomes a new tab.
6. After the import, `currentFileName` holds the file name.

**List values.** `parseListEntries(raw)` splits the text on `;`. It trims each part and removes empty parts. A part that looks like a URL becomes `{ label: part, url: normalizedUrl }`. Other parts become `{ label: part, url: '' }`.

**Important.** The List delimiter is a semicolon. It was a comma in an earlier version. Text that contains commas stays Text.

### 8.2 Export (`openExportDialog`, `confirmExport`, `exportExcel`)

1. The **Export Excel** button is disabled when no sheet has columns (`hasExportableData`).
2. The button opens a dialog that asks for a file name.
3. The default name is `currentFileName` without `.xlsx` or `.xls`. If it is empty, the default is `use-cases-<YYYY-MM-DD>`.
4. `exportExcel(fileName)` writes one worksheet per sheet, in tab order, named after the tab (`buildWorksheet`).
5. The app replaces the characters `\ / : * ? " < > |` in the name with `-`. It adds `.xlsx`.
6. Each cell is written as follows:
   - Link: the label (or the URL if there is no label). The cell gets a real hyperlink.
   - List: the labels joined with `; `.
   - Other types: the stored value. A null value becomes an empty string.
7. Each column width in the file is 22 characters.

---

## 9. User interface features

### 9.1 Toolbar and icon-only buttons

The toolbar has these items, from left to right: Add Entry, Add Column, Show/Hide Columns, Clear Filters, Search box, Add Tab Filter.

The page header (`.page-head`) has the app name, tagline, and entry count on the left. On the right (`.head-actions`) it has "You are editing <file>" (`.current-file`, shown after an import), then Import Excel, Export Excel, Reset All, and New Window. The header has no description line.

The app has no Save App button. The **Download App** link in the header of `index.html` (between Documentation and Open the app) downloads the app instead. It is a plain `<a href="talika.html" download="talika.html">`, with no script.

- The Search box (`.search-wrap`) takes the free space, so the toolbar spans the table width. Its CSS is `flex: 1 1 320px; min-width: 180px`: it grows to fill the row and shrinks to 180px. The toolbar wraps on a narrow screen.
- The toolbar has no spacer. The Search box fills the gap.
- A round × button (`.search-clear`) shows inside the Search box while it has text. A click, or Esc in the box, clears the search.
- The toolbar buttons show an icon and no text. The text is in the `title` attribute (the tooltip) and in `aria-label`.
- The selection bar buttons (**Edit selected**, **Clear selection**) are also icon-only.
- The empty-state buttons (Add Entry, Add Column, Import Excel) are also icon-only.
- The buttons in the dialogs keep their text (Close, Save & Close, Save & Next, Delete, and so on). The text shows the effect of a choice that the user cannot undo.
- Add Column uses a "columns with a plus" icon. Add Entry uses a plain plus icon.
- When you add a toolbar button, give it a `title` and an `aria-label`. Do not use `title` twice on one element. For a dynamic tooltip, use `:title` only.
- Playwright finds an icon-only button with `getByRole('button', { name: ... })`. The `aria-label` gives the name.
- Show/Hide Columns is a round button (`.btn-round`). When `hiddenColumnCount > 0` it gets the class `has-hidden` (red), the icon changes to a crossed-out eye, and a badge (`.btn-count`) shows the count. The tooltip includes the count. The button is disabled when there are no columns.
- Clear Filters is also a round button (`.btn-round.btn-filters`). When `activeFilterCount > 0` it gets the class `has-filters` (amber) and the `.btn-count` badge. A click runs `clearAllFilters()`, which clears the column filters and the tab filters. It is disabled when no filter is active.
- Add Tab Filter (funnel with a plus) runs `addTabFilter()`. It is disabled when the sheet has no visible column or no other sheet has columns (`tabFilterSources`). The tooltip says which. See 9.10.

### 9.2 Table

The table renders only when `filteredRows.length > 0`. Otherwise the app shows the empty state.

The header and body loop over `visibleColumns`, not `columns`. Code that works on data (search, filters, bulk dialog, import, export) still uses `columns`. The entry dialog draws `dialogColumns`, but `saveEntry` still writes every column in `columns`.

The table sits in `.table-wrap`. The `.table-wrap` is the scroll container. It scrolls up and down and left and right. Its height is limited (see 10.1). The header row is sticky at the top (see 10.4).

**Header cell for a custom column** (`th.col-custom-th`):

| Part | Function |
| --- | --- |
| Sort button | Cycles ascending, descending, no sort (`toggleSort`). |
| Name | Shows the column name. The tooltip shows the type. Double-click starts a rename. |
| Filter button | Shown for every column. Opens the filter popover. Filled amber when the filter is active. |
| Hover icons | Change type (`openChangeType`), Rename (`startRename`), Remove (`removeColumn`). |
| Resize handle | A drag changes the width (`startResize`). |

**Other table columns.** After the custom columns there are two more columns:

1. A 44 px column with the **Add column** (+) button. The body cells in this column are empty.
2. An 80 px actions column (`th.col-actions-th`, `td.col-actions`). The header cell holds the more-columns button (see 9.8).

**Behavior:**

- **Sort** uses `sortValue` and `compareRows`. Empty values go last.
- **Filter** values (`emptyFilterFor`): Number uses `{ min, max }`. Date and Date & Time use `{ from, to }`. Set uses `{ values: [] }`, where `''` means Unset; a row passes if its value is in the list. Text, Link, List, and ID use `{ text }`; a row passes if `cellText` contains the text (case-insensitive). `matchesFilters` applies all filters; a row must pass every one.
- A header with an active filter gets the class `is-filtered` (amber background, amber text, amber bottom line).
- There is one filter popover. It is teleported to `body` from outside the table and reads `openFilterCol`. It must stay outside the table: the table unmounts when no row matches, and a popover inside it would close under the user's click.
- When no row matches and filters are active, the empty state says how many are active and shows a **Clear Filters** button.
- **Reorder** uses native HTML5 drag and drop on the header (`onColDragStart`, `onColDragOver`, `onColDrop`, `onColDragEnd`). The interactive child elements have `draggable="false"`.
- **Pills.** Link and List values use the classes `pill`, `pill-link` (clickable) and `pill-text`.
- **Set values** use the `badge` class.

### 9.3 Dialogs

All dialogs use `.overlay` with `.modal`. A mouse-down on the overlay closes the dialog.

| Dialog | Open flag |
| --- | --- |
| Add or Edit Entry | `dialogOpen` |
| Add Column | `addColumnOpen` |
| Export Excel | `exportDialogOpen` |
| Import choice | `importChoiceOpen` |
| Bulk edit | `bulkOpen` |
| Show / Hide Columns | `columnVisOpen` |
| Change data type | `changeTypeCol` |
| Delete entry | `deleteTarget` |
| Remove column | `columnToRemove` |
| Reset all | `resetAllOpen` |

- The Add or Edit dialog and the Bulk edit dialog draw each field with the `FieldEditor` component. It takes `col`, `values`, `selectOptions`, and `nextId`. It changes `values[col.id]` in place. If you add a column type, add its editor to `FieldEditor`.
- The Add or Edit dialog shows one field for each column in `dialogColumns`. Hidden columns are left out unless the user ticks **Show hidden columns** (shown only when `hiddenColumnCount > 0`). Each label has a badge (`.field-type`) with the type name.
- **Save & Close** (`saveAndClose`) calls `saveEntry`, which returns the saved row (or `null` when nothing was saved), and then closes the dialog.
- **Save & Next** (`editNext`) reads the next row from `filteredRows` first, then calls `saveEntry`, then `openEdit(next)`. Reading first keeps the right next row when the save moves the current row in the sort or filter. When there is no next row (the last row, or adding), it calls `openAdd()` for a blank entry, which is added only when saved. `canSaveAndNext` disables the button only on the Index sheet's last row, since the Index takes no new rows.
- The footer shows `editingPosition` of `filteredRows.length`. **Close** (`closeDialog`) closes without saving.
- The Edit dialog copies values. For Link and List columns, `openEdit` makes deep copies. This prevents changes to the table before the user saves.

### 9.4 ID columns

- The user chooses the type **ID** in the Add Column dialog and enters a prefix. `cleanPrefix` removes spaces and trailing hyphens.
- A value is `PREFIX-number`. `idNumberOf(value, prefix)` returns the number, or `null` if the value does not match.
- `nextIdNumber(col)` returns the highest number in the column plus 1. The first number is 1. Numbers have no zero padding.
- `openAdd` fills each ID field with `nextIdValue(col)`. In the entry dialog, a **Next** button refills the field. The user can type any value. The app does not force unique IDs.
- Sort uses the number part (`sortValue`). The filter is a "contains" text match on the full value, such as `uc-4`.
- Change type: `convertValue` with `opts.prefix` makes IDs. A text that already matches the prefix is kept. A text with only digits becomes `PREFIX-digits`. Other text is cleared. On an existing ID column the same dialog changes the prefix and keeps the numbers.

### 9.5 Multi-select and bulk edit

- The first table column is a sticky checkbox column (`th.col-select-th`, `td.col-select`). The header checkbox selects all shown rows. Shift-click selects a range (`toggleRowSelect`).
- Selected rows get the class `is-selected`. A selection bar above the table shows the count and two icon-only buttons: **Edit selected** (pencil icon) and **Clear selection** (X icon).
- Selection stays when the user searches or filters. The bar shows how many selected rows are hidden. Delete, reset, and an import with Replace remove rows from the selection.
- **Edit selected** opens the Bulk edit dialog. The user ticks the columns to update. Unticked columns do not change.
- `applyBulk` writes the entered value to every selected row for each ticked column. It uses `normalizeValueFor` (the same code as the entry dialog's save) and copies the value for each row, so Link and List values are not shared.
- For a ticked ID column, the dialog has no value field. It gives each row the next free number in table order. The tick **Keep IDs that already exist** (default on) skips rows that already have an ID.

### 9.6 Change data type

- The user opens the column menu (`.col-menu-btn`) and chooses **Change type**.
- `convertValue(from, to, value)` converts each value through its text form (`valueToText`).
- `changeTypeLossCount` counts the values that become empty. The dialog shows this count. When the target type is ID, the dialog also asks for a prefix.
- Rules:
  - To Text or Set: the text form.
  - To Number: the text, without commas, parsed as a number. If it is not a number, the result is `null`.
  - To Date or Date & Time: the text, only if it reads as a date. A text that has only digits is not a date.
  - To Link: from a List, use the first entry. From text, a URL-like text becomes a link.
  - To List: from a Link, make one entry. From text, use `parseListEntries`.
- After the change, the app clears the filter of that column and runs `fixTabFilterOp` on tab filter conditions of that column, so their operator fits the new type. It calls `persistColumns()` and `persist()`.

### 9.7 Show and hide columns

- The Show / Hide Columns dialog lists every column with a checkbox, the name, and the type badge. A hidden column gets the class `is-hidden` (red tint, struck-through name).
- A change applies at once. `setColumnHidden(col, hidden)` sets or deletes `col.hidden` and calls `persistColumns()`.
- When a column is hidden, the app clears its filter, removes the tab filter conditions on it (`removeTabFiltersOn`), closes its filter popover, and clears the sort if it was the sort column. A filter or sort on a hidden column would change the table without a visible cause.
- **Show All** runs `showAllColumns()`. The footer shows "N of M hidden" or "All columns shown".
- Hidden columns stay in search (`cellSearchText`), the bulk edit dialog, and the Excel export. The entry dialog shows them only when **Show hidden columns** is ticked (see 9.3).
- An import with Replace gives new columns, so all are shown. An import with Merge keeps the `hidden` flag of existing columns.

### 9.8 More-columns button

- The button is `button.more-cols-btn`. It sits in the header cell `th.col-actions-th`. It shows three horizontal dots. Its tooltip is "More columns to the right".
- The button shows only when `hasMoreRight` is `true`. This means `scrollLeft + clientWidth < scrollWidth - 2` for `.table-wrap`.
- `updateMoreCols()` sets `hasMoreRight`. It runs on the `scroll` event of `.table-wrap`, on the window `resize` event, and from a `ResizeObserver` on `.table-wrap` and on its `table`. The observer callback uses `requestAnimationFrame`. This prevents the browser error "ResizeObserver loop completed with undelivered notifications". A 300 ms timer also runs `updateMoreCols()`. The timer finds a new `table` element after the empty state changes.
- A click runs `scrollMoreCols()`. It scrolls `.table-wrap` to the right by 80 percent of its visible width (at least 200 px).
- The button does not change the width of the header cell. The button does not depend on any column name.

### 9.6a Column menu

- Each header cell has a `.col-menu-btn` (▾) after the name, in place of the old filter, change-type, rename and remove icons. It is amber (`.active`) when the column has a filter.
- The menu `.col-menu` is teleported to `body` (the table scroller would clip it) and positioned under the button with `position: fixed`, kept on screen, and opened upwards when there is no room below. Items: **Filter…** (opens the filter popover under the button, `openFilterPopoverAt`), **Change type** (shows the current type), **Rename**, and **Remove column**. Locked Index columns show only Filter… and a note.
- **Hover.** `pointerenter` on the button with `pointerType === 'mouse'` opens it (`onColMenuHover`). Leaving the button or the menu starts a 250ms close timer (`onColMenuLeave`); entering the menu clears it (`keepColMenu`).
- **Click or tap** toggles it (`clickColMenu`). A click on a menu opened by hover keeps it open. Touch never opens on hover, so a tap opens and a second tap closes.
- It closes on an outside `mousedown`, on any scroll outside it (the button moves with the table), and after an item is chosen (`colMenuAction`). The filter popover's outside-click check ignores clicks in the menu and on the button.

### 9.7a Touch screens (phones, iPad)

- `touchOnly` is true when `(hover: none) and (pointer: coarse)` matches, which means no mouse or trackpad. Tabs and column headers are then not `draggable`, because on iOS a long-press on a draggable element starts a native drag.
- **Tab menu by long-press.** `onTabTouchStart` starts a 500ms timer. `onTabTouchMove` cancels it when the finger moves more than 10px, since that's a scroll. When the timer fires, `openSheetMenu` opens at the touch point. `onTabTouchEnd` calls `preventDefault()` after a long-press so that the click (which would switch tabs) and the compatibility `mousedown` (which would close the menu) don't fire. `.sheet-tab` has `-webkit-touch-callout: none` so iOS doesn't show its own callout. `openSheetMenu` keeps the menu on screen.
- **Move without dragging.** The tab menu has **Move left** and **Move right** (`canMoveSheet`, `moveSheet`; the Index stays first). The Show/Hide Columns dialog has up and down arrows on each column (`moveColumn`).
- **Resize** uses pointer events (`pointerdown` on `.col-resize-handle`, then `pointermove`, `pointerup`, `pointercancel` on `window`), so a mouse, finger, or pen can resize. The handle has `touch-action: none`.
- **CSS.** `@media (pointer: coarse)` sets 16px text in fields (iOS zooms in on smaller text when a field gets focus) and gives bigger tap targets to the header buttons, the resize handle, `.icon-btn`, the tabs, the tab menu, filter values, and the column list. `@media (max-width: 640px)` makes the page and dialog padding smaller. `body` has `touch-action: manipulation`, which removes the double-tap zoom delay but keeps pinch zoom. `.table-wrap` also uses `100dvh`, so the iOS address bar is taken into account.
- The column filter popover (`toggleFilterPopover`) is moved back inside the screen after it is drawn, and its `maxWidth` is the screen width less 16px, so it doesn't run off the right edge on narrow screens.
- The touch CSS sits at the end of the stylesheet so it overrides the base rules.

### 9.8a Column formats

- The **Format Columns** button is a round toolbar button just before the search box, after Clear Filters. It is disabled when the shown sheet has no Date, Date & Time, or Number column (`formattableColumns`).
- Formats are saved on the column, only when they differ from the default: `dateFormat` (Date and Date & Time: `iso`, `dmy`, `mdy`, `dmony`, `mondy`, `long`; missing means `toLocaleDateString()`), `timeFormat` (Date & Time: `24`, `12`), `decimals` (Number: 0 to 6; missing means auto), and `grouping: false` (Number: no thousands separator).
- `formatColumnValue` uses `formatDateWith`, `formatTimeWith`, and `formatNumberWith`. A Date & Time column with neither format keeps `toLocaleString()`. `formatTotal(col, n)` uses the column's decimals and grouping.
- The dialog edits `formatDraft` (one entry per column) and writes it to the columns only on **Apply** (`applyFormats`); Cancel discards it. `formatPreview(col)` shows the column's first non-empty value, or a sample, in the draft format. Option labels show the sample date 25 March 2026 14:30 (`dateFormatLabel`, `timeFormatLabel`).
- **Defaults for new entries.** The dialog also sets `defaultMode` on the column: `today` (Date), `now` (Date & Time), or `value` with `defaultValue` (a number, `YYYY-MM-DD`, or `YYYY-MM-DDTHH:mm`). A Number column has only a number box; empty means no default. `openAdd` fills each column from `columnDefaultValue(col)`, which works out Today and Now when the form opens. Defaults apply only to the Add Entry form, not to imports, bulk edit, or new columns.
- Changing a column's type (`confirmChangeType`) drops the formats that do not fit the new type, and always drops the default.
- Export: `excelNumberFormat(col)` gives `0.00`, `#,##0.00`, and so on, when `decimals` is set; `buildWorksheet` puts it on the numeric cells as `z`. Date formats are not exported.

### 9.8b Number totals

- The table has a `tfoot` row (`v-if="hasNumberTotals"`: the sheet has a visible Number column, `hasNumberColumns`, and the sheet's `hideTotals` flag is not set). The round toolbar button `.totals-toggle` (a Σ sign; the tooltip says "Show totals row" or "Hide totals row") sits between the Format Columns button and the search box. It is filled teal (`.is-on`, `aria-pressed="true"`) while the totals row is shown, and disabled when the sheet has no visible Number column. A click calls `setTotalsShown`, which sets or clears `hideTotals` on the shown sheet and saves at once. `persist` saves it as `hideTotals: true` on the sheet, and `loadPersisted` restores it that is sticky at the bottom of `.table-wrap`, like the sticky header. Its select and actions cells are sticky left and right.
- `numberTotals` (computed) maps each visible Number column's id to `{ all, shown, partial }`: `all` sums `rows`, `shown` sums `filteredRows` (search, column filters, and tab filters applied), and `partial` is true when `filteredRows` is shorter than `rows`. Values go through `toNumber`, so empty and non-numeric cells are skipped.
- The cell shows `all`, or `shown / all` when `partial` (shown in amber, `.col-total.is-partial`). `formatTotal` rounds to 9 decimal places to drop floating-point noise and formats with `toLocaleString`.

### 9.9 Sheets (tabs)

- The tab bar `.sheet-tabs` sits under `.table-wrap`, like the sheet tabs in Excel. Each tab is `div.sheet-tab` (`.active` on the shown sheet). The **+** button `.sheet-add-btn` adds a sheet named `SheetN`.
- **Switch.** `switchSheet(id)` runs `stashActiveSheet()` and then `activateSheet(sheet)`. Stash moves `columns`, `rows`, and the view state (search, sort, column filters, tab filters) onto the old sheet. Activate moves the new sheet's data into `columns` and `rows`, restores its view state, and clears the row selection. Never call `activateSheet` without a stash first unless the shown data must be discarded (delete, Replace All, Reset All).
- **Rename.** Double-click a tab, or use the context menu. `cleanSheetName()` applies the Excel rules: at most 31 characters, none of `: \ / ? * [ ]`. A name that another sheet has (ignoring case) is refused with a toast.
- **Context menu.** Right-click a tab to open `.sheet-menu` (teleported to `body`, opens upwards): Rename, Duplicate, Delete. Delete is disabled on the only sheet. Delete asks for confirmation only when the sheet has columns or rows.
- **Reorder.** Drag a tab onto another tab (`onSheetDragStart`, `onSheetDrop`).
- **Reset All** leaves the Index and a single blank `Sheet1`.
- **Index sheet.** Every document has one sheet with `isIndex: true`, saved as `index: true`. `loadSheets()` calls `ensureIndexSheet(list)`, which adds it (renaming any other sheet called "Index"), moves it first, and makes sure it has the locked columns `name` (`indexName: true`) and `description`, both Text with `locked: true`. `loadSheets()` also adds `Sheet1` when no other sheet is left. `syncIndex(renamed?)` rebuilds the Index rows, one per other sheet in tab order, matching by name (case-insensitive) so descriptions survive; `renamed` (`{ from, to }`) carries a description across a rename. It runs after add, duplicate (the copy gets the source's description), delete, rename, reorder, and import.
- **Index rules.** Locked columns hide the rename, change-type, and remove buttons (`startRename`, `openChangeType`, and `removeColumn` return early). The Name cell is a `.index-link` button that calls `openSheetByName`. On the Index, Add Entry is disabled and rows have no Delete button. The entry dialog shows the name as read-only text and `saveEntry` keeps the stored name; the bulk dialog lists `bulkColumns`, which leaves the name out. The Index tab is not draggable, not a drop target, and its Rename, Duplicate, and Delete menu items are disabled. Delete on another sheet is disabled when it is the only one left (`contentSheetCount`).
- **Index import and export.** Export writes the Index as a normal worksheet. On import, `readIndexDescriptions()` recognizes a worksheet named "Index" with Name and Description columns; it is not added as a sheet, and `applyIndexDescriptions()` copies its descriptions into the Index (Replace All: all of them; Add as New Sheets: following the new sheet names; Merge: only into empty descriptions). Its other columns are not imported.

### 9.10 Tab filters (filter across sheets)

- The bar `.tab-filter-bar` sits between the toolbar and the selection bar. It shows while `tabFilters` has at least one condition. Each condition is a `.tab-filter-rule` row that reads as a sentence: *Where* [column] [operator] [source column] *value* [Any or one value] [×]. The second and later rows start with *and*. The footer has **+ Add condition**, the hint "Entries must meet every condition." (two or more conditions), and **Clear** (`clearTabFilters`).
- A condition is `{ id, colId, op, sheetId, srcColId, srcValue }`. `colId` is a visible column of the shown sheet. `sheetId` and `srcColId` name a column of another sheet. `srcValue` is `''` for **Any**, or one value from the source column as text.
- **Operators** (`tabFilterOpsFor(type)`, labels in `tabFilterOpLabels`):

| Column type | Operators |
| --- | --- |
| Number | `equal`, `greater`, `less` |
| Date, Date & Time | `equal`, `before`, `after` |
| Text, Set, Link, List, ID | `contains`, `equal` |

- **Source column.** The dropdown `select.tab-filter-source` groups the columns of `tabFilterSources` by sheet. The shown sheet is never a source. Choosing a column (`setTabFilterSource`) resets `srcValue` to `''`.
- **Value.** After a source is chosen, `select.tab-filter-value` lists **Any** and the distinct values of the source column (`tabFilterValueOptions`): sorted, numbers sorted as numbers, numbers and dates labelled with `formatColumnValue`, one option per List entry, the label (or URL) of a Link. A chosen value that no longer exists in the source stays in the list.
- **Matching** (`tabFilterTests`). With **Any**, an entry meets a condition when its value stands in `op` to at least one value of the source column. So `greater` means greater than the smallest source value, and `less` and `before` mean less than the largest. With a chosen value, the entry is compared with that value only.
  - Number: values parsed with `toNumber`; text that is not a number is skipped.
  - Date and Date & Time: `before` and `after` compare `Date.parse` times. `equal` on a Date column compares the day (`YYYY-MM-DD`), so a date equals a date and time on the same day; on a Date & Time column it compares the exact time.
  - Text types: `matchKeys` gives the lower-cased, trimmed strings of a value (Link: label and URL; List: each entry's label and URL). `equal` needs one key in common. `contains` checks whether `cellText` of the entry contains any source key.
- `filteredRows` applies the column filters (`matchesFilters`), then every non-null test, then search and sort. An entry must meet every complete condition.
- A condition without a source column, or whose source sheet or column was deleted, is incomplete: its test is `null`, it is ignored, and its source dropdown has a dashed amber border (`.is-incomplete`).
- Removing a column (`confirmRemoveColumn`) or hiding it removes the conditions on it (`removeTabFiltersOn`).
- Tab filters are part of a sheet's view state: `stashActiveSheet` copies them into `sheet.view.tabFilters`, and `activateSheet` restores them. They are not saved in `localStorage`.

---

## 10. Styling

- CSS variables are in `:root`: `--ink`, `--ink-soft`, `--line`, `--line-soft`, `--paper`, `--panel`, `--teal`, `--teal-dark`, `--teal-tint`, `--amber`, `--amber-tint`, `--danger`, `--danger-tint`, `--radius`, `--shadow`.
- The page container is `#app` with a maximum width of 1800 px.
- The font is the system UI font stack.

### 10.1 Table layout rules (important)

```
.table-wrap { overflow: auto; max-height: calc(100vh - 236px); min-height: 240px; }
table { width: max-content; min-width: 100%; border-collapse: collapse; table-layout: fixed; }
```

- `width: max-content` makes each column keep its set pixel width. Many columns make the table wider than the page. The `.table-wrap` then scrolls horizontally.
- `min-width: 100%` makes a table with few columns fill the page width.
- **Do not use `width: auto`.** With `table-layout: fixed`, it makes the browser shrink all columns to fit the page. The columns lose their set widths.
- `max-height` makes `.table-wrap` scroll up and down on its own. The horizontal scroll bar stays at the bottom of the visible frame. The value leaves room for the sheet tabs under the table. If you add content above or below the table, change the `236px` value.

### 10.2 Sticky actions column

```
.col-actions-th { position: sticky; right: 0; z-index: 3; background: #eef2f1; }
td.col-actions  { position: sticky; right: 0; z-index: 1; background: var(--panel); }
tbody tr:hover td.col-actions { background: #fafcfb; }
```

- The backgrounds must be opaque. A transparent background shows the scrolled cells under the column.
- If you change the row hover color, change the hover color of `td.col-actions` also.

### 10.3 More-columns button

```
.more-cols-btn { display: inline-flex; width: 34px; height: 24px; }
.col-actions-th { text-align: center; }
```

- The button is inside `th.col-actions-th`. It uses the colors of the other toolbar buttons. The hover color is `--teal-tint`.

### 10.4 Sticky header row

```
thead th { position: sticky; top: 0; z-index: 2; }
thead th.col-select-th { z-index: 3; }   /* also sticky left */
thead th.col-actions-th { z-index: 4; }  /* also sticky right */
.col-custom-th, thead th.col-custom-th { position: sticky; top: 0; }
```

- Each header cell is sticky at the top of `.table-wrap`. The header cells need an opaque background.
- `.col-custom-th` must not set `position: relative`. This rule would remove the sticky position. The resize handle still works because a sticky element is a positioned parent.
- The body cells with `z-index: 1` stay under the header cells.

---

## 11. Known constraints and causes

| Constraint | Cause and action |
| --- | --- |
| Do not use native `confirm()`. | It is not reliable in embedded preview contexts. Use a custom modal. |
| Dropdowns and popovers use `<teleport to="body">`. | The table wrapper clips content that overflows. The filter popover and the ComboBox dropdown must leave the wrapper. |
| The filter popover sits outside `<table>`. | The table unmounts when no row matches. A popover inside it disappears when a filter removes the last row. |
| Interactive header elements need `draggable="false"`. | The `<th>` is draggable. Without this attribute, a click or a text selection starts a drag. |
| The resize handle selector is `.col-custom-th > span.col-resize-handle`. | A less specific selector loses to a rule that sets `cursor` on the `span`. |
| List pills must keep the original entry order. | Use one `v-for` with `v-if` and `v-else`. Two separate loops group links before text. |
| An empty Link value is an object. | `formatColumnValue` returns `—` for any object value. Without this, the cell shows raw JSON. |
| The table does not render when no row matches. | An empty table with columns shows the empty state. This is correct. |
| Storage is limited to about 5 MB in `localStorage`. | Very large tables can fail to save. `persist()` ignores the error. |
| Sort and filter are not saved. | This is the current design. |
| A `ResizeObserver` callback must not change layout in the same frame. | The browser logs "ResizeObserver loop completed with undelivered notifications". Use `requestAnimationFrame` in the callback. |
| A scroll event closes the filter popover. | A global capture listener on `window` does this. It also fires when `.table-wrap` scrolls. This is correct. |
| An element must not have both `title` and `:title`. | Vue shows a duplicate attribute warning. Use one of them. |

---

## 12. Work procedure for a change

1. **Prepare.** Copy `/mnt/user-data/outputs/talika.html` to `/home/claude/talika.html`. If it is missing, read it from the Project (see Section 3).
2. **Find the code.** Search for the function or CSS class name in the working copy.
3. **Edit.** Use the Edit tool for small changes. If you add something that the template uses, add it to the `return { ... }` object.
4. **Check the structure.** Count the opening and closing tags of `div`, `table`, `tr`, `th`, `td`. They must match.
5. **Test.** Write a Playwright script in the scratchpad. See Section 13.
6. **Look at the result.** Take a screenshot. Use the Read tool to look at it. Test with many rows. A short table can hide layout errors.
7. **Deliver.** Copy the file to `/mnt/user-data/outputs/talika.html`. Send it with `SendUserFile`. Write one short caption.
8. **Update the documents.** See Section 14.

---

## 13. Test method

No automated test suite exists. Each session writes short Playwright scripts. Chromium is installed. Use `require('playwright')` in a Node script. Open the file with `file:///home/claude/talika.html`. If `require` fails, set `NODE_PATH` to the output of `npm root -g`.

**Selectors that work:**

| Target | Selector or call |
| --- | --- |
| Toolbar button | `page.getByRole('button', { name: 'Add Column' })` (the name comes from `aria-label`) |
| Add Column name field | `input[placeholder="e.g. Owner"]` |
| Confirm Add Column | `getByRole('button', { name: /^Add Column$/ }).last()` |
| Add Entry text fields | `.overlay textarea` (for Text columns) |
| Save entry | `getByRole('button', { name: /save entry/i })` |
| Save and next | `getByRole('button', { name: /save & next/i })` |
| Export confirm | `getByRole('button', { name: /^Export$/ })` |
| Import a file | `page.setInputFiles('input[type=file]', path)` |
| Header cells | `thead th.col-custom-th` |
| Column menu button | `.col-menu-btn` (hover or click it); menu `.col-menu`, items by text: Filter…, Change type, Rename, Remove column |
| Actions cell | `td.col-actions` |
| Table scroller | `.table-wrap` |
| More-columns button | `.more-cols-btn` |
| Column filter | `.col-menu-btn`, then `.col-menu` button **Filter…** |
| Filter popover | `.filter-popover` (Set values: `.filter-value`) |
| Clear Filters button | `getByRole('button', { name: 'Clear Filters' }).first()`, or `.btn-filters` |
| Row checkbox | `td.col-select input` |
| Show/Hide Columns button | `getByRole('button', { name: 'Show/Hide Columns' })`, or `.btn-round` |
| Hidden-count badge | `.btn-round .btn-count` |
| Sheet tab | `.sheet-tab` (by name: `.sheet-tab >> text=Sheet1`), shown tab `.sheet-tab.active` |
| New sheet | `.sheet-add-btn` |
| Sheet context menu | right-click `.sheet-tab`, then `.sheet-menu` |
| Add Tab Filter button | `getByRole('button', { name: 'Add Tab Filter' })` |
| Tab filter bar and conditions | `.tab-filter-bar`, `.tab-filter-rule` (selects in order: column, operator, `.tab-filter-source`, `.tab-filter-value`) |
| Add condition, Clear | `.tab-filter-foot .btn-link` (first: Add condition, last: Clear) |

**Hints:**

- Add at least one entry before you test the table. Without a row the table does not render.
- Do not fill the search box by mistake. Use `.overlay` to limit a selector to a dialog.
- A native drag does not work with mouse events in Playwright. Use `dispatchEvent` with a `DataTransfer` object for `dragstart`, `dragover`, `drop` and `dragend`.
- To test a download, use `page.waitForEvent('download')` and read `download.suggestedFilename()`.
- To make a test `.xlsx` file, use Python and `openpyxl`. The `xlsx` Node module is not installed. If the user attaches an `.xlsx` file, test with that file.
- To test overflow, set a small viewport, such as 1000 by 700. Add 8 or more columns. Compare `scrollWidth` with `clientWidth` of `.table-wrap`.
- To test the more-columns button, check that `.more-cols-btn` exists at scroll position 0. Scroll `.table-wrap` to the end. Check that the button is gone. Wait 500 ms after a scroll or a resize.
- To test the vertical scroll, import 20 or more rows. Set `.table-wrap` `scrollTop`. Check with a screenshot that the header row stays at the top.
- Also check that the page logs no console errors or warnings (`page.on('pageerror', ...)`, `page.on('console', ...)`). Headless Chromium can miss the ResizeObserver error. Keep the `requestAnimationFrame` guard.

---

## 14. Documents to keep current

| Document | Update when |
| --- | --- |
| `README.md` | A user-visible feature changes. |
| Design document (Claude Docs) | A feature, a data shape, or a rule in this file changes. Use the Claude Docs tools. Read the document first. Change only the text that is wrong. |
| `TECHNICAL_DOCUMENTATION.md` | Any section here becomes wrong. |

---

## 15. Change history

| Step | Change |
| --- | --- |
| 1 | Generic column and row model. Seven types. Excel import and export. |
| 2 | The column header shows the name only. The type is in the tooltip. |
| 3 | Resizable columns. Widths are saved. |
| 4 | List type. Import detects it. |
| 5 | The app name is DynaDoc. |
| 6 | Independent instances for each tab. New Window button. |
| 7 | The Status type is renamed Set. |
| 8 | Column rename. |
| 9 | Sort for all columns. Range filter for Number, Date, Date & Time. |
| 10 | Column reorder by drag and drop. |
| 11 | Colored pills for Link and List. |
| 12 | Sticky actions column. |
| 13 | Table width fix: `width: max-content` with `min-width: 100%`. |
| 14 | Export asks for a file name. The default is the imported file name. |
| 15 | The List delimiter changes from comma to semicolon. |
| 16 | Change data type for a column. Type badge in the entry dialog. |
| 17 | README, LICENSE (MIT), file rename to `dynadoc.html`. |
| 18 | ID column type (prefix and auto number). Row multi-select and bulk edit. `FieldEditor` component shared by the entry and bulk dialogs. |
| 19 | More-columns button (three dots). It shows when columns are hidden to the right. A click scrolls right. `requestAnimationFrame` guard for the `ResizeObserver`. |
| 20 | The table scrolls on its own (`max-height` on `.table-wrap`). The header row is sticky. The more-columns button moves into the header cell of the actions column. |
| 21 | Icon-only buttons in the toolbar, the selection bar, and the empty state. The text is in the tooltip. New icon for Add Column. |
| 22 | New toolbar order: Add Entry, Add Column, Search, Import, Export, Reset, New Window. The Search box is at least 30 percent wide. |
| 23 | Show and hide columns. Round toolbar button with an eye icon. It turns red and shows a count when columns are hidden. `hidden` flag saved on the column. |
| 24 | Filters for every column type: Set value checklist, "contains" text for Text, Link, List, ID. Filtered headers turn amber. Clear Filters toolbar button with an active-filter count. The filter popover moves outside the table. |
| 25 | Sheets (tabs) like Excel: add, rename, duplicate, delete, reorder. Each sheet keeps its own search, sort, and filters. Import reads every sheet; export writes every sheet. Import adds **Add as New Sheets**; Merge matches sheets by name. New storage key `dynadoc-sheets-v1::<instance>`; the old keys load as the first sheet. |
| 26 | The app is renamed Talika, with the tagline "The List". The icon, the file name `dynadoc.html`, the storage keys, and the site URL do not change. |
| 27 | The description under the header is removed. The imported file name, Import, Export, Reset All, and New Window move to the right side of the header. |
| 28 | Clear (×) button inside the Search box. Esc in the box also clears it. |
| 29 | Save App button at the right corner of the header. It downloads the app as `talika.html`. Hidden when the app runs from the local disk. |
| 30 | Tab filters: Add Tab Filter toolbar button after the Search box and a condition bar above the table. A condition compares a column of the shown sheet (contains, equals, greater, less, before, after, by type) with a column of another sheet, or with one value picked from it. All conditions must match. Kept per sheet with the view state; not saved. |
| 31 | Save App is removed from the app. A Download App link in the `index.html` header, between Documentation and Open the app, downloads `talika.html` instead. |
| 32 | Entry dialog: Save Entry keeps the dialog open. Save & Next saves and opens the next row in `filteredRows`, with an "N of M" counter. Cancel is renamed Close. Hidden columns are left out of the dialog unless **Show hidden columns** is ticked. |
| 33 | Index tab: every document has an Index sheet, pinned first, with locked Name and Description (Text) columns and one row per other sheet, kept in sync. Names link to their sheet. Saved with `index: true`; an imported Index fills in descriptions instead of becoming a sheet. |
| 34 | Entry dialog: Save Entry becomes **Save & Close** (saves and closes). **Save & Next** on the last row or when adding opens a blank new entry; it is disabled only on the Index sheet's last row. |
| 35 | Number totals: a sticky footer row shows the sum of each visible Number column; with search or filters active it shows the shown sum / the total of all entries. |
| 36 | Format Columns: a button at the top right opens a dialog to set date formats (Date, Date & Time), time formats (Date & Time), and decimal places and thousands separator (Number), per column. Saved on the column; display only; Excel export uses the decimal places. |
| 37 | The Format Columns button moves from the page header to the header of the actions column. |
| 38 | **Show totals row** checkbox in the Format Columns dialog hides or shows the totals row, per sheet (saved as `hideTotals`). |
| 39 | The Show totals row checkbox moves from the Format Columns dialog to the actions column header, with no label (the text is in the tooltip). It applies at once. |
| 40 | The Format Columns button and the Show totals row checkbox move from the actions column header to the toolbar, before the search box; both are disabled when they do not apply. The search box is narrower (up to 320px, at least 180px). The actions column is back to 80px. |
| 41 | The search box grows again to take the free space in the toolbar (`flex: 1 1 320px; min-width: 180px`), so the toolbar spans the table width. |
| 42 | The totals toggle is a round Σ button instead of a checkbox; it is filled teal while the totals row is shown. |
| 43 | Defaults for new entries in the Format Columns dialog: a number (Number), Today or a date (Date), Now or a date and time (Date & Time). Saved on the column as `defaultMode` and `defaultValue`; used by `openAdd`. |
| 44 | Touch screens: long-press a tab for its menu; Move left and Move right in the tab menu; column order arrows in Show/Hide Columns; resizing with pointer events; header buttons always shown without hover; 16px text in fields; bigger tap targets; smaller padding on phones; no native drag and drop on touch-only devices. |
| 45 | Column menu: the filter, change-type, rename and remove icons in each header are replaced by one ▾ button that opens a menu on hover (mouse) or tap (touch). |

---

## 16. Open items

- **Browser extension.** The user plans to convert Talika to a Manifest V3 extension. A todo exists in the Floty app, starting 19 October 2026. Extension pages block inline scripts and `eval`. The work needs these steps:
  1. Move the three scripts into separate `.js` files.
  2. Precompile the Vue templates with a Vite build.
  3. Add `manifest.json` and a background script. The script opens `talika.html` in a tab when the user clicks the toolbar icon.
  4. Think about `chrome.storage.local` with the `unlimitedStorage` permission for large tables.
- **Export default name.** It is still `use-cases-<date>`. The user can ask to change it to `dynadoc-<date>`.
- **Storage key prefix.** It is still `use-case-tracker`. Do not change it unless you also write a migration.
- **Design document.** It does not yet describe the table scroll, the more-columns button, the icon-only buttons, or the toolbar order. The as-of date in the byline is 9 October 2026.
