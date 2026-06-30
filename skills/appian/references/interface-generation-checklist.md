# Interface Generation Pre-Implementation Checklist

**Purpose:** Quick validation before writing SAIL code to catch common errors early.

**When to use:** After loading all reference files (Steps 1-4), before calling MCP tools (Step 7).

---

## Critical Validation Checklist

### 1. Reference Loading Complete

- [ ] **Step 1 complete:** All 6 universal patterns loaded
  - tools-mcp.md
  - confirmation-patterns.md
  - function-reference.md
  - component-reference.md
  - null-safety-patterns.md
  - short-circuit-patterns.md

- [ ] **Step 2 complete:** Primary domain references loaded
  - interfaces.md (always for interfaces)
  - sail.md (always for interfaces)

- [ ] **Step 3 complete:** UI pattern examples loaded (if applicable)
  - KPI cards? → kpis.md
  - List cards? → card_lists.md
  - Message banners? → messages.md
  - Tab navigation? → tabs.md

- [ ] **Step 4 complete:** Functions and components verified
  - Step 4A: All functions checked against anti-hallucination list
  - Step 4B: All components verified in registry
  - Instruction files loaded for complex components

### 2. Data Source Clarity

**Choose ONE approach per component:**

- [ ] **Mockup/Static Data:**
  - Charts: Use `categories` + `series` (column/bar/line/area) OR `series` only (pie)
  - Grids: Use `data` parameter with local variable
  - Cards: Use local variables for all content

- [ ] **Record Data:**
  - Charts: Use `data: recordType!X` + `config: a!columnChartConfig(...)`
  - Grids: Use `data: recordType!X` + `columns` with `a!gridColumn(...)`
  - Components: Use `a!recordData()` for queries

**⚠️ Do NOT mix approaches** - If using mockup data, never use `config` parameter with charts.

### 3. Chart Pattern Verification

**For mockup charts:**

- [ ] **Column/Line/Bar/Area charts:**
  - Using `categories` parameter (array of labels)
  - Using `series` parameter with `a!chartSeries(label, data, color)`
  - Each series `data` is an array of numbers matching categories length
  - NOT using `data` + `config` approach

- [ ] **Pie charts - Option 1 (Hardcoded):**
  - Using `series` parameter ONLY (no `categories`)
  - Each `a!chartSeries(label, data, color)` = ONE slice
  - Each series `data` is a SINGLE number (not an array)

- [ ] **Pie charts - Option 2 (Dynamic with a!forEach):**
  - Using `series: a!forEach(items: local!data, expression: a!chartSeries(...))`
  - Inside expression: `label: fv!item.labelField`, `data: fv!item.valueField`
  - Using `colorScheme` parameter for automatic colors
  - Each iteration creates ONE slice

**For record-based charts:**

- [ ] Using `data: recordType!X` or `data: a!recordData(...)`
- [ ] Using `config` parameter with appropriate config function
- [ ] Grouping by Text/Date fields (NOT Integer IDs)
- [ ] Measure fields are numeric and use valid aggregation functions

### 4. Layout Hierarchy Validation

- [ ] **Top-level layout chosen:**
  - `a!headerContentLayout()` - for pages with header and content
  - `a!formLayout()` - for forms with submission buttons
  - `a!sectionLayout()` - for simple sectioned content
  - `a!columnsLayout()` - for multi-column pages

- [ ] **Content layouts nested correctly:**
  - Cards inside cardGroupLayout
  - Components inside cardLayout or columnLayout
  - No layout orphans (every component has a parent layout)

- [ ] **Width parameters correct:**
  - `a!sideBySideItem` widths: "AUTO", "MINIMIZE", "1X" to "10X" (NOT "NARROW", "MEDIUM")
  - `a!columnLayout` widths: "AUTO", "NARROW", "MEDIUM", "WIDE", etc.
  - Don't confuse sideBySide widths with column widths

- [ ] **Card width appropriate for card count (a!cardGroupLayout):**
  - **4 KPI cards** → use `"NARROW"` (240px) to fit on one row
  - **3 KPI cards** → use `"NARROW_PLUS"` (320px)
  - **2 KPI cards** → use `"MEDIUM"` (400px) or `"MEDIUM_PLUS"` (560px)
  - **2 charts side-by-side** → use `"WIDE"` (800px) for balanced layout
  - **3 charts** → use `"NARROW_PLUS"` (320px)
  - Rule: Viewport ~1200px / cardWidth = cards per row
  - Common mistake: Using NARROW_PLUS for 4 cards causes wrapping to 2 rows

### 5. Component Parameter Validation

- [ ] **All parameters exist for this component:**
  - Checked against component-reference.md or registry instruction file
  - No fabricated parameters (e.g., `values` doesn't exist for a!chartSeries)
  - No deprecated parameters

- [ ] **Required parameters present:**
  - Charts need either (categories + series) OR (data + config)
  - Forms need `contents` parameter
  - Grids need either `data` parameter OR `data` + `columns`

- [ ] **Color values valid:**
  - Hex codes are 6 characters: "#3B82F6" (not "#3B8" or "3B82F6")
  - Semantic colors: "ACCENT", "POSITIVE", "NEGATIVE", "SECONDARY"
  - NOT using HTML color names: "RED", "BLUE", "GREEN" are invalid

- [ ] **Icon names valid:**
  - From Appian icon library (folder-open, check-circle, exclamation-triangle)
  - No fabricated icons
  - Check icon-aliases.md if unsure

### 6. Null Safety Applied

- [ ] **All data access wrapped in null checks:**
  ```sail
  if(a!isNullOrEmpty(local!data), {}, /* safe operation */)
  ```

- [ ] **Index operations protected:**
  ```sail
  if(a!isNullOrEmpty(local!items), null, index(local!items, 1, null))
  ```

- [ ] **Array operations safe:**
  - Use `{}` for empty array defaults
  - Use `null` for missing single values
  - Don't assume arrays have length > 0

### 7. Function Variable Usage (fv!item, fv!index)

- [ ] **fv!item only used inside a!forEach:**
  - Not used outside forEach context
  - Correctly references current iteration item

- [ ] **fv!index used correctly:**
  - 1-based index (first item = 1, not 0)
  - Only used inside forEach/apply contexts

- [ ] **Local variables use local! prefix:**
  - All variables: `local!varName`
  - No bare variable names

### 8. Record Data Queries (if using record data)

- [ ] **Query structure correct:**
  ```sail
  a!recordData(
    recordType: recordType!CaseName,
    filters: a!queryFilter(...),  /* optional */
    pagingInfo: a!pagingInfo(...)  /* optional */
  )
  ```

- [ ] **Relationship navigation correct:**
  - Use dot notation: `recordType!Case.relationships.assignedUser.fields.firstName`
  - Relationship must exist in record type definition
  - Don't fabricate relationship names

- [ ] **Query filters valid:**
  - Field exists in record type
  - Operator valid for field type (e.g., "=" for Text, "between" for Date)
  - Filter values match field type

- [ ] **Date filtering correct:**
  - Use `a!subtractDateTime()` for past dates (NOT negative days)
  - Use `a!addDateTime()` for future dates only
  - Always use named parameters: `startDateTime:`, `days:`, `hours:`, etc.
  - Example: `a!subtractDateTime(startDateTime: today(), days: 7)` for 7 days ago

- [ ] **Aggregation queries sorted correctly:**
  - When using `a!aggregationFields()` with `a!grouping()`, sort by **alias** (text string)
  - NOT by field reference
  - Example: `sort: a!sortInfo(field: "myAlias", ascending: true)`

### 9. Expression Rule Extraction Plan (if complex logic)

- [ ] **Identify logic to extract:**
  - Calculations longer than 3 lines
  - Reusable transformations
  - Complex validations
  - Data formatting

- [ ] **Plan to create expression rules:**
  - Note rule name, inputs, outputs
  - Create rule after interface if needed
  - Document in comments for now

### 10. Common Anti-Patterns Avoided

- [ ] **NOT using these non-existent functions:**
  - `regexmatch()` - use `like()` or `search()`
  - `property()` - use `index()`
  - `a!dateTimeValue()` - use `datetime()`
  - `a!multipleDropdownField()` - use `a!multipleDropdownFieldByIndex()`

- [ ] **NOT using these non-existent components:**
  - `a!richTextEditor` - use `a!paragraphField` or `a!richTextDisplayField`
  - `a!textAreaField` - use `a!paragraphField`

- [ ] **NOT mixing data approaches:**
  - Don't use `categories` + `config` together
  - Don't use `data` (array) + `values` together (values doesn't exist)

- [ ] **NOT skipping instruction files:**
  - Complex components (grid, chart, card) need instruction files loaded
  - Don't rely on training data alone

---

## Quick Decision Trees

### Chart Data Approach Decision

```
Are you using mockup data?
├─ Yes → Use categories + series (or series only for pie)
│         Each series = a!chartSeries(label, data, color)
│         For pie: each series = ONE slice
│
└─ No (using record data) → Use data + config
                             data: recordType!X or a!recordData()
                             config: a!chartConfig(grouping, measures)
```

### Pie Chart Pattern Decision

```
Do you have data in a!map arrays with label and value fields?
├─ Yes → Use a!forEach pattern
│         series: a!forEach(items: local!data, expression: a!chartSeries(...))
│         Use fv!item.labelField and fv!item.valueField
│         Use colorScheme for automatic colors
│
└─ No (hardcoded slices) → Use static series
                            series: {a!chartSeries(...), a!chartSeries(...)}
                            Specify color for each series
```

### Layout Selection Decision

```
What are you building?
├─ Form with Submit button → a!formLayout(contents: {...}, buttons: a!buttonLayout(...))
│
├─ Page with header + content → a!headerContentLayout(header: ..., contents: {...})
│
├─ Simple sections → a!sectionLayout(contents: {...})
│
└─ Multi-column page → a!columnsLayout(columns: {...})
```

---

## Validation Summary Template

**After completing checklist, confirm:**

```
✅ All reference files loaded (Steps 1-4)
✅ Data source clarity confirmed (mockup vs record)
✅ Chart patterns verified (correct approach chosen)
✅ Layout hierarchy valid (no orphans, correct widths)
✅ All parameters exist (no fabrications)
✅ Null safety applied throughout
✅ Function variables used correctly (fv!item, local!)
✅ Query structure valid (if using record data)
✅ Anti-patterns avoided (checked non-existent list)

Ready to call MCP tools: YES / NO
```

If ANY item is NO, stop and resolve before proceeding.

---

## Integration with Loading Strategy

**This checklist runs at Step 6** in the mandatory loading strategy:

1. Step 1: Load universal patterns
2. Step 2: Load primary domain reference
3. Step 3: Load UI pattern examples
4. Step 4: Verify functions & components (MANDATORY CHECKPOINT)
5. Step 5: Load supplementary references
6. **Step 6: Run this checklist** (Final Pre-Implementation Verification)
7. Step 7: Call MCP tools

---

## Notes

- This is a **lightweight** checklist focused on the most critical validation items
- For complex interfaces with record data, consider loading full orchestration guidance
- Update this checklist as new patterns emerge from testing
- Each item should take < 30 seconds to verify
- Total checklist time: 5-10 minutes for thorough validation
