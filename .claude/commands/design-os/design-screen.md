# Design Screen

You are helping the user create a screen design for a section of their product. The screen design will be a props-based React component that can be exported and integrated into any React codebase.

## Step 1: Check Prerequisites

First, identify the target section and verify that `spec.md`, `data.json`, and `types.ts` all exist.

Read `/product/product-roadmap.md` to get the list of available sections.

If there's only one section, auto-select it. If there are multiple sections, use the AskUserQuestion tool to ask which section the user wants to create a screen design for.

Then verify all required files exist:

- `product/sections/[section-id]/spec.md`
- `product/sections/[section-id]/data.json`
- `product/sections/[section-id]/types.ts`

If spec.md doesn't exist:

"I don't see a specification for **[Section Title]** yet. Please run `/shape-section` first to define the section's requirements."

If data.json or types.ts don't exist:

"I don't see sample data for **[Section Title]** yet. Please run `/sample-data` first to create sample data and types for the screen designs."

Stop here if any file is missing.

## Step 2: Check for Design System and Shell

Check for optional enhancements:

**Design Tokens:**
- Check if `/product/design-system/colors.json` exists
- Check if `/product/design-system/typography.json` exists

If design tokens exist, read them and use them for styling. If they don't exist, show a warning:

"Note: Design tokens haven't been defined yet. I'll use default styling, but for consistent branding, consider running `/design-tokens` first."

**Shell:**
- Check if `src/shell/components/AppShell.tsx` exists

If shell exists, the screen design will render inside the shell in Design OS. If not, show a warning:

"Note: An application shell hasn't been designed yet. The screen design will render standalone. Consider running `/design-shell` first to see section screen designs in the full app context."

## Step 3: Analyze Requirements

Read and analyze all three files:

1. **spec.md** - Understand the user flows and UI requirements
2. **data.json** - Understand the data structure and sample content
3. **types.ts** - Understand the TypeScript interfaces and available callbacks

Identify what views are needed based on the spec. Common patterns:

- List/dashboard view (showing multiple items)
- Detail view (showing a single item)
- Form/create view (for adding/editing)

## Step 4: Clarify the Screen Design Scope

If the spec implies multiple views, use the AskUserQuestion tool to confirm which view to build first:

"The specification suggests a few different views for **[Section Title]**:

1. **[View 1]** - [Brief description]
2. **[View 2]** - [Brief description]

Which view should I create first?"

If there's only one obvious view, proceed directly.

## Step 5: Invoke the Frontend Design Skill

Before creating the screen design, read the `frontend-design` skill to ensure high-quality design output.

Read the file at `.claude/skills/frontend-design/SKILL.md` and follow its guidance for creating distinctive, production-grade interfaces.

## Step 6: Create the Props-Based Component

Create the main component file at `src/sections/[section-id]/components/[ViewName].tsx`.

### Component Structure

The component MUST:

- Import types from the types.ts file
- Accept all data via props (never import data.json directly)
- Accept callback props for all actions
- Be fully self-contained and portable

Example:

```tsx
import type { InvoiceListProps } from '@/../product/sections/[section-id]/types'

export function InvoiceList({
  invoices,
  onView,
  onEdit,
  onDelete,
  onCreate
}: InvoiceListProps) {
  return (
    <div className="max-w-4xl mx-auto">
      {/* Component content here */}

      {/* Example: Using a callback */}
      <button onClick={onCreate}>Create Invoice</button>

      {/* Example: Mapping data with callbacks */}
      {invoices.map(invoice => (
        <div key={invoice.id}>
          <span>{invoice.clientName}</span>
          <button onClick={() => onView?.(invoice.id)}>View</button>
          <button onClick={() => onEdit?.(invoice.id)}>Edit</button>
          <button onClick={() => onDelete?.(invoice.id)}>Delete</button>
        </div>
      ))}
    </div>
  )
}
```

### Design Requirements

- **Mobile responsive:** Use Tailwind responsive prefixes (`sm:`, `md:`, `lg:`) and ensure the design layout works gracefully on mobile, tablet and desktop screen sizes.
- **Light & dark mode:** Use `dark:` variants for all colors
- **Use design tokens:** If defined, apply the product's color palette and typography
- **Follow the frontend-design skill:** Create distinctive, memorable interfaces

### Applying Design Tokens

**If `/product/design-system/colors.json` exists:**
- Use the primary color for buttons, links, and key accents
- Use the secondary color for tags, highlights, secondary elements
- Use the neutral color for backgrounds, text, and borders
- Example: If primary is `lime`, use `lime-500`, `lime-600`, etc. for primary actions

**If `/product/design-system/typography.json` exists:**
- Note the font choices for reference in comments
- The fonts will be applied at the app level, but use appropriate font weights

**If design tokens don't exist:**
- Fall back to `stone` for neutrals and `lime` for accents (Design OS defaults)

### What to Include

- Implement ALL user flows and UI requirements from the spec
- Use the prop data (not hardcoded values)
- Include realistic UI states (hover, active, etc.)
- Use the callback props for all interactive elements
- Handle optional callbacks with optional chaining: `onClick={() => onDelete?.(id)}`

### What NOT to Include

- No `import data from` statements - data comes via props
- No features not specified in the spec
- No routing logic - callbacks handle navigation intent
- No navigation elements (shell handles navigation)

## Step 7: Create Sub-Components (If Needed)

For complex views, break down into sub-components. Each sub-component should also be props-based.

Create sub-components at `src/sections/[section-id]/components/[SubComponent].tsx`.

Example:

```tsx
import type { Invoice } from '@/../product/sections/[section-id]/types'

interface InvoiceRowProps {
  invoice: Invoice
  onView?: () => void
  onEdit?: () => void
  onDelete?: () => void
}

export function InvoiceRow({ invoice, onView, onEdit, onDelete }: InvoiceRowProps) {
  return (
    <div className="flex items-center justify-between p-4 border-b">
      <div>
        <p className="font-medium">{invoice.clientName}</p>
        <p className="text-sm text-stone-500">{invoice.invoiceNumber}</p>
      </div>
      <div className="flex gap-2">
        <button onClick={onView}>View</button>
        <button onClick={onEdit}>Edit</button>
        <button onClick={onDelete}>Delete</button>
      </div>
    </div>
  )
}
```

Then import and use in the main component:

```tsx
import { InvoiceRow } from './InvoiceRow'

export function InvoiceList({ invoices, onView, onEdit, onDelete }: InvoiceListProps) {
  return (
    <div>
      {invoices.map(invoice => (
        <InvoiceRow
          key={invoice.id}
          invoice={invoice}
          onView={() => onView?.(invoice.id)}
          onEdit={() => onEdit?.(invoice.id)}
          onDelete={() => onDelete?.(invoice.id)}
        />
      ))}
    </div>
  )
}
```

## Step 8: Create the Preview Wrapper

Create a preview wrapper at `src/sections/[section-id]/[ViewName].tsx` (note: this is in the section root, not in components/).

This wrapper is what Design OS renders. It imports the sample data and feeds it to the props-based component.

Example:

```tsx
import data from '@/../product/sections/[section-id]/data.json'
import { InvoiceList } from './components/InvoiceList'

export default function InvoiceListPreview() {
  return (
    <InvoiceList
      invoices={data.invoices}
      onView={(id) => console.log('View invoice:', id)}
      onEdit={(id) => console.log('Edit invoice:', id)}
      onDelete={(id) => console.log('Delete invoice:', id)}
      onCreate={() => console.log('Create new invoice')}
    />
  )
}
```

The preview wrapper:

- Has a `default` export (required for Design OS routing)
- Imports sample data from data.json
- Passes data to the component via props
- Provides console.log handlers for callbacks (for testing interactions)
- Is NOT exported to the user's codebase - it's only for Design OS
- **Will render inside the shell** if one has been designed

## Step 9: Create Component Index

Create an index file at `src/sections/[section-id]/components/index.ts` to cleanly export all components.

Example:

```tsx
export { InvoiceList } from './InvoiceList'
export { InvoiceRow } from './InvoiceRow'
// Add other sub-components as needed
```

## Step 10: Validate Screen Design

Before presenting to the user, validate that the screen design renders correctly without errors.

### Check for Playwright MCP

First, verify that you have access to the Playwright MCP tool. Look for a tool named `browser_navigate` or `mcp__playwright__browser_navigate`.

If the Playwright MCP tool is not available, output this EXACT message to the user (copy it verbatim):

---
To validate the screen design, I need the Playwright MCP server installed. Please run:

```
claude mcp add playwright npx @playwright/mcp@latest
```

Then restart this Claude Code session and I'll validate the screen design automatically.
---

Do not proceed with validation if Playwright MCP is not available. Skip to Step 11 instead.

### Validation Process

If Playwright MCP is available:

1. **Start a fresh dev server**:
  **ALWAYS** start a new dev server.  **NEVER** look for an already running dev server to use.
   ```bash
   npm run dev
   ```
   Run it in the background so you can continue with validation.
   **Save the task_id or shell_id** returned from the background process - you'll need it to stop this specific server later.
   Wait 5 seconds for the dev server to be ready.

2. **Get the port number**:
   Read the dev server output file to find which port it started on. Look for a line like:
   ```
   ➜  Local:   http://localhost:3001/
   ```
   Extract the port number (e.g., 3001) from this line.

3. **Navigate to screen design**:
   - URL: `http://localhost:[PORT]/sections/[section-id]/screen-designs/[ViewName]/fullscreen`
   - Note: Use the exact ViewName (e.g., `CompetitionListView`, not `competition-list`)
   - Wait for the page to fully load (3-5 seconds)

4. **Check for errors**:
   - Use Playwright's console message tools to check for JavaScript errors
   - Look for error-level console messages
   - Check that the page doesn't show "Loading..." or "Screen design not found" indefinitely
   - Verify the screen actually renders with data

5. **Take a validation screenshot** (optional but recommended):
   - Capture at desktop viewport (1280px width recommended)
   - Use full page screenshot to capture scrollable content
   - This helps verify the design looks correct

6. **Stop the dev server**:
   - Always stop the dev server after validation completes
   - Use the task_id/shell_id saved in step 1 to kill ONLY the specific background process you started
   - **NEVER use commands like `ps aux | grep -E "[v]ite|[n]pm.*dev"`** - these will kill ALL running dev servers, not just yours
7. **Close the browser window**:
   - Always close the browser window you opened for testing
   - **NEVER** close any other browser window that might be open, only close the one you opened for your Playwright testing

### Validation Results

**If validation succeeds** (page loads with data, no errors):
- Include this in your completion message: "✅ Screen design validated successfully — renders correctly without errors"
- Mention any warnings if present, but don't block on warnings

**If validation fails** (errors present or screen doesn't render):
- Report the specific errors to the user
- Check if the issue is:
  - Wrong file naming (ViewName doesn't match)
  - Missing default export in preview wrapper
  - Data import issues
  - Component render errors
- Offer to fix the issues before completing
- Don't present the screen design as "complete" until it passes validation
- Always stop the dev server even if validation fails

### Common Validation Issues

- **"Screen design not found"**: The ViewName in the URL doesn't match the filename
- **"Loading..." forever**: Preview wrapper may be missing default export
- **Data errors**: Check that data.json structure matches the TypeScript types
- **Import errors**: Verify all component imports use correct paths

### Example Validation Sequence

```
1. Finding free port... port 3421 available
2. Starting dev server on port 3421...
3. Navigating to screen design at http://localhost:3421/sections/[section-id]/screen-designs/[ViewName]...
4. Checking for errors...
5. ✅ No errors detected
6. Screen renders correctly with sample data
7. Design is responsive and displays properly
8. Stopping dev server...
```

## Step 11: Confirm and Next Steps

Let the user know:

"I've created the screen design for **[Section Title]**:

**Exportable components** (props-based, portable):

- `src/sections/[section-id]/components/[ViewName].tsx`
- `src/sections/[section-id]/components/[SubComponent].tsx` (if created)
- `src/sections/[section-id]/components/index.ts`

**Preview wrapper** (for Design OS only):

- `src/sections/[section-id]/[ViewName].tsx`

[Include validation status here if validation was performed]

[If shell exists]: The screen design renders inside your application shell, showing the full app experience.

[If design tokens exist]: I've applied your color palette ([primary], [secondary], [neutral]) and typography choices.

**To preview:** Navigate to `http://localhost:3000/sections/[section-id]/screen-designs/[ViewName]` in your dev server

**Next steps:**

- Run `/screenshot-design` to capture a screenshot of this screen design for documentation
- If the spec calls for additional views, run `/design-screen` again to create them
- When all sections are complete, run `/export-product` to generate the complete export package"

If the spec indicates additional views are needed:

"The specification also calls for [other view(s)]. Run `/design-screen` again to create those, then `/screenshot-design` to capture each one."

## Important Notes

- ALWAYS read the `frontend-design` skill before creating screen designs
- ALWAYS validate the screen design with Playwright MCP before presenting to the user (if available)
- Components MUST be props-based - never import data.json in exportable components
- The preview wrapper is the ONLY file that imports data.json
- Use TypeScript interfaces from types.ts for all props
- Callbacks should be optional (use `?`) and called with optional chaining (`?.`)
- Sub-components should also be props-based for maximum portability
- Apply design tokens when available for consistent branding
- Screen designs render inside the shell when viewed in Design OS (if shell exists)
- Validation catches common issues: missing exports, data mismatches, import errors

## Step 12: Iteration

**Critical:** The user may come back with requests to change the screen design. Treat these requests as requirements changes and be sure to update the spec and data design to catalog these change requests.