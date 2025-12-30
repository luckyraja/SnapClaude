# Update Collection Command

Scrape my Marvel Snap card collection from Untapped.gg and save it to a CSV file with complete card data (name, cost, power, ability).

## Instructions

Use the Playwright MCP server to automate the following workflow:

### 1. Navigate to Cards Tier List Page
- Open browser: `playwright_navigate` to `https://snap.untapped.gg/en/meta/cards-tier-list`
- Use `headless: false` so the user can see progress
- Set viewport: 1280x720

### 2. Handle Authentication (AUTOMATED)
**Credentials are stored in Playwright MCP environment variables:**
- Username: `UNTAPPED_USERNAME` (raja@rajaperumal.com)
- Password: `UNTAPPED_PASSWORD`

**Login workflow:**
1. Check if login form is visible with `playwright_get_visible_html`
2. If login required:
   - Fill email field: `playwright_fill` with selector `input[type="email"]` or `input[name="email"]`
   - Fill password field: `playwright_fill` with selector `input[type="password"]`
   - Click login button: `playwright_click` on button with "Log in" or "Sign in" text
   - Wait 3 seconds for redirect
3. Verify logged in by checking for user-specific elements

### 3. Click "Owned" Filter
- Use `playwright_evaluate` to click the "Owned" button filter:
```javascript
document.querySelector('button[aria-label="Owned"]')?.click() ||
document.evaluate("//button[contains(text(), 'Owned')]", document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue?.click()
```
- Wait 2 seconds for page to update

### 4. Extract All Owned Cards
**Important:** The page uses virtual scrolling, so we need to scroll and extract iteratively.

Use `playwright_evaluate` with this complete extraction script:

```javascript
(async function() {
  const allCards = new Map();
  let scrollAttempts = 0;
  const maxScrolls = 80;

  function extractVisibleCards() {
    // Find all card detail containers
    const cardContainers = document.querySelectorAll('.sc-ff04f7bf-13.eeNwPG');

    cardContainers.forEach(container => {
      // Get card name from h2
      const nameEl = container.querySelector('h2');
      if (!nameEl) return;
      const name = nameEl.textContent.trim();

      // Get ability from p tag
      const abilityEl = container.querySelector('p');
      const ability = abilityEl ? abilityEl.textContent.trim() : '';

      // Get cost and power from parent div
      const parent = container.parentElement;
      if (!parent) return;

      // Extract cost and power from parent text
      const parentText = parent.textContent;
      const match = parentText.match(/^(-?\d+)(-?\d+)/);
      const cost = match ? match[1] : '';
      const power = match ? match[2] : '';

      if (name && !allCards.has(name)) {
        allCards.set(name, {
          name,
          cost,
          power,
          ability: ability.replace(/"/g, '""') // Escape quotes for CSV
        });
      }
    });
  }

  // Initial extraction
  extractVisibleCards();

  // Scroll through page to load all cards
  for (let i = 0; i < maxScrolls; i++) {
    const beforeCount = allCards.size;

    window.scrollBy(0, 2000);
    await new Promise(r => setTimeout(r, 500));

    extractVisibleCards();

    // Stop if no new cards found for 8 consecutive scrolls
    if (allCards.size === beforeCount) {
      scrollAttempts++;
      if (scrollAttempts > 8) break;
    } else {
      scrollAttempts = 0;
    }
  }

  return {
    totalCards: allCards.size,
    cards: Array.from(allCards.values())
  };
})()
```

### 5. Save to CSV File
- Parse the returned JSON data
- Create CSV content with header: `Card Name,Cost,Power,Ability,Variants,Series`
- Format each card as: `"Card Name","Cost","Power","Ability","1",""`
- Save to: `/Users/raja/projects/SnapClaude/marvel_snap_collection_YYYY-MM-DD_updated.csv`
- Use current date for YYYY-MM-DD

### 6. Compare with Previous Collection (Optional)
- Read previous CSV file if it exists
- Compare card counts and identify new cards
- Display summary of changes

### 7. Close Browser
- Use `playwright_close` to clean up

## Expected Output

Display a summary showing:
- ✓ Total cards extracted: XXX
- ✓ Cards with complete data (cost/power/ability): XXX
- ✓ File saved to: /Users/raja/projects/SnapClaude/marvel_snap_collection_YYYY-MM-DD_updated.csv
- ✓ New cards since last update: X (if applicable)

## Technical Notes

- **Why tier list page?** It has an "Owned" filter button, unlike the cards database page
- **Virtual scrolling:** Page only renders visible cards, requiring scroll + extract loop
- **CSS selectors:** Cards are in `.sc-ff04f7bf-13.eeNwPG` containers with `h2` (name) and `p` (ability)
- **Cost/Power parsing:** Extracted from parent div text that starts with concatenated numbers
- **Deduplication:** Use Map to avoid duplicate cards during scrolling
- **Error handling:** If extraction fails, take a screenshot for debugging

## Environment Setup Required

Credentials must be configured in `~/.claude.json` MCP settings:
```bash
claude mcp add --transport stdio playwright \
  --env UNTAPPED_USERNAME=raja@rajaperumal.com \
  --env UNTAPPED_PASSWORD=kebben-2gajji-pumkiW \
  -- npx -y @executeautomation/playwright-mcp-server
```
