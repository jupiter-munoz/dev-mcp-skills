# SAIL Verification Checkpoint

🛑 **MANDATORY: You cannot proceed to code generation without completing this checkpoint.**

This checkpoint prevents hallucinating non-existent functions or SAIL components. Complete the relevant section(s) based on what you're building:

- **Expression rules:** Complete **Step 3A only** (verify functions)
- **Interfaces:** Complete **BOTH Step 3A and Step 3B** (verify functions AND components)

**Why both for interfaces?** Every interface uses:
- **Components** for UX (a!cardLayout, a!textField, a!gridField)
- **Functions** for logic (a!localVariables, a!recordData, if(), a!queryFilter, etc.)

---

## Step 3A: Verify Functions

**Required for:** Expression rules AND interfaces

### 1. List All Functions You Plan to Use

Write them down before proceeding. Example:
```
Functions I will use:
- a!localVariables
- a!recordData
- a!queryFilter
- a!map
- if
- a!isNullOrEmpty
- tostring
- index
```

### 2. Check Anti-Hallucination List

**Action:** Load `function-reference.md` and check the "Functions That DO NOT Exist" section.

**Common non-existent functions to avoid:**
- ❌ `regexmatch()` → Use `like()` or `search()` instead
- ❌ `property()` → Use `index()` or dot notation instead
- ❌ `a!recordQuery()` → Use `a!recordData()` instead
- ❌ `a!dateTimeValue()` → Use `datetime()` instead

**If any function you plan to use is on the non-existent list:**
- ✅ Remove it from your code plan
- ✅ Use the recommended alternative
- ✅ Update your function list above

### 3. Verify Functions via Tier 2A (functions.json)

**When to do this:** For any function NOT fully documented in `function-reference.md` with signature and parameters.

**Action:** Verify each unknown function exists in your Appian version:

```bash
# Read VERSION from SKILL.md Configuration section
VERSION="26.6"  # Use the configured version

# Template (function names must be lowercase):
curl -sL "https://docs.appian.com/suite/help/$VERSION/functions.json" | jq -r '.["functionname"]'

# Example 1 - Verify a!map exists:
curl -sL "https://docs.appian.com/suite/help/$VERSION/functions.json" | jq -r '.["a!map"]'

# Example 2 - Verify multiple functions at once:
for func in "a!map" "tostring" "a!localvariables"; do
  result=$(curl -sL "https://docs.appian.com/suite/help/$VERSION/functions.json" | jq -r ".[\"$func\"]")
  echo "$func: $result"
done
```

**Expected output (function exists):**
```
/suite/help/$VERSION/fnc_system_map.html
```

**Expected output (function does NOT exist):**
```
null
```

**If lookup returns null:**
- ❌ Function does not exist in your Appian version
- ✅ Remove from your code plan
- ✅ Search for alternative in `function-reference.md` or via Tier 3 (doc search)

**If Tier 2A fails (network timeout, DNS failure):**
- ⚠️ Proceed with Tier 1 (skill references) only
- ⚠️ Inform user: "Could not verify function existence. Proceeding with skill references only."

### 4. Step 3A Exit Checklist

**Before proceeding to Step 3B (interfaces) or Step 4 (expression rules):**

- [ ] All functions listed
- [ ] Anti-hallucination list in `function-reference.md` checked
- [ ] Unknown functions verified via Tier 2A (or Tier 2A skipped due to network failure)
- [ ] No non-existent functions in my code plan
- [ ] All functions confirmed to exist OR have documented alternatives

✅ **Step 3A complete. Expression rules: proceed to Step 4. Interfaces: continue to Step 3B.**

---

## Step 3B: Verify SAIL Components

**Required for:** Interfaces only (skip this for expression rules)

### 1. List All SAIL Components You Plan to Use

Write them down before proceeding. Example:
```
Components I will use:
- a!localVariables (actually a function - verify in Step 3A)
- a!headerContentLayout
- a!cardLayout
- a!columnsLayout
- a!columnLayout
- a!textField
- a!dropdownField
- a!buttonWidget
- a!gridField
```

**Note:** Some Appian constructs are functions, not components (e.g., `a!localVariables`, `a!save`, `a!map`). If unsure, check both Step 3A and Step 3B.

### 2. Check Component Existence via Registry

**Action:** For each component, verify it exists in the registry.

```bash
# Check if component exists:
grep '"COMPONENT_NAME"' /Users/jupiter.munoz/.claude/skills/appian/registry/components-registry.json

# Example - Check a!cardLayout:
grep '"a!cardLayout"' /Users/jupiter.munoz/.claude/skills/appian/registry/components-registry.json

# Example - Check multiple components:
grep -E '"a!(cardLayout|textField|gridField)"' /Users/jupiter.munoz/.claude/skills/appian/registry/components-registry.json
```

**Expected output (component exists):**
```json
  "a!cardLayout": {
    "exists": true,
    "hasInstructionFile": true,
    "instructionFile": "layouts/card-layout-instructions.md",
    ...
  }
```

**Expected output (component marked as non-existent):**
```json
  "a!richTextEditor": {
    "exists": false,
    "reason": "No rich text editor exists",
    "alternatives": ["a!paragraphField", "a!styledTextEditorField", "a!richTextDisplayField"]
  }
```

**If `exists: false`:**
- ❌ Component does not exist in Appian
- ✅ Remove from your code plan
- ✅ Use alternatives listed in registry

**If component not found in registry at all:**
- ⚠️ Component may not exist (registry has 145 components, should be comprehensive)
- ✅ Proceed to section 4 below (Tier 2B verification) to confirm

### 3. Load Instruction Files for Components

**Action:** For each component with `hasInstructionFile: true`, load the instruction file.

```bash
# Check which components have instruction files:
jq '.["COMPONENT_NAME"].instructionFile' /Users/jupiter.munoz/.claude/skills/appian/registry/components-registry.json

# Example:
jq '.["a!cardLayout"].instructionFile' /Users/jupiter.munoz/.claude/skills/appian/registry/components-registry.json
# Output: "layouts/card-layout-instructions.md"

jq '.["a!gridField"].instructionFile' /Users/jupiter.munoz/.claude/skills/appian/registry/components-registry.json
# Output: "components/grid-field-instructions.md"
```

**Then load each instruction file:**
```
Load: /Users/jupiter.munoz/.claude/skills/appian/components/grid-field-instructions.md
```

**Why load instruction files?**
- Contains critical warnings (e.g., `a!gridField.showSearchBox` only works with recordType! data)
- Documents parameter constraints not visible in tool schemas
- Provides usage patterns and common mistakes

**Common components with critical instruction files:**
- `a!gridField` → **Critical:** showSearchBox/showRefreshButton/recordActions only work with recordType! data
- `a!cardLayout` → Layout and styling constraints
- `a!formLayout` → Cannot be nested, button footer requirements
- `a!wizardLayout` → Step navigation patterns
- Chart components → Data structure requirements

### 4. Verify Components via Tier 2B (functions.json)

**When to do this:** For components WITHOUT instruction files (`hasInstructionFile: false` or missing from registry).

**Action:** Verify component exists and fetch signature from functions.json:

```bash
# Read VERSION from SKILL.md Configuration section
VERSION="26.6"  # Use the configured version

# Components are in functions.json! Use lowercase for lookup:
curl -sL "https://docs.appian.com/suite/help/$VERSION/functions.json" | jq -r '.["componentname_lowercase"]'

# Example - Verify a!textfield (convert to lowercase):
curl -sL "https://docs.appian.com/suite/help/$VERSION/functions.json" | jq -r '.["a!textfield"]'

# Note: functions.json uses lowercase, but you should use camelCase in code (a!textField)
```

**Expected output (component exists):**
```
/suite/help/$VERSION/Text_Component.html
```

**Expected output (component does NOT exist):**
```
null
```

**If Tier 2B fails (network timeout, DNS failure):**
- ⚠️ Use `component-reference.md` for parameter reference
- ⚠️ Inform user: "Component exists (per registry), but couldn't fetch official signature. Using parameter reference."

### 5. Document Critical Warnings

**Action:** For each component with loaded instruction files, note critical warnings.

Example documentation:
```
Critical warnings for this interface:
- a!gridField: DO NOT use showSearchBox with local data (only recordType! data)
- a!formLayout: Cannot nest inside another a!formLayout
- a!cardLayout: style parameter affects border and background
```

### 6. Step 3B Exit Checklist

**Before proceeding to Step 4:**

- [ ] All SAIL components listed
- [ ] Registry existence checked for each component
- [ ] Components with `exists: false` removed from code plan
- [ ] Instruction files loaded for components that have them
- [ ] Critical warnings documented
- [ ] Components without instruction files verified via Tier 2B (or skipped if network failure)
- [ ] No non-existent components in my code plan

✅ **Step 3B complete. Return to SKILL.md to continue with Step 4 (supplementary references) and Step 5 (pre-implementation verification).**

---

## Final Verification Summary

Before generating code, complete this summary:

### Expression Rules
- [ ] Step 3A completed (all functions verified)
- [ ] No non-existent functions in code plan
- [ ] Ready to proceed to Step 4

### Interfaces
- [ ] Step 3A completed (all functions verified)
- [ ] Step 3B completed (all components verified)
- [ ] Instruction files loaded and critical warnings noted
- [ ] No non-existent functions or components in code plan
- [ ] Ready to proceed to Step 4

---

## Quick Reference

**Related documentation:**
- Anti-hallucination lists: `function-reference.md`, `component-reference.md`
- Function catalog: `function-reference.md` (curated list of ~50 common functions)
- Component catalog: `component-reference.md` (~53 common components)
- Component registry: `registry/components-registry.json` (all 145 components)
- Full lookup workflow: `documentation-lookup-strategy.md`
- Component instruction file index: `component-loading-index.md`

**Verification tier summary:**
- **Tier 1:** Skill references (anti-hallucination lists, curated catalogs)
- **Tier 2A:** functions.json for function existence (495 functions, definitive)
- **Tier 2B:** Registry + functions.json for component existence (145 components)
- **Tier 3:** Documentation search for patterns/recipes (optional, when Tier 1/2 insufficient)

**Path to files:**
- Skill root: `/Users/jupiter.munoz/.claude/skills/appian/`
- References: `/Users/jupiter.munoz/.claude/skills/appian/references/`
- Registry: `/Users/jupiter.munoz/.claude/skills/appian/registry/components-registry.json`
- Component instructions: `/Users/jupiter.munoz/.claude/skills/appian/components/` and `layouts/`

---

🎯 **Checkpoint complete. You are now ready to generate code with verified functions and components.**
