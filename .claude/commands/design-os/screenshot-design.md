# Screenshot Screen Design

You are helping the user capture a screenshot of a screen design they've created. The screenshot will be saved to the product folder for documentation purposes.

## Prerequisites: Check for Playwright MCP

Before proceeding, verify that you have access to the Playwright MCP tool. Look for a tool named `browser_take_screenshot` or `mcp__playwright__browser_take_screenshot`.

If the Playwright MCP tool is not available, output this EXACT message to the user (copy it verbatim, do not modify or "correct" it):

---
To capture screenshots, I need the Playwright MCP server installed. Please run:

```
claude mcp add playwright npx @playwright/mcp@latest
```

Then restart this Claude Code session and run `/screenshot-design` again.
---

Do not substitute different package names or modify the command. Output it exactly as written above.

Do not proceed with the rest of this command if Playwright MCP is not available.

## Step 1: Identify the Screen Design

First, determine which screen design to screenshot.

Read `/product/product-roadmap.md` to get the list of available sections, then check `src/sections/` to see what screen designs exist.

If only one screen design exists across all sections, auto-select it.

If multiple screen designs exist, use the AskUserQuestion tool to ask which one to screenshot:

"Which screen design would you like to screenshot?"

Present the available screen designs as options, grouped by section:
- [Section Name] / [ScreenDesignName]
- [Section Name] / [ScreenDesignName]

## Step 2: Start Dev Server

**Start a fresh dev server**:

*If Playwright MCP is available:

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

3. **Navigate to shell preview**:
   - URL: `http://localhost:[PORT]/shell/design`
   - Wait for the page to fully load (3-5 seconds)

4. **Check for errors**:
   - Use Playwright's console message tools to check for JavaScript errors
   - Look for error-level console messages
   - Check that the page doesn't show "Loading..." indefinitely



## Step 3: Capture Comprehensive Screenshots

**IMPORTANT**: You must capture comprehensive screenshots that show ALL states, sections, and interactions of the screen design. This means multiple screenshots showing different views, not just one default screenshot. If you fail to capture a full comprehensive set of screenshots, you will have completely failed in your task and will be wasting everyone's time.

Use the Playwright MCP tool to navigate to the screen design and capture screenshots.

The screen design URL pattern is: `http://localhost:[PORT]/sections/[section-id]/screen-designs/[screen-design-name]/fullscreen`

**Note**: Use the `/fullscreen` route to view the screen design without the Design OS chrome (navigation header).

### Initial Setup

1. Use `browser_navigate` to go to the fullscreen URL (using the port from Step 2)
2. Wait 2-3 seconds for the page to fully load and any dynamic content to render
3. Use `browser_snapshot` to understand the page structure

### Screenshot Strategy

You must capture screenshots showing:

**1. All Major Sections**
- If the page has multiple sections (like Contact, Participant, Judge, Organizer), scroll to each section and capture it
- Use `browser_run_code` or `browser_evaluate` to scroll to specific sections by finding headings
- Capture overlapping screenshots so all content is documented

**2. All Interactive States**
- Filter states (if there are filters, capture each filter option)
- Search functionality (show a search in action)
- Collapsed/Expanded states (if sections can be toggled)
- Form states (empty, filled, validation errors if applicable)
- Any dialogs, tabs, accordions, or other interactive UI

**3. BOTH Light and Dark Modes**
- After capturing all states in one mode, toggle the theme
- Use `browser_evaluate` to toggle: `document.documentElement.classList.toggle('dark')`
- If this doesn't work, try interactively clicking the toggle button in the UI
  - Note that the toggle may be nested in a user menu or somewhere else, you may have to look at the code to find it
- If it *still* doesn't work, **ask the user to toggle the mode manually**
- **DO NOT** skip this!  You **MUST** capture all screenshots in **both** light and dark modes, if available
- Capture the same critical views in the other mode

**4. Different Viewport Widths**
- Screenshots should be captured to represent **both** mobile and desktop viewports
- Capture the same critical views in both viewport widths

**4. Special Features**
- Unsaved changes indicators
- Success/error states
- Loading states (if easily reproducible)
- Empty states
- Populated states with realistic data

### Screenshot Specifications

- Desktop viewport width (default Playwright viewport)
- PNG format for best quality
- Use descriptive filenames that indicate what the screenshot shows
- Use `browser_take_screenshot` with `fullPage: true` for full page captures
- Use `browser_take_screenshot` without fullPage for viewport-specific captures when scrolled to a section

### Verification Requirement

<critical>
<seriously_critical>
<absoulutely_fucking_critical_don't_you_dare_ignore_this>
**CRITICAL**: After capturing each screenshot, you MUST verify it shows what it's supposed to show by looking at the returned screenshot image. If a screenshot doesn't show the expected content (e.g., you tried to capture the bottom section but it's showing the top), adjust your approach and retake it. You **MUST** do this after **every** screenshot you capture. **DO NOT** capture a batch of screenshots and then go verify them. Verify **each screenshot** right after you capture it

**When Verifying Screenshots, Make Sure:**
- The screenshot shows the correct page
- The screenshot shows the correct section of the page
- The screenshot properly demonstrates the intended functionality
- The screenshot demonstrates the intended theme
- The screenshot demonstrates mobile/desktop as intended

If the screenshot fails to fulfill any of these, you **MUST** adjust and retake the screenshot.
</absoulutely_fucking_critical_don't_you_dare_ignore_this>
</seriously_critical>
</critical>

### Example Comprehensive Capture Process

For a profile management page with 4 sections and filter functionality:

1. Navigate to fullscreen URL
2. Capture top sections (Contact, Participant) - viewport screenshot
3. Scroll to Judge section, capture it - viewport screenshot
4. Scroll to bottom (Organizer), capture it - viewport screenshot
5. If there are filters: capture each filter state
6. If there's search: type a search term and capture results
7. Toggle to light/dark mode
8. Repeat critical screenshots in the other mode
9. Verify each screenshot shows the intended content

### Scrolling for Page Sections

The page may have internal scrollable containers. To scroll to specific sections:

```javascript
// Find the scrollable main container
const mains = document.querySelectorAll('main');
const main = Array.from(mains).find(m => m.scrollHeight > m.clientHeight);

// Scroll to a specific section
const heading = await page.getByRole('heading', { name: 'Section Name' }).first();
await heading.scrollIntoViewIfNeeded();

// Or scroll to bottom
if (main) {
  main.scrollTop = main.scrollHeight - main.clientHeight;
}
```

## Step 4: Save the Screenshots

The Playwright MCP tool can only save screenshots to its default output directory (`.playwright-mcp/`). You must save the screenshots there first, then copy them to the product folder.

1. **First**, use `browser_take_screenshot` with just a filename (no path):
   - Use descriptive filenames that indicate what the screenshot shows
   - The file will be saved to `.playwright-mcp/[filename].png`

2. **Then**, copy all screenshots to the product folder using Bash:
   ```bash
   cp .playwright-mcp/[pattern]*.png product/sections/[section-id]/
   ```

**Naming convention:** `[screen-design-name]-[what-it-shows]-[mode].png`

Examples:
- `profile-contact-section-dark.png` (Contact section in dark mode)
- `profile-judge-volunteer-section-dark.png` (Judge section in dark mode)
- `profile-organizer-complete-dark.png` (Bottom section with approval in dark mode)
- `style-prefs-default.png` (Default view with all filters)
- `style-prefs-filter-prefer.png` (Filtered to show only "Prefer" categories)
- `style-prefs-search.png` (Search functionality in action)
- `style-prefs-beer-collapsed.png` (Beer section collapsed)
- `style-prefs-unsaved-changes.png` (Showing unsaved changes indicator)

The filenames should be descriptive enough that someone can understand what each screenshot shows without opening it.

**Stop the dev server**:
- Always stop the dev server after screenshots are captured
- Kill the background process that was started in Step 2

## Step 5: Shut down the dev server 
**Stop the dev server**:
   - Always stop the dev server after validation completes
   - Use the task_id/shell_id saved in step 1 to kill ONLY the specific background process you started
   - **NEVER use commands like `ps aux | grep -E "[v]ite|[n]pm.*dev"`** - these will kill ALL running dev servers, not just yours

**Close the browser window**:
   - Always close the browser window you opened for testing
   - **NEVER** close any other browser window that might be open, only close the one you opened for your Playwright testing

## Step 5: Confirm Completion

Provide a summary of all captured screenshots:

"I've captured comprehensive screenshots for the **[ScreenDesignName]** screen design and saved them to `product/sections/[section-id]/`:

**[ScreenDesignName]** ([N] screenshots):
1. **[filename].png** - [Brief description of what it shows]
2. **[filename].png** - [Brief description of what it shows]
3. **[filename].png** - [Brief description of what it shows]
...

These screenshots document:
- [List the key features/states captured, e.g., "All 4 major sections"]
- [e.g., "Different filter states (All, Prefer, OK to Judge, Dislike)"]
- [e.g., "Search functionality"]
- [e.g., "Collapsed/expanded states"]
- [e.g., "Both light and dark modes" or "Dark mode"]"

## Important Notes

### Server Management
- Check if dev server is already running on port 3000 before starting a new one
- Start the dev server yourself only if not running - do not ask the user to do it
- **Track the process ID (PID)**: When you start a dev server with `run_in_background: true`, save the task_id or shell_id returned
- **Only stop what you started**: Use the saved task_id/shell_id to kill ONLY the specific background process you started
- **NEVER kill all dev servers**: Do NOT use commands like `ps aux | grep -E "[v]ite|[n]pm.*dev"` followed by kill commands - these will shut down ALL running dev servers, not just yours
- Leave servers running if they were already running when you started

### Screenshot Quality and Coverage
- **BE THOROUGH**: Capture multiple screenshots showing all sections, states, and interactions
- **VERIFY EACH SCREENSHOT**: Look at the returned image to confirm it shows what you intended
- If a screenshot doesn't show the expected content, adjust your scrolling/approach and retake it
- Screenshots are saved to `product/sections/[section-id]/` alongside spec.md and data.json
- Use descriptive filenames that clearly indicate what each screenshot shows
- Capture at a consistent viewport width for documentation consistency
- Use fullPage screenshots for overview captures, viewport screenshots for scrolled sections

### What to Capture
- All major page sections (scroll to show the full view for each one)
- All filter/search states
- All modal dialogs
- Collapsed/expanded states for accordions or toggles
- Unsaved changes indicators
- Success/error states where applicable
- Both light and dark modes for critical views
- Any special features or interactive elements
- Validation error samples

### Common Scrolling Patterns
- Pages with internal scrollable containers require JavaScript to scroll the correct element
- Use `browser_run_code` or `browser_evaluate` to find and scroll the scrollable container
- Use `browser_snapshot` first to understand the page structure
- Verify scrolling worked by checking the screenshot output

### Success Metrics:
- ✅ 100% of scenarios include BOTH light and dark mode screenshots
- ✅ 100% of interactive elements captured in screenshots
- ✅ Zero instances of marking todo complete without satisfying completion criteria
- ✅ All blocking failures result in user notification, not rationalization