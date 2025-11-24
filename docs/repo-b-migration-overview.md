# Repo B Migration Overview

## Executive Summary

This document provides a comprehensive analysis of the **azure-search-openai-demo** repository (Repo B) to facilitate its integration into the **px.ai.ops.dash** monorepo (Repo A). This repository is a RAG (Retrieval Augmented Generation) chat application built with React + TypeScript frontend using Fluent UI, and a Python Quart backend.

## Repository Identity

- **Repository Name**: azure-search-openai-demo
- **Role in Migration**: Repo B (Legacy Application)
- **Target Integration**: Will be imported into px.ai.ops.dash as `apps/legacy-web` and `services/legacy-api-py`

## Current Architecture

### High-Level Structure

```
azure-search-openai-demo/
├── app/
│   ├── frontend/          # React + Vite + TypeScript (Fluent UI)
│   ├── backend/           # Python Quart backend
│   └── functions/         # Azure Functions for cloud ingestion
├── data/                  # Sample data files
├── docs/                  # Existing documentation (25+ files)
├── infra/                 # Bicep templates for Azure deployment
├── scripts/               # Deployment and setup scripts
└── tests/                 # Test suite (pytest)
```

### Application Ports

Based on the codebase analysis, the application uses:
- **Frontend (Vite)**: Port 5173 (default) or 50505 (production)
- **Backend (Quart)**: Port 50505 (configurable)

## Technology Stack Analysis

### Frontend (`app/frontend/`)

**Core Technologies:**
- **React**: 18.3.1
- **TypeScript**: 5.6.3
- **Build Tool**: Vite 6.0.0
- **Routing**: react-router-dom 6.28.0

**UI Framework - Fluent UI:**
- `@fluentui/react`: 8.112.5 (v8 - older API)
- `@fluentui/react-components`: 9.56.2 (v9 - newer API)
- `@fluentui/react-icons`: 2.0.265

**Authentication:**
- `@azure/msal-browser`: 4.16.0
- `@azure/msal-react`: 3.0.16

**Other Key Dependencies:**
- `react-markdown`: 9.0.1 (for rendering chat messages)
- `react-syntax-highlighter`: 16.1.0
- `i18next`: 24.2.0 (internationalization)
- `dompurify`: 3.2.4 (XSS protection)
- `idb`: 8.0.0 (IndexedDB wrapper)

**Development Dependencies:**
- Prettier for code formatting
- TypeScript with strict type checking
- Vite plugin for React

### Backend (`app/backend/`)

**Framework:**
- **Quart**: Async Python web framework (Flask-compatible API)
- **Python Version**: 3.10+

**Key Azure Services:**
- Azure OpenAI (GPT models)
- Azure AI Search (vector search, semantic ranking)
- Azure Cosmos DB (chat history)
- Azure Blob Storage (document storage)
- Azure Document Intelligence
- Azure Cognitive Services Speech
- Azure Functions (for cloud ingestion pipeline)

**Key Python Libraries:**
- `openai`: OpenAI/Azure OpenAI client
- `azure-search-documents`: Azure AI Search client
- `azure-cosmos`: Cosmos DB client
- `azure-identity`: Azure authentication
- `azure-storage-blob`: Blob storage operations
- `quart-cors`: CORS support
- `opentelemetry`: Observability and monitoring
- `tiktoken`: Token counting for OpenAI models

**Backend Structure:**
```
app/backend/
├── approaches/           # RAG approach implementations
│   ├── chatreadretrieveread.py  # Chat with history
│   ├── retrievethenread.py      # Simple Q&A
│   └── prompts/                 # Prompty templates
├── chat_history/        # Chat history management (CosmosDB)
├── core/                # Core utilities (auth, sessions)
├── prepdocslib/         # Document ingestion library
├── app.py               # Main Quart application
├── config.py            # Configuration management
└── requirements.txt     # Python dependencies
```

### Functions (`app/functions/`)

Azure Functions for cloud ingestion pipeline:
- **document_extractor**: Extract content from documents
- **figure_processor**: Process images and figures
- **text_processor**: Process and chunk text

Each function contains a synchronized copy of `prepdocslib`.

## Fluent UI Component Inventory

### Fluent UI Packages in Use

The frontend uses THREE different Fluent UI package sets:

1. **@fluentui/react** (v8 - legacy API)
2. **@fluentui/react-components** (v9 - modern API)
3. **@fluentui/react-icons** (shared icons)
4. **@fluentui/react-hooks** (utility hooks)
5. **@fluentui/react-table** (table components)

### Components by Category

#### Layout & Structure
- **Stack**: Used extensively for layout (15+ files)
  - Files: Answer.tsx, AnswerLoading.tsx, Settings.tsx, VectorSettings.tsx, QuestionInput.tsx, HelpCallout.tsx, AnalysisPanel/ThoughtProcess.tsx, AnswerError.tsx

#### Buttons
- **DefaultButton**: Standard action buttons (8 files)
  - Files: Ask.tsx, Chat.tsx, HelpCallout.tsx, HistoryItem.tsx, LoginButton.tsx
- **PrimaryButton**: Primary action buttons (1 file)
  - Files: AnswerError.tsx
- **IconButton**: Icon-only buttons (8 files)
  - Files: Answer.tsx, SpeechOutputAzure.tsx, SpeechOutputBrowser.tsx, HelpCallout.tsx, MarkdownViewer.tsx, Layout.tsx
- **Button** (v9): Modern button component (7 files)
  - Files: ClearChatButton.tsx, HistoryButton.tsx, QuestionInput.tsx, SpeechInput.tsx, SettingsButton.tsx, UploadFile.tsx

#### Forms & Inputs
- **TextField**: Text input fields (3 files)
  - Files: Settings.tsx, QuestionInput.tsx
- **Checkbox**: Checkbox inputs (2 files)
  - Files: Settings.tsx, VectorSettings.tsx
- **Dropdown**: Dropdown selects (4 files)
  - Files: Settings.tsx, VectorSettings.tsx, LanguagePicker.tsx

#### Overlays & Panels
- **Panel**: Side panel (3 files)
  - Files: Ask.tsx, Chat.tsx, HistoryPanel.tsx
- **Callout**: Popup callout (2 files)
  - Files: HelpCallout.tsx, UploadFile.tsx

#### Feedback & Status
- **Spinner**: Loading indicators (3 files)
  - Files: Ask.tsx, HistoryPanel.tsx, MarkdownViewer.tsx
- **MessageBar**: Status messages (1 file)
  - Files: MarkdownViewer.tsx

#### Navigation
- **Pivot/PivotItem**: Tab navigation (1 file)
  - Files: AnalysisPanel.tsx

#### Data Display
- **Link**: Hyperlinks (1 file)
  - Files: MarkdownViewer.tsx
- **Label**: Text labels (2 files)
  - Files: UploadFile.tsx, TokenClaimsDisplay.tsx
- **Text**: Text display (1 file)
  - Files: UploadFile.tsx

#### Icons (from @fluentui/react-icons)
- **Sparkle28Filled**: AI/assistant icon
- **ErrorCircle24Regular**: Error state icon
- **Delete24Regular**: Delete action icon
- **Add24Regular**: Add action icon
- **History24Regular**: History button icon
- **Mic28Filled**: Speech input icon
- **Send28Filled**: Send message icon
- **Settings24Regular**: Settings icon
- **LocalLanguage24Regular**: Language picker icon

#### Utilities
- **useId**: Generate unique IDs (4 files)
- **useBoolean**: Boolean state hook (1 file)
- **initializeIcons**: Initialize Fluent icon font (1 file)
- **ITextFieldProps, ICheckboxProps, IDropdownProps, IDropdownOption**: TypeScript interfaces
- **IButtonStyles, IStackStyles, IStackTokens**: Styling interfaces

## Component File Mapping

### Critical Components Using Fluent UI

| Component | File Path | Fluent UI Components Used | Complexity |
|-----------|-----------|---------------------------|------------|
| Ask (Q&A Page) | `pages/ask/Ask.tsx` | Panel, DefaultButton, Spinner | Medium |
| Chat (Chat Page) | `pages/chat/Chat.tsx` | Panel, DefaultButton | Medium |
| Layout (App Shell) | `pages/layout/Layout.tsx` | IconButton | Low |
| Settings | `components/Settings/Settings.tsx` | TextField, Checkbox, Dropdown, Stack | High |
| VectorSettings | `components/VectorSettings/VectorSettings.tsx` | Stack, Dropdown, Checkbox | Medium |
| QuestionInput | `components/QuestionInput/QuestionInput.tsx` | Stack, TextField, Button, Tooltip | Medium |
| Answer | `components/Answer/Answer.tsx` | Stack, IconButton | Medium |
| AnswerError | `components/Answer/AnswerError.tsx` | Stack, PrimaryButton | Low |
| AnalysisPanel | `components/AnalysisPanel/AnalysisPanel.tsx` | Pivot, PivotItem | Medium |
| HistoryPanel | `components/HistoryPanel/HistoryPanel.tsx` | Panel, PanelType, Spinner | Medium |
| HelpCallout | `components/HelpCallout/HelpCallout.tsx` | DefaultButton, IconButton, Callout, Stack | High |
| MarkdownViewer | `components/MarkdownViewer/MarkdownViewer.tsx` | Spinner, MessageBar, Link, IconButton | Medium |
| UploadFile | `components/UploadFile/UploadFile.tsx` | Button, Callout, Label, Text | Medium |
| LanguagePicker | `i18n/LanguagePicker.tsx` | Dropdown | Low |

## API Contracts & Integration Points

### Backend API Routes

The backend exposes the following key routes (from `app.py`):

**Core RAG Routes:**
- `POST /ask` - Simple Q&A endpoint
- `POST /chat` - Chat with history endpoint
- `GET /config` - Frontend configuration
- `POST /speech` - Text-to-speech synthesis

**Document Management:**
- `POST /upload` - Upload user documents
- `POST /delete_uploaded` - Delete uploaded documents
- `GET /list_uploaded` - List user's uploaded documents

**Chat History (CosmosDB):**
- `GET /history/list` - List conversation history
- `GET /history/read` - Read specific conversation
- `POST /history/update` - Update conversation
- `DELETE /history/delete` - Delete conversation
- `DELETE /history/delete_all` - Clear all history
- `POST /history/rename` - Rename conversation
- `POST /history/ensure` - Ensure conversation exists

**Health & Info:**
- `GET /` - Serves frontend index.html
- `GET /.auth/me` - Authentication status (Azure App Service)

### Authentication Flow

The application uses Microsoft Authentication Library (MSAL) for Azure AD authentication:

1. **Frontend**: `@azure/msal-react` with `authConfig.ts`
2. **Backend**: Azure App Service Easy Auth (`/.auth/me` endpoint)
3. **Access Control**: Document-level security using security filters in Azure AI Search

## Internationalization (i18n)

The application supports multiple languages via `i18next`:

**Supported Languages:**
- English (en) - default
- Danish (da)
- Spanish (es)
- French (fr)
- Italian (it)
- Japanese (ja)
- Dutch (nl)
- Portuguese/Brazil (ptBR)
- Turkish (tr)

Translation files located at: `app/frontend/src/locales/{lang}/translation.json`

## State Management

**Frontend State:**
- React hooks (useState, useEffect) for local state
- Context API for authentication (loginContext.tsx)
- IndexedDB (via `idb`) for client-side chat history storage
- No external state management library (no Redux, Zustand, etc.)

**Backend State:**
- Stateless request/response model
- Session data stored in Cosmos DB (chat history)
- Document indexing state in Azure AI Search

## Build & Development Workflow

### Frontend Build

```bash
cd app/frontend
npm install
npm run dev          # Development server
npm run build        # Production build
npm run preview      # Preview production build
```

**Build Output:**
- Vite bundles to `app/frontend/dist/`
- Backend serves from `app/backend/static/` (copied during deployment)

### Backend Development

```bash
cd app/backend
python -m venv .venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
pip install -r requirements.txt
python -m quart --app app:app run --port 50505 --reload
```

Or use the convenience scripts:
- `app/start.sh` (Linux/Mac)
- `app/start.ps1` (Windows)

### Testing

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov --cov-report=html

# Run specific test file
pytest tests/test_app.py
```

Test structure:
- `tests/test_app.py` - Backend API tests
- `tests/test_*.py` - Unit tests for various modules
- `tests/conftest.py` - Pytest fixtures and mocks

## Deployment Architecture

### Azure Resources Provisioned

The application deploys to Azure with the following resources (via Bicep templates in `infra/`):

1. **App Service** or **Container Apps** - Hosts the application
2. **Azure OpenAI** - GPT models for chat and embeddings
3. **Azure AI Search** - Vector search and document indexing
4. **Azure Storage Account** - Document blob storage
5. **Azure Cosmos DB** (optional) - Chat history persistence
6. **Azure Document Intelligence** (optional) - Document parsing
7. **Azure Speech Services** (optional) - Text-to-speech
8. **Azure Monitor** - Application insights and logging

### Deployment Methods

- **Azure Developer CLI (azd)**: `azd up`
- **GitHub Actions**: `.github/workflows/azure-dev.yml`
- **Azure DevOps**: `.azdo/pipelines/azure-dev.yml`

## Integration Considerations for Repo A

### Alignment Points

**Similarities with Repo A:**
- ✅ Uses Vite for frontend build
- ✅ TypeScript frontend
- ✅ React-based UI
- ✅ Node.js 20+ requirement
- ✅ Uses PNPM-compatible lockfile format

**Differences Requiring Migration:**
- ❌ Uses Fluent UI → needs migration to shadcn/ui
- ❌ Uses react-router-dom → Repo A uses TanStack Router
- ❌ No state management library → Repo A uses Zustand
- ❌ Uses Quart (Python) → different from Repo A's FastAPI
- ❌ Different authentication setup (MSAL vs Repo A's approach)
- ❌ No monorepo structure → needs workspace integration

### Migration Complexity Assessment

**Low Complexity:**
- Basic components (buttons, text inputs, checkboxes)
- Layout and spacing (Stack → Tailwind flex/grid)
- Icons (Fluent icons → Lucide icons or similar)

**Medium Complexity:**
- Forms and validation (preserve controlled state)
- Panels and overlays (Dialog/Sheet mapping)
- Settings and configuration UI

**High Complexity:**
- Analysis panel with Pivot tabs
- File upload with drag-and-drop
- Help callout system
- Markdown viewer with syntax highlighting
- Speech input/output integration
- Chat history management UI

### Recommended Migration Phases

**Phase 1: Import & Setup**
1. Copy `app/frontend/` → `apps/legacy-web/` in Repo A
2. Copy `app/backend/` → `services/legacy-api-py/` in Repo A
3. Create workspace package configuration
4. Ensure build and dev scripts work

**Phase 2: Tooling Alignment**
1. Align TypeScript configuration with `@px/web`
2. Set up Tailwind CSS
3. Configure shadcn/ui
4. Update import paths and module resolution

**Phase 3: Component Migration (Iterative)**
1. Start with layout shell and navigation
2. Migrate primitive components (buttons, inputs)
3. Migrate forms and settings
4. Migrate complex overlays and panels
5. Migrate specialized features (speech, file upload)

**Phase 4: Backend Integration**
1. Align Python backend with Repo A conventions
2. Decide on consolidation vs. separation strategy
3. Update BFF routing if needed
4. Migrate environment configuration

**Phase 5: Cleanup**
1. Remove Fluent UI dependencies
2. Remove unused code and files
3. Update documentation
4. Final testing and validation

## Dependencies to Remove

**Frontend (`package.json`):**
```json
{
  "@fluentui/react": "^8.112.5",
  "@fluentui/react-components": "^9.56.2",
  "@fluentui/react-icons": "^2.0.265"
}
```

**Potential Additions for shadcn/ui:**
```json
{
  "@radix-ui/react-*": "various versions",
  "class-variance-authority": "^0.7.0",
  "clsx": "^2.0.0",
  "tailwind-merge": "^2.0.0",
  "tailwindcss": "^3.4.0",
  "tailwindcss-animate": "^1.0.7"
}
```

## Key Files for Migration Team

### Must Review Files

**Frontend:**
- `app/frontend/package.json` - Dependencies
- `app/frontend/vite.config.ts` - Build configuration
- `app/frontend/tsconfig.json` - TypeScript settings
- `app/frontend/src/index.tsx` - App entry point
- `app/frontend/src/pages/layout/Layout.tsx` - Main layout
- `app/frontend/src/authConfig.ts` - MSAL configuration

**Backend:**
- `app/backend/app.py` - Main application
- `app/backend/config.py` - Configuration
- `app/backend/requirements.txt` - Python dependencies

### Fluent UI Hotspot Files

Files with heavy Fluent UI usage (priority for migration):
1. `components/Settings/Settings.tsx` (10+ Fluent components)
2. `components/HelpCallout/HelpCallout.tsx` (Complex callout with state)
3. `components/VectorSettings/VectorSettings.tsx` (Form with validation)
4. `pages/ask/Ask.tsx` (Panel, buttons, spinner)
5. `pages/chat/Chat.tsx` (Panel, buttons)
6. `components/HistoryPanel/HistoryPanel.tsx` (Panel with list)

## Repository-Specific Features

### Unique Features to Preserve

1. **Multimodal Support**: Image understanding in RAG responses
2. **Speech I/O**: Browser and Azure Speech Services integration
3. **Document Upload**: User file upload and indexing
4. **Chat History**: Browser (IndexedDB) and server (Cosmos DB) storage
5. **Internationalization**: 9 language support
6. **Analysis Panel**: Token usage tracking and thought process display
7. **Agentic Retrieval**: Knowledge base integration with web and SharePoint

### Features That May Conflict with Repo A

1. **Authentication**: MSAL implementation may differ
2. **Routing**: react-router-dom vs TanStack Router
3. **Styling**: Fluent UI theming vs Tailwind/shadcn approach
4. **State Management**: Local hooks vs Zustand
5. **Build Output**: Different static file serving strategy

## Testing Strategy

### Existing Test Coverage

The repository has comprehensive testing:
- **Unit Tests**: Backend components and utilities
- **Integration Tests**: API endpoints with mocked services
- **E2E Tests**: Playwright tests for UI flows

**Test Files:**
```
tests/
├── conftest.py              # Shared fixtures
├── test_app.py              # Main API tests
├── test_auth_*.py           # Authentication tests
├── test_approaches.py       # RAG approach tests
├── test_ingestion.py        # Document ingestion tests
├── e2e.py                   # End-to-end UI tests
└── ...
```

### Testing During Migration

**Recommended Test Strategy:**
1. Keep existing tests running against legacy implementation
2. Create parallel tests for migrated components
3. Use snapshot testing for UI component migration
4. Maintain API contract tests
5. Add integration tests for Repo A/B interactions

## Documentation Assets

The repository includes extensive documentation in `docs/`:

**Key Documentation Files:**
- `architecture.md` - System architecture
- `data_ingestion.md` - Document ingestion pipeline
- `localdev.md` - Local development setup
- `customization.md` - Customization guide
- `deploy_features.md` - Feature flags and configuration
- `login_and_acl.md` - Authentication and access control
- `monitoring.md` - Application monitoring
- `productionizing.md` - Production deployment guide

These docs should be:
1. Reviewed for Repo A integration
2. Updated with monorepo paths
3. Merged with or linked from Repo A docs

## Migration Risks & Mitigations

### High Risk Areas

1. **Authentication Flow Changes**
   - Risk: MSAL implementation differs between repos
   - Mitigation: Document differences, plan unified auth strategy

2. **State Management Migration**
   - Risk: Moving from local hooks to Zustand may break features
   - Mitigation: Incremental migration, maintain feature parity testing

3. **Routing Changes**
   - Risk: react-router-dom → TanStack Router breaks navigation
   - Mitigation: Create routing adapter layer initially

4. **Backend Framework Differences**
   - Risk: Quart vs FastAPI may have different async behavior
   - Mitigation: Comprehensive API contract tests

### Medium Risk Areas

1. **Fluent UI → shadcn/ui Component Mapping**
   - Risk: Not all Fluent components have direct shadcn equivalents
   - Mitigation: Create custom components where needed, document mapping

2. **Build Pipeline Integration**
   - Risk: Vite configs may conflict
   - Mitigation: Align configs early, test builds frequently

3. **Internationalization**
   - Risk: i18next setup may conflict with Repo A's approach
   - Mitigation: Standardize on one i18n approach across monorepo

## Success Metrics

**Migration is successful when:**

✅ All frontend components are shadcn/ui based (zero Fluent UI imports)  
✅ Application builds successfully in monorepo context  
✅ All existing features work identically  
✅ All tests pass (with updates for new component library)  
✅ Development workflow matches Repo A standards  
✅ Documentation is updated and accurate  
✅ Performance is equal or better than original  
✅ No Fluent UI packages in `node_modules`  

## Next Steps

1. **Review this document** with migration team and stakeholders
2. **Create Fluent → shadcn mapping document** (see separate doc)
3. **Set up branch in Repo A** for integration work
4. **Begin Phase 1**: Import code into monorepo structure
5. **Establish testing baseline** before making changes
6. **Start iterative migration** following phases above

## Appendix: Quick Reference

### Clone & Run Locally

```bash
# Clone
git clone https://github.com/zachpetersen/azure-search-openai-demo.git
cd azure-search-openai-demo

# Frontend
cd app/frontend
npm install
npm run dev  # http://localhost:5173

# Backend (separate terminal)
cd app/backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m quart --app app:app run --port 50505 --reload
```

### Key Commands

```bash
# Run tests
pytest

# Check types (mypy)
cd app/backend && mypy . --config-file=../../pyproject.toml

# Format code
cd app/frontend && npm run format

# Deploy to Azure
azd up
```

---

**Document Version**: 1.0  
**Last Updated**: 2025-11-24  
**Prepared for**: px.ai.ops.dash Integration (Repo A)
