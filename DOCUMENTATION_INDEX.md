# WeatherOps Frontend - Documentation Index

Welcome to the WeatherOps Frontend Documentation. This index helps you navigate all available documentation.

## 📚 Documentation Files

### 1. **ARCHITECTURE.md** - System Design & High-Level Overview
**Best for**: Understanding the overall system structure, infrastructure, and major components

**Contains**:
- System architecture diagram
- Data flow architecture
- Component hierarchy tree
- Authentication flow diagram
- Storage strategy
- Error handling & recovery
- Key features & workflows

**Quick Links**:
- System overview visualization
- Component interaction patterns
- Data persistence strategy
- 5-step authentication process

---

### 2. **STATE_MANAGEMENT.md** - Application State & State Charts
**Best for**: Understanding how state flows through the application and component lifecycle

**Contains**:
- Global state structure (TypeScript interface)
- App initialization flow
- Authentication state machine
- Page/Route navigation state chart
- Data loading & sync workflow
- UI state management (theme, sidebar, modals)
- Toast notification system
- CRUD operation flows
- Component lifecycle patterns

**Key Diagrams**:
- Authentication state chart (4 states)
- Page navigation routing (8 pages)
- Data loading state machine
- Toast queue system
- Location, Rules, and Alerts CRUD flows

**State Variables Tracked**:
```typescript
user: UserProfile | null
locations: Location[]
rules: Rule[]
alerts: Alert[]
activePage: string
isSidebarCollapsed: boolean
darkMode: boolean
isLoading: boolean
toasts: Toast[]
```

---

### 3. **COMPONENTS.md** - Component Documentation & Interactions
**Best for**: Finding detailed information about specific components, props, and usage patterns

**Contains**:
- Component architecture overview
- Page components (8 pages with full documentation)
- Shared/utility components
- Form components (FormInput, FormSelect)
- Display components (KPICard, Badge, EmptyState)
- Modal & notification components
- Component communication patterns
- Data flow examples

**Page Components Documented**:
1. AuthPage - Login/Registration
2. OverviewPage - Dashboard
3. WeatherPage - Weather data
4. LocationsPage - Location management
5. RulesPage - Rule management
6. AlertsPage - Alert management
7. TreesPage - Tree analysis
8. SettingsPage - User settings

**Component Communication Patterns**:
- Props flow (parent to child)
- Event callbacks (child to parent)
- Data update cycles

---

### 4. **API_INTEGRATION.md** - API Layer & Data Integration
**Best for**: Understanding API calls, request-response cycles, and error handling

**Contains**:
- API layer architecture
- Complete API client structure
- Authentication flow with token management
- Location CRUD endpoints with sequence diagrams
- Rules API endpoints and workflows
- Alerts API endpoints and resolution flow
- Weather API data fetching flow
- Error handling strategies (10 error types)
- Request retry logic with exponential backoff
- Frontend data validation rules

**API Clients Documented**:
- authApi (login, register, logout)
- locationsApi (CRUD)
- rulesApi (create, toggle, delete)
- alertsApi (list, resolve)
- weatherApi (current, forecast, usage)
- treeApi (analyze, usage)

**Error Types Covered**:
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 409 Conflict
- 429 Rate Limited
- 500+ Server Errors
- Network Errors

---

## 🗺️ How to Use This Documentation

### Scenario 1: I'm adding a new feature
1. Check **COMPONENTS.md** for similar component patterns
2. Review **STATE_MANAGEMENT.md** for state management approach
3. Check **API_INTEGRATION.md** if your feature needs API calls
4. Reference **ARCHITECTURE.md** for system-level integration points

### Scenario 2: I'm debugging a state issue
1. Go to **STATE_MANAGEMENT.md** and find the relevant state machine
2. Check **COMPONENTS.md** for component lifecycle patterns
3. Trace data flow in **ARCHITECTURE.md**

### Scenario 3: I'm implementing API integration
1. Start with **API_INTEGRATION.md** - API client structure
2. Check request-response cycle diagrams
3. Review error handling strategies
4. Look at **STATE_MANAGEMENT.md** for how to handle responses

### Scenario 4: I'm optimizing component rendering
1. Review **COMPONENTS.md** - component hierarchy
2. Check **STATE_MANAGEMENT.md** - useEffect patterns
3. Verify prop flow in **ARCHITECTURE.md**

---

## 🔄 Quick Reference: Data Flows

### User Authentication Flow
```
Login Page → Validate → API Call → Store Token → Load Data → Dashboard
```
See: **ARCHITECTURE.md** (Authentication Flow) & **API_INTEGRATION.md** (Auth Endpoints)

### Create Location Flow
```
Form Input → Validate → API Call → Reload Data → Update State → Render
```
See: **STATE_MANAGEMENT.md** (Location CRUD) & **API_INTEGRATION.md** (Location Endpoints)

### Display Alert Flow
```
Alert Created (Backend) → API Fetch → Add to State → Filter/Sort → Render
```
See: **STATE_MANAGEMENT.md** (Alerts Flow) & **API_INTEGRATION.md** (Alert Endpoints)

### Toggle Rule Flow
```
User Click → Optimistic Update → API Call → Confirm/Revert → Reload Data
```
See: **STATE_MANAGEMENT.md** (Rules CRUD) & **API_INTEGRATION.md** (Rules Endpoints)

---

## 📊 Key Diagrams Index

| Diagram | Location | Purpose |
|---------|----------|---------|
| System Architecture | ARCHITECTURE.md | Overall system structure |
| Data Flow Architecture | ARCHITECTURE.md | End-to-end data movement |
| Component Hierarchy | ARCHITECTURE.md | Component tree structure |
| Authentication Flow | ARCHITECTURE.md | User login process |
| State Management Flow | STATE_MANAGEMENT.md | App initialization |
| Auth State Chart | STATE_MANAGEMENT.md | Authentication states |
| Page Navigation Chart | STATE_MANAGEMENT.md | Route management |
| Data Loading State | STATE_MANAGEMENT.md | Async data loading |
| UI State Management | STATE_MANAGEMENT.md | Theme, sidebar, modals |
| Toast System | STATE_MANAGEMENT.md | Notification queue |
| Location CRUD Flow | STATE_MANAGEMENT.md | Location operations |
| Rules CRUD Flow | STATE_MANAGEMENT.md | Rule operations |
| Alerts Flow | STATE_MANAGEMENT.md | Alert operations |
| Component Lifecycle | STATE_MANAGEMENT.md | React hooks patterns |
| API Layer Architecture | API_INTEGRATION.md | API client organization |
| Login Cycle | API_INTEGRATION.md | Authentication request-response |
| Token Management | API_INTEGRATION.md | JWT token lifecycle |
| Location Endpoints | API_INTEGRATION.md | Location CRUD diagram |
| Rule Creation | API_INTEGRATION.md | Rule workflow |
| Alert Resolution | API_INTEGRATION.md | Alert workflow |
| Weather Data Fetch | API_INTEGRATION.md | Weather request flow |
| Error Handling | API_INTEGRATION.md | Error recovery |

---

## 🔑 Key Concepts

### Authentication
- JWT token-based authentication
- Token stored in localStorage
- Auto-logout on token expiration (401 response)
- Protected routes require valid token

### State Management
- Global state in App component (React hooks)
- No Redux/Context API - pure useState/useEffect
- Data flows down via props
- Events bubble up via callbacks
- Derived state from URL hash

### API Integration
- Axios-based HTTP client
- Request/response interceptors
- Automatic retry on network errors
- Error classification and user messaging
- Parallel data loading with Promise.all

### Component Patterns
- Functional components with hooks
- Controlled components for forms
- Optimistic UI updates
- Skeleton loading states
- Error boundary error handling

### Data Validation
- Frontend validation on form submit
- Backend validation on API endpoints
- Type checking with TypeScript
- Validation error messages in toast

---

## 🎯 Common Tasks

### Adding a New Page
1. Create component file in `src/components/`
2. Add case in App.tsx router
3. Add navigation link in Sidebar
4. Add state if needed
5. Wire up API calls in useEffect
6. Handle loading/error states

### Adding an API Endpoint
1. Create API client in `src/lib/api/`
2. Create Axios request function
3. Add error handling
4. Add TypeScript types
5. Call from component handler
6. Update state on success
7. Show toast notification

### Modifying State Flow
1. Check current state in App.tsx
2. Review STATE_MANAGEMENT.md for impact
3. Update all dependent useEffect hooks
4. Verify data flow in ARCHITECTURE.md
5. Test state transitions

### Fixing a Bug
1. Identify which system is affected (State/API/Component)
2. Check relevant documentation file
3. Trace data flow with diagrams
4. Review error handling strategy
5. Check related tests

---

## 📋 File Organization

```
src/
├── main.tsx                    # App entry point
├── App.tsx                     # Root component
├── types.ts                    # TypeScript interfaces
├── components/
│   ├── AuthPage.tsx           # Login/Register page
│   ├── OverviewPage.tsx       # Dashboard
│   ├── WeatherPage.tsx        # Weather data
│   ├── LocationsPage.tsx      # Locations CRUD
│   ├── RulesPage.tsx          # Rules CRUD
│   ├── AlertsPage.tsx         # Alerts view
│   ├── TreesPage.tsx          # Tree analysis
│   ├── SettingsPage.tsx       # User settings
│   ├── Sidebar.tsx            # Navigation
│   ├── Navbar.tsx             # Top bar
│   ├── ExportModal.tsx        # Export dialog
│   ├── FormInput.tsx          # Form field
│   ├── FormSelect.tsx         # Select field
│   ├── KPICard.tsx            # Metric card
│   ├── Badge.tsx              # Status badge
│   ├── EmptyState.tsx         # Empty content
│   ├── Pagination.tsx         # Page control
│   ├── Skeletons.tsx          # Loading state
│   └── ErrorBoundary.tsx      # Error handling
├── lib/
│   ├── api/
│   │   ├── auth.ts            # Auth API client
│   │   ├── locations.ts       # Locations API
│   │   ├── rules.ts           # Rules API
│   │   ├── alerts.ts          # Alerts API
│   │   ├── weather.ts         # Weather API
│   │   └── tree.ts            # Tree API
│   ├── auth/
│   │   └── tokenManager.ts    # Token management
│   ├── export.ts              # Data export utility
│   └── validation.ts          # Form validation
└── assets/                    # Images & static files
```

---

## 🔗 Related Files

- `package.json` - Dependencies and scripts
- `vite.config.ts` - Build configuration
- `tsconfig.json` - TypeScript configuration
- `tailwind.config.js` - Style configuration
- `.env` - Environment variables
- `vercel.json` - Deployment configuration

---

## 💡 Tips for Developers

1. **Always check STATE_MANAGEMENT.md first** when understanding an existing feature
2. **Use the diagram search index** above to quickly find relevant diagrams
3. **Follow the component communication patterns** in COMPONENTS.md for consistency
4. **Review error handling** before implementing new features
5. **Keep state flat** - avoid deeply nested structures
6. **Use TypeScript interfaces** from types.ts for all API responses
7. **Add proper error handling** for all API calls
8. **Show loading states** during async operations
9. **Use toast notifications** for user feedback
10. **Test state transitions** manually before submitting PR

---

## 📞 Documentation Maintenance

- **Last Updated**: June 2026
- **Version**: 1.0.0
- **Maintained By**: Development Team

When making significant changes to:
- State structure → Update STATE_MANAGEMENT.md
- Component hierarchy → Update COMPONENTS.md
- API endpoints → Update API_INTEGRATION.md
- System design → Update ARCHITECTURE.md

---

**For questions or clarifications, refer to the specific documentation file with diagrams and detailed explanations.**
