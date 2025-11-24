# Repository Integration Guide: Repo B → Repo A

## Purpose

This guide provides step-by-step instructions for importing this repository (azure-search-openai-demo, Repo B) into the px.ai.ops.dash monorepo (Repo A).

## Prerequisites

Before starting the integration:

- [ ] Access to both repositories (Repo A and Repo B)
- [ ] Understanding of the target monorepo structure (see migration plan)
- [ ] Node.js 20+ installed
- [ ] Python 3.10+ installed
- [ ] PNPM installed (for Repo A)
- [ ] UV installed (for Python package management)
- [ ] Git configured for the monorepo

## Target Structure in Repo A

After integration, the structure will be:

```
px.ai.ops.dash/
├── apps/
│   ├── web/              # Existing: @px/web (shadcn)
│   ├── api-ts/           # Existing: @px/api (BFF)
│   └── legacy-web/       # NEW: @px/legacy-web (from Repo B frontend)
├── packages/
│   └── shared/           # Existing: @px/shared
├── services/
│   ├── api-py/           # Existing: FastAPI service
│   └── legacy-api-py/    # NEW: from Repo B backend
└── docs/
```

## Integration Steps

### Phase 1: Initial Import

#### Step 1.1: Create Feature Branch in Repo A

```bash
cd /path/to/px.ai.ops.dash
git checkout main
git pull origin main
git switch -c feature/import-legacy-app
```

#### Step 1.2: Create Target Directories

```bash
# In px.ai.ops.dash root
mkdir -p apps/legacy-web
mkdir -p services/legacy-api-py
```

#### Step 1.3: Copy Frontend Code

```bash
# Assuming Repo B is cloned at ../azure-search-openai-demo
cp -R ../azure-search-openai-demo/app/frontend/* apps/legacy-web/

# Verify the copy
ls -la apps/legacy-web/
```

Expected files in `apps/legacy-web/`:
- `src/` - Source code
- `public/` - Static assets
- `package.json` - Dependencies
- `tsconfig.json` - TypeScript config
- `vite.config.ts` - Vite config
- `index.html` - HTML template

#### Step 1.4: Copy Backend Code

```bash
# Copy backend Python code
cp -R ../azure-search-openai-demo/app/backend/* services/legacy-api-py/

# Copy Azure Functions if needed
mkdir -p services/legacy-api-py/functions
cp -R ../azure-search-openai-demo/app/functions/* services/legacy-api-py/functions/

# Verify the copy
ls -la services/legacy-api-py/
```

Expected files in `services/legacy-api-py/`:
- `app.py` - Main application
- `requirements.txt` - Python dependencies
- `approaches/` - RAG implementations
- `prepdocslib/` - Document ingestion library
- `config.py` - Configuration
- Other Python modules

#### Step 1.5: Create Workspace Package for Frontend

Edit `apps/legacy-web/package.json` to make it a workspace package:

```json
{
  "name": "@px/legacy-web",
  "private": true,
  "version": "0.0.1",
  "type": "module",
  "engines": {
    "node": ">=20.0.0"
  },
  "scripts": {
    "dev": "vite --host 127.0.0.1",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "@azure/msal-browser": "^4.16.0",
    "@azure/msal-react": "^3.0.16",
    "@fluentui/react": "^8.112.5",
    "@fluentui/react-components": "^9.56.2",
    "@fluentui/react-icons": "^2.0.265",
    "@react-spring/web": "^9.7.5",
    "dompurify": "^3.2.4",
    "i18next": "^24.2.0",
    "i18next-browser-languagedetector": "^8.0.2",
    "i18next-http-backend": "^3.0.1",
    "idb": "^8.0.0",
    "ndjson-readablestream": "^1.2.0",
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-helmet-async": "^2.0.5",
    "react-i18next": "^15.4.1",
    "react-markdown": "^9.0.1",
    "react-router-dom": "^6.28.0",
    "react-syntax-highlighter": "^16.1.0",
    "rehype-raw": "^7.0.0",
    "remark-gfm": "^4.0.0",
    "scheduler": "^0.20.2"
  },
  "devDependencies": {
    "@types/dom-speech-recognition": "^0.0.7",
    "@types/dompurify": "^3.0.5",
    "@types/react": "^18.3.12",
    "@types/react-dom": "^18.3.1",
    "@types/react-syntax-highlighter": "^15.5.13",
    "@vitejs/plugin-react": "^4.3.3",
    "prettier": "^3.3.3",
    "rollup-plugin-visualizer": "^5.12.0",
    "typescript": "^5.6.3",
    "vite": "^6.0.0"
  }
}
```

#### Step 1.6: Update Workspace Configuration

Edit `pnpm-workspace.yaml` in Repo A root (or create if it doesn't exist):

```yaml
packages:
  - "apps/*"
  - "packages/*"
  - "services/*"  # Ensure this line exists
```

This ensures PNPM recognizes all workspace packages including the new ones.

#### Step 1.7: Add Root Scripts

Edit `package.json` in Repo A root to add scripts for the legacy app:

```json
{
  "scripts": {
    "dev": "pnpm -r dev",
    "dev:web": "pnpm --filter @px/web dev",
    "dev:api": "pnpm --filter @px/api dev",
    "dev:py": "cd services/api-py && uv run python -m api_py.main",
    
    "dev:legacy-web": "pnpm --filter @px/legacy-web dev",
    "dev:legacy-py": "cd services/legacy-api-py && uv run python -m quart --app app:app run --port 3052 --reload",
    
    "build": "pnpm -r build",
    "build:legacy-web": "pnpm --filter @px/legacy-web build",
    
    "type-check": "pnpm -r type-check",
    "type-check:legacy-web": "pnpm --filter @px/legacy-web type-check"
  }
}
```

#### Step 1.8: Create README for Legacy Frontend

Create `apps/legacy-web/README.md`:

```markdown
# Legacy Web Frontend (@px/legacy-web)

This is the legacy RAG chat application frontend, imported from azure-search-openai-demo.

## Technology Stack

- React 18
- TypeScript
- Vite
- Fluent UI (v8 and v9) - **TO BE MIGRATED to shadcn/ui**
- react-router-dom
- MSAL for authentication

## Development

```bash
# From monorepo root
pnpm dev:legacy-web

# Or from this directory
pnpm dev
```

The app will be available at http://localhost:5173

## Building

```bash
# From monorepo root
pnpm build:legacy-web

# Or from this directory
pnpm build
```

## Migration Status

This application is in the process of being migrated from Fluent UI to shadcn/ui to align with the @px/web standards.

See migration documentation:
- `docs/fluent-to-shadcn-mapping.md` - Component migration guide
- `docs/repo-b-migration-overview.md` - Complete repository analysis

## Configuration

The app connects to the backend API at the port specified in environment variables or defaults to port 50505.

Environment variables are loaded from `.env` files (not committed).
```

#### Step 1.9: Create README for Legacy Backend

Create `services/legacy-api-py/README.md`:

```markdown
# Legacy Python API (services/legacy-api-py)

This is the legacy RAG chat backend, imported from azure-search-openai-demo.

## Technology Stack

- Python 3.10+
- Quart (async Flask)
- Azure OpenAI
- Azure AI Search
- Azure Cosmos DB (chat history)
- Azure Blob Storage

## Development

### Setup Environment

```bash
cd services/legacy-api-py

# Create virtual environment (if not using uv)
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Or use uv (recommended)
uv sync
```

### Run Development Server

```bash
# From monorepo root
pnpm dev:legacy-py

# Or directly
cd services/legacy-api-py
uv run python -m quart --app app:app run --port 3052 --reload
```

The API will be available at http://localhost:3052

## Configuration

The backend requires Azure service connections. Configuration is loaded from:
1. Environment variables
2. `.env` files (not committed)
3. Azure App Configuration (when deployed)

Required environment variables:
- `AZURE_OPENAI_ENDPOINT`
- `AZURE_OPENAI_KEY` or use Azure Identity
- `AZURE_SEARCH_SERVICE`
- `AZURE_SEARCH_INDEX`
- And others (see config.py for full list)

## API Endpoints

Key routes:
- `POST /ask` - Simple Q&A
- `POST /chat` - Chat with history
- `GET /config` - Frontend configuration
- `POST /upload` - Upload documents
- `GET /history/list` - List chat history

See `docs/http_protocol.md` for full API documentation.

## Testing

```bash
pytest
pytest --cov
```

## Port Configuration

Default port: **3052** (chosen to avoid conflict with existing services)

Ports in use:
- 3050: TypeScript BFF (@px/api)
- 3051: Python FastAPI service (services/api-py)
- 3052: Legacy Python API (this service)
```

#### Step 1.10: Install Dependencies

```bash
# From Repo A root
pnpm install

# This will install dependencies for all workspace packages including @px/legacy-web
```

#### Step 1.11: Set Up Python Environment for Legacy Backend

```bash
cd services/legacy-api-py

# Option 1: Using uv (recommended)
uv sync

# Option 2: Using pip
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

#### Step 1.12: Test the Import

```bash
# Test frontend build
pnpm dev:legacy-web
# Open http://localhost:5173 in browser

# In another terminal, test backend (requires Azure credentials)
pnpm dev:legacy-py
# Should start on http://localhost:3052
```

#### Step 1.13: Commit Initial Import

```bash
git add apps/legacy-web services/legacy-api-py
git add pnpm-workspace.yaml package.json pnpm-lock.yaml
git commit -m "feat: import legacy web and Python backend into monorepo

- Add apps/legacy-web (@px/legacy-web) from azure-search-openai-demo frontend
- Add services/legacy-api-py from azure-search-openai-demo backend
- Update workspace configuration to include new packages
- Add dev scripts for legacy services
- Add README files for development setup

This is the initial import before Fluent UI → shadcn/ui migration."

git push origin feature/import-legacy-app
```

#### Step 1.14: Open Pull Request

Create a PR with:
- **Title**: "feat: import legacy app into monorepo"
- **Description**: Use the PR template, link to migration plan
- **Labels**: `enhancement`, `migration`, `legacy-app`

### Phase 2: Tooling Alignment

**Goal**: Align `@px/legacy-web` with `@px/web` tooling standards (TypeScript, Tailwind, shadcn/ui).

#### Step 2.1: Review @px/web Configuration

```bash
# Study the existing @px/web setup
cat apps/web/tsconfig.json
cat apps/web/tailwind.config.js  # or .ts
cat apps/web/postcss.config.js
cat apps/web/vite.config.ts
```

Identify:
- TypeScript path aliases (e.g., `@/components`)
- Tailwind configuration
- shadcn/ui setup
- Vite plugins and settings

#### Step 2.2: Align TypeScript Configuration

Update `apps/legacy-web/tsconfig.json` to match `@px/web` patterns:

```json
{
  "extends": "../../tsconfig.base.json",  // If you have a base config
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    
    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    
    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    
    /* Path aliases - align with @px/web */
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/lib/*": ["./src/lib/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

#### Step 2.3: Install Tailwind CSS and Dependencies

```bash
cd apps/legacy-web

# Install Tailwind and related packages
pnpm add -D tailwindcss postcss autoprefixer
pnpm add -D tailwindcss-animate
pnpm add class-variance-authority clsx tailwind-merge

# Initialize Tailwind (this creates tailwind.config.js and postcss.config.js)
npx tailwindcss init -p
```

#### Step 2.4: Configure Tailwind

Update `apps/legacy-web/tailwind.config.js` to match `@px/web`:

```js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: ["class"],
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
}
```

#### Step 2.5: Set Up shadcn/ui

```bash
cd apps/legacy-web

# Install shadcn CLI
pnpm add -D shadcn-ui

# Initialize shadcn (this creates components.json)
npx shadcn-ui@latest init
```

When prompted, choose:
- Style: **Default**
- Base color: **Slate** (or match @px/web)
- CSS variables: **Yes**
- Import alias: **@/components** or **@/components/ui**

Update `components.json` to match `@px/web` if it exists there.

#### Step 2.6: Add CSS Variables

Update `apps/legacy-web/src/index.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

Keep any existing app-specific styles below these base styles.

#### Step 2.7: Add Utility Functions

Create `apps/legacy-web/src/lib/utils.ts`:

```typescript
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

#### Step 2.8: Install Initial shadcn Components

Install the most commonly used components:

```bash
cd apps/legacy-web

npx shadcn-ui@latest add button
npx shadcn-ui@latest add input
npx shadcn-ui@latest add label
npx shadcn-ui@latest add select
npx shadcn-ui@latest add checkbox
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add sheet
npx shadcn-ui@latest add popover
npx shadcn-ui@latest add tooltip
npx shadcn-ui@latest add alert
npx shadcn-ui@latest add tabs
```

#### Step 2.9: Verify Build

```bash
# From monorepo root
pnpm build:legacy-web

# Should succeed (Fluent UI still present but not conflicting)
```

#### Step 2.10: Commit Tooling Changes

```bash
git add apps/legacy-web
git commit -m "feat(legacy-web): align tooling with @px/web

- Add Tailwind CSS configuration
- Set up shadcn/ui with base components
- Align TypeScript configuration
- Add utility functions (cn)
- Update CSS with design tokens

Fluent UI is still present and functional. Component migration starts next."

git push origin feature/import-legacy-app
```

### Phase 3: Component Migration (Iterative)

**Goal**: Replace Fluent UI components with shadcn/ui equivalents.

This phase is done iteratively, one component type at a time, with testing after each step.

#### Migration Order (Recommended)

1. **Buttons** (lowest risk)
2. **Layout (Stack → Tailwind)** (low risk)
3. **Form inputs** (medium risk)
4. **Panels and dialogs** (medium risk)
5. **Complex components** (higher risk)

#### Step 3.1: Migrate Buttons

**Files to update** (from inventory):
- `pages/ask/Ask.tsx`
- `pages/chat/Chat.tsx`
- `pages/layout/Layout.tsx`
- `components/Answer/Answer.tsx`
- `components/AnswerError.tsx`
- `components/HelpCallout/HelpCallout.tsx`
- `components/HistoryItem/HistoryItem.tsx`
- `components/LoginButton/LoginButton.tsx`
- `components/ClearChatButton/ClearChatButton.tsx`
- `components/HistoryButton/HistoryButton.tsx`
- `components/SettingsButton/SettingsButton.tsx`

For each file:
1. Replace Fluent button imports with shadcn Button
2. Map props according to `fluent-to-shadcn-mapping.md`
3. Test functionality

**Example diff** for one file:

```diff
- import { DefaultButton, PrimaryButton, IconButton } from "@fluentui/react";
+ import { Button } from "@/components/ui/button";
+ import { Trash2 } from "lucide-react";

- <DefaultButton text="Clear chat" onClick={clearChat} />
+ <Button variant="outline" onClick={clearChat}>Clear chat</Button>

- <PrimaryButton text="Submit" onClick={onSubmit} />
+ <Button onClick={onSubmit}>Submit</Button>

- <IconButton iconProps={{ iconName: 'Delete' }} onClick={onDelete} />
+ <Button variant="ghost" size="icon" onClick={onDelete}>
+   <Trash2 className="h-4 w-4" />
+ </Button>
```

**Commit after buttons:**
```bash
git add apps/legacy-web
git commit -m "feat(legacy-web): migrate buttons from Fluent UI to shadcn

- Replace DefaultButton, PrimaryButton with shadcn Button
- Replace IconButton with shadcn Button + icons
- Update button styling with Tailwind variants

Tested: All button interactions work correctly"
git push
```

#### Step 3.2: Migrate Layout (Stack)

**Files to update**:
- All files using `Stack` component (15+ files)

Replace `Stack` with Tailwind flex utilities:

```diff
- import { Stack } from "@fluentui/react";
- 
- <Stack tokens={{ childrenGap: 10 }} styles={{ root: { padding: 20 } }}>
-   <div>Item 1</div>
-   <div>Item 2</div>
- </Stack>

+ <div className="flex flex-col gap-2.5 p-5">
+   <div>Item 1</div>
+   <div>Item 2</div>
+ </div>
```

**Commit after layout:**
```bash
git commit -m "feat(legacy-web): migrate Stack components to Tailwind flex

- Replace all Stack components with div + Tailwind flex
- Map tokens.childrenGap to Tailwind gap-*
- Preserve spacing and alignment

Tested: Layouts render correctly across all pages"
```

#### Step 3.3: Migrate Form Inputs

**Files to update**:
- `components/Settings/Settings.tsx` (high priority - many form controls)
- `components/VectorSettings/VectorSettings.tsx`
- `components/QuestionInput/QuestionInput.tsx`

See `fluent-to-shadcn-mapping.md` for detailed TextField → Input mapping.

**Commit after forms:**
```bash
git commit -m "feat(legacy-web): migrate form controls from Fluent to shadcn

- Replace TextField with Input + Label
- Replace Dropdown with Select
- Replace Checkbox with shadcn Checkbox
- Preserve validation logic and error messages

Tested: All form interactions, validation, and submission work"
```

#### Step 3.4: Migrate Panels and Dialogs

**Files to update**:
- `pages/ask/Ask.tsx` - Panel
- `pages/chat/Chat.tsx` - Panel
- `components/HistoryPanel/HistoryPanel.tsx` - Panel
- `components/HelpCallout/HelpCallout.tsx` - Callout
- `components/UploadFile/UploadFile.tsx` - Callout

```diff
- import { Panel, PanelType } from "@fluentui/react";
+ import { Sheet, SheetContent, SheetHeader, SheetTitle } from "@/components/ui/sheet";

- <Panel
-   isOpen={isOpen}
-   onDismiss={onClose}
-   headerText="Settings"
-   type={PanelType.medium}
- >
+ <Sheet open={isOpen} onOpenChange={setIsOpen}>
+   <SheetContent className="w-96">
+     <SheetHeader>
+       <SheetTitle>Settings</SheetTitle>
+     </SheetHeader>
```

**Commit after panels:**
```bash
git commit -m "feat(legacy-web): migrate panels and overlays to shadcn

- Replace Panel with Sheet for side panels
- Replace Callout with Popover
- Preserve open/close behavior and content

Tested: All panels open, close, and function correctly"
```

#### Step 3.5: Migrate Remaining Components

Continue with:
- Pivot/PivotItem → Tabs (AnalysisPanel)
- Spinner → Custom spinner (various files)
- MessageBar → Alert (MarkdownViewer)
- Link, Label, Text → HTML + Tailwind

### Phase 4: Final Cleanup

#### Step 4.1: Remove Fluent UI Dependencies

Edit `apps/legacy-web/package.json`:

```diff
  "dependencies": {
-   "@fluentui/react": "^8.112.5",
-   "@fluentui/react-components": "^9.56.2",
-   "@fluentui/react-icons": "^2.0.265",
    "@azure/msal-browser": "^4.16.0",
    ...
  }
```

```bash
cd apps/legacy-web
pnpm install  # This will remove Fluent UI from node_modules
```

#### Step 4.2: Verify No Fluent Imports Remain

```bash
cd apps/legacy-web
grep -r "@fluentui" src/
# Should return nothing

grep -r "from '@fluentui" src/
# Should return nothing
```

#### Step 4.3: Remove Fluent Initialization

In `src/index.tsx`, remove:

```diff
- import { initializeIcons } from "@fluentui/react";
- 
- initializeIcons();
```

#### Step 4.4: Update README

Update `apps/legacy-web/README.md`:

```diff
- Fluent UI (v8 and v9) - **TO BE MIGRATED to shadcn/ui**
+ shadcn/ui (Radix UI + Tailwind CSS)
```

#### Step 4.5: Final Build and Test

```bash
pnpm build:legacy-web
pnpm dev:legacy-web

# Test all features:
# - Chat functionality
# - Settings panel
# - File upload
# - History panel
# - All button interactions
# - Form submissions
```

#### Step 4.6: Final Commit

```bash
git add apps/legacy-web
git commit -m "feat(legacy-web): complete Fluent UI to shadcn migration

- Remove all Fluent UI dependencies
- Verify no Fluent imports remain
- Update documentation
- All components now use shadcn/ui

Tested: Complete application functionality verified"
git push
```

#### Step 4.7: Open PR for Review

Create final PR:
- **Title**: "feat: complete Fluent UI to shadcn/ui migration for legacy-web"
- **Description**: 
  - Link to migration tracking issue
  - List all migrated components
  - Describe testing performed
  - Screenshots of key pages (before/after if available)

### Phase 5: Backend Integration (Optional Next Steps)

#### Step 5.1: Align Python Backend with Repo A Standards

- Review `services/api-py` for conventions
- Update logging, error handling to match
- Align configuration management
- Update port assignments

#### Step 5.2: Integration with TypeScript BFF

Decide on strategy:
- **Option A**: Keep as separate service, route through BFF
- **Option B**: Merge functionality into existing `services/api-py`
- **Option C**: Keep separate with direct frontend connection

#### Step 5.3: Environment Configuration

- Move Azure service configs to monorepo-wide `.env`
- Update deployment scripts
- Align with Repo A's Azure provisioning

## Common Issues and Solutions

### Issue: PNPM Install Fails

**Symptom**: Dependency resolution errors after adding `@px/legacy-web`

**Solution**:
```bash
# Clear PNPM cache
pnpm store prune

# Remove node_modules and lock file
rm -rf node_modules pnpm-lock.yaml

# Reinstall
pnpm install
```

### Issue: TypeScript Path Alias Not Resolving

**Symptom**: `Cannot find module '@/components/ui/button'`

**Solution**:
- Verify `tsconfig.json` has correct `paths` configuration
- Check `vite.config.ts` has alias resolution:
  ```ts
  import path from "path"
  
  export default defineConfig({
    resolve: {
      alias: {
        "@": path.resolve(__dirname, "./src"),
      },
    },
  })
  ```

### Issue: Tailwind Classes Not Applied

**Symptom**: Components have no styling

**Solution**:
- Verify `tailwind.config.js` content paths include all source files
- Ensure `index.css` imports Tailwind directives
- Check PostCSS config exists
- Restart dev server

### Issue: shadcn Components Not Found

**Symptom**: `Module not found: '@/components/ui/button'`

**Solution**:
- Run `npx shadcn-ui@latest add button` to install the component
- Check `components.json` has correct paths
- Verify `@/components` alias is configured

### Issue: Python Backend Port Conflict

**Symptom**: "Port 50505 already in use"

**Solution**:
- Choose different port (3052 recommended)
- Update start scripts in root `package.json`
- Update frontend API base URL if needed

## Testing Checklist

After integration, verify:

### Frontend
- [ ] `pnpm install` succeeds
- [ ] `pnpm dev:legacy-web` starts dev server
- [ ] `pnpm build:legacy-web` builds successfully
- [ ] Application loads in browser
- [ ] No console errors related to Fluent UI
- [ ] All pages navigate correctly
- [ ] Forms submit correctly
- [ ] Panels/dialogs open and close
- [ ] Icons display correctly
- [ ] Styles look correct

### Backend
- [ ] Python environment activates
- [ ] Dependencies install successfully
- [ ] `pnpm dev:legacy-py` starts server
- [ ] Health check endpoint responds
- [ ] API endpoints return expected responses
- [ ] Authentication works (if configured)

### Integration
- [ ] Frontend can call backend API
- [ ] CORS configured correctly
- [ ] Authentication flows work end-to-end

## Rollback Plan

If integration fails or causes issues:

```bash
# From feature branch
git reset --hard origin/main

# Or from main after merge
git revert <merge-commit-sha>
```

## Post-Integration Tasks

After successful integration:

1. **Update Documentation**
   - Add legacy app to main README
   - Update architecture diagrams
   - Document new scripts and commands

2. **Update CI/CD**
   - Add build jobs for `@px/legacy-web`
   - Add test jobs for legacy backend
   - Update deployment workflows

3. **Team Onboarding**
   - Share new repository structure
   - Document how to run legacy app
   - Explain migration status and next steps

4. **Monitoring**
   - Set up alerts for legacy services
   - Monitor build times and bundle sizes
   - Track migration progress

## Next Steps After Integration

1. Continue Fluent UI migration (Phase 3)
2. Align authentication with Repo A
3. Consider routing migration (react-router-dom → TanStack Router)
4. Evaluate state management migration (hooks → Zustand)
5. Plan backend consolidation strategy

## Resources

- Migration Plan Document (provided)
- `docs/repo-b-migration-overview.md`
- `docs/fluent-to-shadcn-mapping.md`
- shadcn/ui docs: https://ui.shadcn.com
- PNPM workspaces: https://pnpm.io/workspaces

---

**Document Version**: 1.0  
**Last Updated**: 2025-11-24  
**Contact**: Migration team lead
