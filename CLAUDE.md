# Bateau Expense – Developer Guide

## Project Overview

**Bateau Expense** is a minimal, offline-first PWA for tracking shared expenses on boats. No build process, no backend, no external dependencies—pure vanilla HTML5/CSS3/JavaScript deployed as a single file to GitHub Pages.

- **Live**: https://ofthestreet.github.io/bateau-expense/
- **License**: MIT
- **Repo**: https://github.com/ofthestreet/bateau-expense

## Tech Stack

- **Framework**: None. Vanilla HTML5, CSS3, ES2020+
- **Storage**: `localStorage` (key: `bateauExpense_v2`)
- **PWA**: Service Worker (`sw.js`) for offline caching, installable home screen
- **Deployment**: GitHub Pages (auto-deploy on push to `main`)
- **Language**: English (EN) + French (FR), switched via UI toggle, persisted to `localStorage` (key: `boatExpenseLang`)

## File Structure

```
bateau-expense/
├── index.html          # Single monolithic file: HTML + CSS + JS (all inline)
├── manifest.json       # PWA manifest (app name, icons, theme)
├── sw.js               # Service Worker (offline caching strategy)
├── icon.svg            # PWA icon (used as apple-touch-icon + favicon)
├── README.md           # Bilingual project documentation (EN/FR)
├── CLAUDE.md           # This file
└── .github/workflows/
    └── deploy.yml      # GitHub Actions: auto-deploy on push to main
```

## Key Concepts & Conventions

### 1. Monolithic HTML File
All code (HTML markup, CSS styling, JavaScript logic) is embedded in a single `index.html` file. This enables:
- Zero build process
- Single-file PWA deployment
- Easy offline caching (one file = entire app)
- Simple local development (open in browser)

### 2. Internationalization (i18n)
- **Strings are NOT hardcoded.** All user-facing text is looked up via a `translations` object.
- **Helper function**: `t(key)` returns translated string for current language.
- **State**: `currentLang` (set from `localStorage.boatExpenseLang`, defaults to `'fr'`).
- **Supported languages**: EN, FR (extensible to ES, DE, IT, etc.).
- **Language toggle**: Header button switches language + saves to localStorage.

### 3. Offline Storage
- **Data key**: `localStorage.boatExpenses_v2` (stores JSON-serialized state)
- **State shape**:
  ```javascript
  {
    participants: ['Alice', 'Bob'],
    expenses: [
      { id: 1234567890, description: 'Fuel', amount: 50.00, paidBy: 'Alice', participants: ['Alice', 'Bob'] }
    ]
  }
  ```
- **Functions**: `loadState()` (on init), `saveState()` (after mutations)

### 4. Service Worker Caching
- **File**: `sw.js`
- **Strategy**: Cache-first (offline-capable)
- **Cache version**: `bateau-expense-v2` (increment to bust cache on deploy)
- **Caches**: `manifest.json`, `index.html`, `sw.js`, `icon.svg`

### 5. Formatting
- **Currency**: `Intl.NumberFormat('fr-FR', { style: 'currency', currency: 'EUR' })`
  - Respects browser/system locale for display (uses `fr-FR` for decimal/grouping formatting)
  - **Note**: Always uses EUR; no currency selection in app.
- **HTML escaping**: `esc(str)` function sanitizes user input before inserting into DOM

## Development Workflow

### Local Development
1. Open `index.html` in any modern browser (Chrome, Safari, Firefox)
2. App loads instantly; no build step or server needed
3. Edit HTML/CSS/JS directly in `index.html`
4. Refresh browser to see changes
5. localStorage persists between refreshes (use DevTools to clear if needed)

### Adding Features
- **UI elements**: Add HTML to modal/shell, style with inline CSS
- **Logic**: Write vanilla JS functions, call them from event handlers
- **Data mutations**: Call `saveState()` after modifying `state` object
- **Strings**: Use `t('key.name')` lookup instead of hardcoding

### Deployment
- Push to `main` branch → GitHub Actions runs `.github/workflows/deploy.yml` → site updates at GitHub Pages
- No build step; raw files deployed as-is

## Internationalization (i18n) Details

### How It Works

**Translation object** (at top of `<script>` block):
```javascript
const translations = {
  en: {
    'app.title': 'Boat Expense',
    'stats.total': 'Total Expenses',
    // ... all strings
  },
  fr: {
    'app.title': 'Bateau Expense',
    'stats.total': 'Total dépenses',
    // ... all strings
  }
};
```

**Helper function**:
```javascript
function t(key) {
  return translations[currentLang][key] || key;
}
```

**Language state**:
```javascript
let currentLang = localStorage.getItem('boatExpenseLang') || 'fr';

function setLanguage(lang) {
  currentLang = lang;
  localStorage.setItem('boatExpenseLang', lang);
  render(); // Re-render UI with new language
}
```

### Adding a New Language

1. Add language block to `translations` object (e.g., `es: { ... }`)
2. Translate all keys (use EN/FR as reference)
3. Test by calling `setLanguage('es')` in DevTools console
4. Add language option to UI toggle button

### Adding a New String

1. Identify the key (e.g., `'dialog.newFeature'`)
2. Add to both `en` and `fr` blocks in translations
3. In code, use `t('dialog.newFeature')` instead of hardcoding

## Key Functions & Modules

### State Management
- **`loadState()`**: Load state from localStorage on init
- **`saveState()`**: Persist state after mutations
- **`state`**: Global object: `{ participants: [], expenses: [] }`

### Rendering
- **`render()`**: Re-render entire UI
- **`renderStats()`**: Update total expense count + amount
- **`renderParticipants()`**: Rebuild participant chip bar
- **`renderExpenses()`**: Rebuild expense list

### Actions
- **`addParticipant(name)`**: Add participant, save, re-render
- **`addExpense(exp)`**: Add expense, save, re-render
- **`confirmRemoveParticipant(name)`**: Show confirm dialog
- **`confirmDeleteExpense(index)`**: Show confirm dialog
- **`confirmClear()`**: Confirm + clear all expenses

### Modals & Dialogs
- **`showAlert(title, msg)`**: Show alert dialog
- **`showConfirm(title, msg, label, callback)`**: Show confirm dialog
- **`showPrompt(title, placeholder, label, callback)`**: Show prompt dialog
- **`showAddParticipant()`**: Open add participant modal
- **`showAddExpense()`**: Open add expense modal
- **`showBalance()`**: Open balance/settlement sheet
- **`openModal(html)`**: Inject modal HTML into DOM root
- **`closeModal()`**: Clear modal root

### Computed State
- **`computeBalances()`**: Calculate per-person balance (positive = owed to them)
- **`computeSettlements()`**: Generate minimal settlement instructions (who pays whom)

### Utilities
- **`t(key)`**: Get translated string for current language
- **`setLanguage(lang)`**: Switch language + re-render
- **`esc(str)`**: Escape HTML special chars (XSS prevention)
- **`fmtCur(amount)`**: Format number as currency (EUR)

## Design System

### Colors (CSS variables)
- Primary: `--blue: #1a6bbf`
- Dark blue: `--blue-dark: #0f4d8a`
- Light blue: `--blue-light: #2d8ce8`
- Success: `--green: #30d158`
- Danger: `--red: #ff453a`
- Background: `--bg: #f2f2f7`
- Card: `--card: #ffffff`
- Text (primary): `--label: #1c1c1e`
- Text (secondary): `--secondary: #3c3c43cc`
- Text (tertiary): `--tertiary: #3c3c4399`
- Separator: `--separator: rgba(60,60,67,0.12)` (iOS-style divider)

### Layout
- **Header**: Sticky, gradient blue background, safe-area insets for notch/navigation bars
- **Modals**: Bottom sheet (iOS-style), blur overlay, smooth animations
- **FAB**: Floating action button (bottom-right), fixed position with safe-area support
- **Stats bar**: Inline stats in header bar
- **Participant bar**: Horizontal chip scroll

## Browser & Device Support

- **Modern browsers** (Chrome, Safari, Firefox, Edge)
- **iOS**: PWA capable, home screen installation, safe-area support
- **Android**: PWA capable via Chrome/Chromium
- **Responsive**: Mobile-first design, works 375px–desktop widths
- **Offline**: Service Worker caches all assets

## Testing Checklist

- [ ] Language toggle switches UI from FR to EN and back
- [ ] Language preference persists after page refresh
- [ ] Add participant → name shows in chip bar
- [ ] Add expense → appears in list, affects balance
- [ ] Delete expense → confirm dialog, removes from list
- [ ] Clear all → confirm dialog, resets expenses
- [ ] Balance sheet → calculates correct per-person balances
- [ ] Settlements → calculates minimal payment instructions
- [ ] Offline: Open app, toggle airplane mode, still works
- [ ] PWA install: iOS "Add to Home Screen" + Android install banner
- [ ] Layouts: Test on phone, tablet, desktop

## Git Workflow

- **Branch naming**: Descriptive snake_case (e.g., `feat/language-toggle`, `fix/balance-calc`)
- **Commits**: Conventional Commits style (e.g., `feat: add EN/FR language support`, `fix: settlement calculation`)
- **PR workflow**: Push to feature branch → open PR against `main` → merge after review
- **Deployment**: Auto-triggers on push to `main` via GitHub Actions

## Troubleshooting

**Strings not translating?**
- Check key exists in both `en` and `fr` blocks of `translations` object
- Check code uses `t('key')` not hardcoded string
- Check `currentLang` is set correctly (DevTools: `console.log(currentLang)`)

**Data not persisting?**
- Check `saveState()` is called after mutations
- Check `localStorage` quota not exceeded (DevTools → Application → Storage)
- Clear cache: `localStorage.removeItem('bateauExpense_v2')`

**Service Worker not caching?**
- Increment cache version in `sw.js` (`bateau-expense-v3`, etc.)
- Unregister old SW: DevTools → Application → Service Workers → Unregister
- Hard refresh: Ctrl+Shift+R or Cmd+Shift+R

**PWA won't install?**
- Check `manifest.json` is served with correct MIME type
- Check HTTPS (required for PWA)
- Check Web App Install Banner requirements (manifest + icons + service worker)
