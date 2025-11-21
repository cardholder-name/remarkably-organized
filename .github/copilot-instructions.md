# Remarkably Organized - Copilot Instructions

## Project Overview
A SvelteKit web app for generating customizable, print-ready planners for e-ink tablets (Remarkable 2). Users configure planner layouts via URL-encoded settings, preview in-browser, and export to PDF using Chrome's print-to-PDF feature.

## Architecture

### Core State Management Pattern
- **Svelte 5 Runes**: All state uses `$state`, `$derived`, and `$effect` runes (no stores or legacy reactivity)
- **Central State Class**: `PlannerSettings` (`src/lib/state/planner-settings.svelte.ts`) manages all configuration as a class with nested setting objects
- **URL-Driven State**: Settings serialize to JSON in URL query params for shareable links; changes auto-sync via `replaceState`
- **Derived Timeframes**: `years`, `quarters`, `months`, `weeks`, `days` are `$derived` computed arrays from `date.start`/`date.end`

### Component Hierarchy
```
+page.svelte (main planner UI)
├── CoverPage, YearPage, QuarterPage, MonthPage, WeekPage, DayPage
├── CollectionPages (custom note sections)
└── Page.svelte (universal page template renderer)
    └── Grid, CalendarMonth, AgendaWeek, NotesDay, HabitsYear, etc.
```

### Page Generation Flow
1. User modifies settings in sidebar → updates `PlannerSettings` state
2. `PlannerSettings` computes timeframes (years/months/weeks/days) from date range
3. Main template loops over timeframes, rendering page components for each
4. Each page uses `Page.svelte` with a `display` prop matching `PageTemplate` types
5. Print-to-PDF exports all rendered pages with high-res option

## Critical Development Patterns

### Date Handling
- **Always use UTC**: All date math in `src/lib/helpers/date.helper.ts` uses UTC methods to avoid timezone bugs
- **Week calculations**: `getFirstDayOfWeek()` and `getWeek()` handle Sunday/Monday start weeks with complex ISO week logic
- **Timeframe IDs**: Format as `${year}-${month}-${day}` or `${year}-wk${weekNumber}` for internal linking

### Component Props Convention
```typescript
// Use $props rune with defaults and TypeScript types
let {
  settings = {} as PlannerSettings,
  timeframe = {} as Timeframe,
  display = 'dotted' as Collection['type']
} = $props();

// Derive computed values using $derived
const size = $derived(
  display.endsWith('large') ? 'large' : 
  display.endsWith('small') ? 'small' : 'medium'
);
```

### Styling System
- **SCSS with Auto-Imports**: `_variables.scss` auto-imported via Vite config (`additionalData`)
- **CSS Custom Properties**: Main theme in `global.scss` defines `--text`, `--bg`, `--outline`, etc.
- **Print Optimization**: All pages sized for e-ink (702×936px default, 1404×1872px high-res)
- **Dynamic Styles**: Page-level CSS vars set via `style:` directives (e.g., `style:--font="'{font.name}'"`)

### Icon Usage
- **unplugin-icons**: Import as Svelte components: `import SettingsIcon from '~icons/fa/cog'`
- **No bundled icon files**: Icons compiled at build time

## Key Files

### Configuration Entry Points
- `src/routes/planner/+page.ts` - Deserializes URL settings, returns `PlannerSettings` instance
- `src/routes/planner/+page.svelte` - Main planner UI with settings modal
- `vite.config.ts` - SCSS auto-import config, unplugin-icons setup

### State & Types
- `src/lib/state/planner-settings.svelte.ts` - Central state class with serialize/deserialize
- `src/lib/state/collection.ts` - `PageTemplate` union type (all valid page layouts)
- `src/lib/state/event.ts` - Calendar event types

### Reusable Components
- `src/lib/components/Page.svelte` - Routes `display` prop to correct sub-component
- `src/lib/components/Grid.svelte` - Renders dotted/lined/numbered grids
- `src/lib/components/Toast.svelte` - Global notifications via `toast()` helper

### API Routes
- `src/routes/api/calendar/+server.ts` - Fetches/parses iCal feeds using `ical.js`

## Common Tasks

### Adding a New Page Template
1. Add type to `PageTemplate` union in `src/lib/state/collection.ts`
2. Create component in `src/lib/components/` (e.g., `NotesCustom.svelte`)
3. Add conditional in `Page.svelte`: `{#if display === 'notes-custom'}<NotesCustom />{/if}`
4. Update `pageTemplates` array in `+page.svelte` with user-facing name
5. Add to `getAvailablePageTemplates()` filter logic if location-specific

### Debugging Print Layout
- Preview: Open in Chrome, check "Background Graphics" in print dialog
- High-res: Toggle `enableHighResolution` state to test 2× scaling
- Page breaks: Ensure `<article>` elements (auto page-break boundaries)

### Working with Calendar Events
- Events stored as `{ name: string, start: number, duration?: number }` (Unix timestamps in seconds)
- Imported via `/api/calendar?url=...&start=...&end=...` endpoint
- Recurring events expanded server-side using `ical.js` iterator pattern

## Development Commands
```bash
pnpm i          # Install dependencies
pnpm dev        # Dev server (localhost:5173)
pnpm build      # Production build
pnpm check      # TypeScript + Svelte type checking
```

## Testing Workflow
- **No automated tests**: Manual browser testing required
- **Print validation**: Always test print-to-PDF for layout regressions
- **URL sharing**: Verify settings persist correctly in query params

## Gotchas
- **Svelte 5 only**: No legacy `$:` reactivity or `writable()` stores
- **Print media queries**: Body styles differ between screen/print (`@media screen` vs `@media print`)
- **Date math errors**: Always use UTC helpers from `date.helper.ts` to avoid off-by-one errors
- **Large PDFs**: Memory-intensive; warn users about performance with multi-year planners
