# Migration Quick Reference Guide

## At a Glance

**What:** Integrate azure-search-openai-demo (Repo B) into px.ai.ops.dash (Repo A)  
**Timeline:** 6-8 weeks  
**Risk Level:** Medium  
**Status:** Ready to start

## Quick Links

- **Full Migration Plan**: See original plan document
- **Repo B Overview**: `docs/repo-b-migration-overview.md`
- **Component Mapping**: `docs/fluent-to-shadcn-mapping.md`
- **Integration Steps**: `docs/repo-a-integration-guide.md`
- **Readiness Assessment**: `docs/migration-readiness-assessment.md`

## Repository Structure

### Current (Repo B)
```
azure-search-openai-demo/
├── app/
│   ├── frontend/    # React + Fluent UI
│   └── backend/     # Python Quart
```

### Target (Repo A)
```
px.ai.ops.dash/
├── apps/
│   ├── web/         # Existing (@px/web, shadcn)
│   └── legacy-web/  # NEW from Repo B frontend (@px/legacy-web)
├── services/
│   ├── api-py/      # Existing (FastAPI)
│   └── legacy-api-py/  # NEW from Repo B backend
```

## Tech Stack Comparison

| Aspect | Repo B (Current) | Repo A (Target) | Action Required |
|--------|------------------|-----------------|-----------------|
| **Frontend Framework** | React 18 | React 19 | ✅ Compatible |
| **UI Library** | Fluent UI v8/v9 | shadcn/ui | ❌ Migrate |
| **Build Tool** | Vite 6 | Vite | ✅ Compatible |
| **TypeScript** | 5.6 | Latest | ✅ Compatible |
| **Routing** | react-router-dom | TanStack Router | ⚠️ Future work |
| **State Mgmt** | React hooks | Zustand | ⚠️ Future work |
| **Backend** | Python Quart | Python FastAPI | ⚠️ Different framework |
| **Package Manager** | npm | PNPM | ✅ Compatible |
| **Auth** | MSAL | MSAL (assumed) | ✅ Compatible |

## Migration Phases

### Phase 1: Import (1-2 days)
```bash
# Copy code to Repo A
cp -R repo-b/app/frontend/* apps/legacy-web/
cp -R repo-b/app/backend/* services/legacy-api-py/

# Update package.json, workspace config
# Install dependencies
pnpm install
```

**Deliverable:** Code compiles in monorepo

### Phase 2: Tooling (2-3 days)
```bash
# Add Tailwind, shadcn/ui
cd apps/legacy-web
pnpm add -D tailwindcss postcss autoprefixer
npx shadcn-ui@latest init

# Align TypeScript config
# Update import paths
```

**Deliverable:** Tooling matches @px/web

### Phase 3: Components (2-3 weeks)

**Week 1:** Buttons + Layout
```bash
# Migrate DefaultButton → Button (variant="outline")
# Migrate PrimaryButton → Button (variant="default")
# Migrate IconButton → Button (size="icon")
# Replace Stack → Tailwind flex
```

**Week 2:** Forms
```bash
# TextField → Input + Label
# Dropdown → Select
# Checkbox → Checkbox + Label
```

**Week 3:** Complex Components
```bash
# Panel → Sheet
# Callout → Popover
# Pivot → Tabs
# Spinner → Custom component
```

**Deliverable:** Zero Fluent UI imports

### Phase 4: Backend (1 week)
```bash
# Align config with Repo A
# Set port to 3052
# Update deployment scripts
```

**Deliverable:** Backend runs in monorepo

### Phase 5: Cleanup (2-3 days)
```bash
# Remove Fluent UI deps
# Update docs
# Final testing
```

**Deliverable:** Production-ready

## Common Commands

### Development
```bash
# Start legacy frontend
pnpm dev:legacy-web

# Start legacy backend
pnpm dev:legacy-py

# Build legacy frontend
pnpm build:legacy-web

# Type check
pnpm type-check:legacy-web
```

### Testing
```bash
# Run tests
pytest

# Run with coverage
pytest --cov

# E2E tests
pytest e2e.py
```

### Component Migration
```bash
# Install shadcn component
npx shadcn-ui@latest add button

# Check for Fluent UI imports
grep -r "@fluentui" src/

# Check bundle size
npm run build && ls -lh dist/
```

## Component Migration Checklist

### Buttons (8 files)
- [ ] Ask.tsx
- [ ] Chat.tsx
- [ ] Layout.tsx
- [ ] Answer.tsx
- [ ] AnswerError.tsx
- [ ] HelpCallout.tsx
- [ ] HistoryItem.tsx
- [ ] LoginButton.tsx

### Layout (15+ files)
- [ ] Replace all `Stack` with `div` + Tailwind flex

### Forms (3 files)
- [ ] Settings.tsx ⚠️ HIGH PRIORITY
- [ ] VectorSettings.tsx
- [ ] QuestionInput.tsx

### Overlays (5 files)
- [ ] Ask.tsx (Panel)
- [ ] Chat.tsx (Panel)
- [ ] HistoryPanel.tsx (Panel)
- [ ] HelpCallout.tsx (Callout)
- [ ] UploadFile.tsx (Callout)

### Navigation (1 file)
- [ ] AnalysisPanel.tsx (Pivot/PivotItem)

### Feedback (3 files)
- [ ] Ask.tsx (Spinner)
- [ ] HistoryPanel.tsx (Spinner)
- [ ] MarkdownViewer.tsx (MessageBar)

## Fluent → shadcn Quick Map

| Fluent Component | shadcn Equivalent | Complexity |
|-----------------|-------------------|------------|
| `DefaultButton` | `<Button variant="outline">` | Low |
| `PrimaryButton` | `<Button>` | Low |
| `IconButton` | `<Button variant="ghost" size="icon">` | Low |
| `TextField` | `<Input>` + `<Label>` | Medium |
| `Dropdown` | `<Select>` + subcomponents | Medium |
| `Checkbox` | `<Checkbox>` + `<Label>` | Low |
| `Panel` | `<Sheet>` + subcomponents | Medium |
| `Callout` | `<Popover>` + subcomponents | Medium |
| `Pivot/PivotItem` | `<Tabs>` + subcomponents | Medium |
| `Stack` | `<div className="flex ...">` | Low |
| `Spinner` | Custom + `<Loader2>` | Low |
| `MessageBar` | `<Alert>` + subcomponents | Medium |

## Icon Migration

| Fluent Icon | Lucide Icon |
|------------|-------------|
| `Delete24Regular` | `Trash2` |
| `Add24Regular` | `Plus` |
| `Settings24Regular` | `Settings` |
| `Sparkle28Filled` | `Sparkles` |
| `History24Regular` | `History` |
| `Mic28Filled` | `Mic` |
| `Send28Filled` | `Send` |
| `ErrorCircle24Regular` | `AlertCircle` |

```bash
# Install Lucide
pnpm add lucide-react
```

## Code Snippets

### Button Migration
```tsx
// Before
import { DefaultButton } from "@fluentui/react";
<DefaultButton text="Click me" onClick={handleClick} />

// After
import { Button } from "@/components/ui/button";
<Button variant="outline" onClick={handleClick}>Click me</Button>
```

### Stack → Flex
```tsx
// Before
<Stack tokens={{ childrenGap: 10 }}>
  <div>Item 1</div>
  <div>Item 2</div>
</Stack>

// After
<div className="flex flex-col gap-2.5">
  <div>Item 1</div>
  <div>Item 2</div>
</div>
```

### TextField → Input
```tsx
// Before
<TextField 
  label="Name"
  value={name}
  onChange={(e, v) => setName(v || '')}
/>

// After
<div className="space-y-2">
  <Label htmlFor="name">Name</Label>
  <Input 
    id="name"
    value={name}
    onChange={(e) => setName(e.target.value)}
  />
</div>
```

### Panel → Sheet
```tsx
// Before
<Panel 
  isOpen={isOpen} 
  onDismiss={onClose}
  headerText="Settings"
>
  Content
</Panel>

// After
<Sheet open={isOpen} onOpenChange={setIsOpen}>
  <SheetContent>
    <SheetHeader>
      <SheetTitle>Settings</SheetTitle>
    </SheetHeader>
    <div>Content</div>
  </SheetContent>
</Sheet>
```

## Gap Mapping (Stack → Tailwind)

| Fluent `childrenGap` | Tailwind `gap-*` |
|---------------------|------------------|
| 4 | `gap-1` |
| 8 | `gap-2` |
| 10 | `gap-2.5` |
| 12 | `gap-3` |
| 16 | `gap-4` |
| 20 | `gap-5` |
| 24 | `gap-6` |

## Testing After Each Change

```bash
# 1. Build succeeds
pnpm build:legacy-web

# 2. Type check passes
pnpm type-check:legacy-web

# 3. Dev server runs
pnpm dev:legacy-web

# 4. Manual testing
# - Navigate to all pages
# - Test all interactions
# - Verify no console errors
```

## Troubleshooting

### Issue: Module not found '@/components/ui/button'
```bash
# Install the component
npx shadcn-ui@latest add button

# Check tsconfig.json paths
"paths": {
  "@/*": ["./src/*"]
}
```

### Issue: Tailwind classes not working
```bash
# Check tailwind.config.js content paths
content: ["./src/**/*.{js,ts,jsx,tsx}"]

# Check index.css has Tailwind directives
@tailwind base;
@tailwind components;
@tailwind utilities;

# Restart dev server
```

### Issue: "isOpen is not a function" with Sheet/Dialog
```bash
# Fluent used: isOpen, onDismiss
<Panel isOpen={isOpen} onDismiss={() => setIsOpen(false)} />

# shadcn uses: open, onOpenChange
<Sheet open={isOpen} onOpenChange={setIsOpen}>
```

### Issue: Event handler signature changed
```bash
# Fluent TextField
onChange={(e, newValue) => setValue(newValue || '')}

# shadcn Input
onChange={(e) => setValue(e.target.value)}
```

## Success Criteria

✅ **Build Phase**
- `pnpm install` succeeds
- `pnpm build:legacy-web` succeeds
- No TypeScript errors
- No console warnings

✅ **Migration Phase**
- Zero `@fluentui` imports
- All pages load
- All features work
- No UI regressions

✅ **Production Phase**
- All tests pass
- Bundle size acceptable
- Performance good
- Documentation updated

## Resources

### Documentation
- shadcn/ui: https://ui.shadcn.com
- Tailwind: https://tailwindcss.com
- Lucide Icons: https://lucide.dev
- Radix UI: https://www.radix-ui.com

### Tools
- Component gallery: https://ui.shadcn.com/docs/components
- Tailwind classes: https://tailwindcss.com/docs
- Color palette: https://ui.shadcn.com/themes

### Internal
- Repo A: px.ai.ops.dash
- Component examples: `apps/web/src/components`
- Shared configs: `tsconfig.base.json`, etc.

## Communication

### Daily Standup Topics
- What was migrated yesterday?
- What's being migrated today?
- Any blockers or issues?
- Test results

### Weekly Report
- Component migration progress (%)
- Files modified
- Tests status
- Risks/issues
- Next week plan

### Escalation Path
1. Team lead (day-to-day issues)
2. Tech lead (architecture questions)
3. Product owner (scope/timeline questions)

## Contacts

| Role | Responsibility |
|------|----------------|
| Migration Lead | Overall coordination |
| Frontend Engineer | Component migration |
| Backend Engineer | Backend integration |
| QA Engineer | Testing & validation |
| DevOps | CI/CD updates |

## Next Steps

1. [ ] Review this guide with the team
2. [ ] Set up Repo A feature branch
3. [ ] Begin Phase 1: Import
4. [ ] Daily progress tracking
5. [ ] Weekly check-ins

---

**Keep this guide handy during the migration!**

Last updated: 2025-11-24
