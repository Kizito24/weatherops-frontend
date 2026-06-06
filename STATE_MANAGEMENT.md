# WeatherOps Frontend - State Management & State Charts

## Global Application State Structure

```typescript
// Root App Component State
{
  // Authentication State
  user: UserProfile | null
  
  // Data State
  locations: Location[]
  rules: Rule[]
  alerts: Alert[]
  
  // UI/Navigation State
  activePage: string // 'login' | 'register' | 'overview' | 'weather' | 'locations' | 'rules' | 'alerts' | 'trees' | 'settings'
  isSidebarCollapsed: boolean
  darkMode: boolean
  isLoading: boolean
  isSimulatingTrigger: boolean
  isExportModalOpen: boolean
  
  // Toast Notifications
  toasts: Toast[]
}
```

## State Management Flow Diagram

```
┌────────────────────────────────────────────────────────┐
│          COMPONENT MOUNT & INITIALIZATION               │
└──────────────────┬─────────────────────────────────────┘
                   │
                   ▼
      ┌────────────────────────────┐
      │  Check Authentication      │
      │  (tokenManager.isAuthenticated)
      └──────────┬─────────┬───────┘
                 │         │
         ┌───────┘         └────────┐
         │                          │
    Not Auth                     Auth
         │                          │
         ▼                          ▼
    ┌─────────┐            ┌──────────────────┐
    │ Login   │            │ Create User      │
    │ Page    │            │ Profile Object   │
    └─────────┘            └────────┬─────────┘
                                    │
                                    ▼
                           ┌──────────────────┐
                           │ Load Platform    │
                           │ Data in Parallel:│
                           │ - Locations      │
                           │ - Alerts         │
                           │ - Rules          │
                           └────────┬─────────┘
                                    │
                                    ▼
                           ┌──────────────────┐
                           │ Render Dashboard │
                           │ - Default Page:  │
                           │   Overview       │
                           └──────────────────┘
```

## Authentication State Chart

```
┌─────────────────────────────────────────────────────────┐
│                  AUTHENTICATION STATE CHART              │
└─────────────────────────────────────────────────────────┘

    ┌─────────────┐
    │  LOGOUT     │
    │  STATE      │ ◄────────────────────────────────┐
    └──────┬──────┘                                  │
           │                                         │
    [User navigates to login]                       │
           │                                         │
           ▼                                         │
    ┌──────────────────┐                            │
    │  LOGIN PAGE      │                            │
    │  DISPLAYED       │                            │
    └──────┬───────────┘                            │
           │                                         │
    [User enters credentials]                       │
           │                                         │
           ▼                                         │
    ┌──────────────────┐                            │
    │  VALIDATING      │                            │
    │  CREDENTIALS     │                            │
    └──────┬───────────┘                            │
           │                                         │
           ├─ Invalid ────┐                         │
           │              │                         │
           │              ▼                         │
           │        ┌────────────────┐              │
           │        │ Show Error     │              │
           │        │ Message        │              │
           │        └────────┬───────┘              │
           │                 │                      │
           │                 └─ Retry ─────┐        │
           │                               │        │
           │                               └──────┬─┘
           │                                      │
           ├─ Valid                               │
           │       │                              │
           │       ▼                              │
           │  ┌──────────────────┐               │
           │  │ STORING TOKEN    │               │
           │  │ IN LOCAL STORAGE │               │
           │  └────────┬─────────┘               │
           │           │                         │
           │           ▼                         │
           │  ┌──────────────────┐               │
           │  │  AUTHENTICATED   │               │
           │  │  STATE           │               │
           │  └────────┬─────────┘               │
           │           │                         │
           │           ├─ Token Valid ─────┐     │
           │           │                   │     │
           │           │                   ▼     │
           │           │            ┌────────────┐
           │           │            │  NAVIGATE  │
           │           │            │  TO APP    │
           │           │            └────────────┘
           │           │
           │           ├─ Token Expired ──┐
           │           │                  │
           │           │                  ▼
           │           │          ┌────────────────┐
           │           │          │ CLEAR TOKEN    │
           │           │          │ RETURN TO      │
           │           │          │ LOGOUT STATE   │
           │           │          └────────────────┘
           │           │
           │           └─ Logout Clicked ─┐
           │                               │
           └───────────────────────────────┘
```

## Page/Route State Chart

```
┌─────────────────────────────────────────────────────────┐
│                   PAGE NAVIGATION STATE                  │
└─────────────────────────────────────────────────────────┘

       ┌──────────────┐
       │   OVERVIEW   │ ◄────────────────────┐
       │   (Default)  │                      │
       └──────┬───────┘                      │
              │                              │
     [Click link / Hash change]              │
              │                              │
              ├─────────────┬─────────────┬─────────────┬─────────────┐
              │             │             │             │             │
              ▼             ▼             ▼             ▼             ▼
         ┌────────┐    ┌───────┐    ┌─────────┐  ┌──────────┐  ┌──────────┐
         │WEATHER │    │LOCS   │    │RULES    │  │ ALERTS   │  │SETTINGS  │
         │        │    │       │    │         │  │          │  │          │
         └────────┘    └───────┘    └─────────┘  └──────────┘  └──────────┘
              │             │             │             │             │
              └─────────────┴─────────────┴─────────────┴─────────────┘
                                  │
                     [Hash changes back to Overview]
                                  │
                                  ▼
                         ┌──────────────┐
                         │   OVERVIEW   │
                         └──────────────┘

Active Page Values:
- 'login' - Authentication page
- 'register' - Registration page
- 'overview' - Main dashboard
- 'weather' - Weather data page
- 'locations' - Locations management
- 'rules' - Rules management
- 'alerts' - Alert history
- 'trees' - Tree analysis
- 'settings' - User settings
```

## Data Loading State Chart

```
┌─────────────────────────────────────────────────────────┐
│            DATA LOADING & SYNC STATE CHART               │
└─────────────────────────────────────────────────────────┘

Start of Session
       │
       ▼
┌────────────────────┐
│  INITIAL STATE     │
│  isLoading: false  │
│  data: []          │
└────────┬───────────┘
         │
  [User authenticates]
         │
         ▼
┌────────────────────────────┐
│  LOADING STATE             │
│  isLoading: true           │
│  Fetch in parallel:        │
│  1. Locations              │
│  2. Alerts                 │
│  3. Rules (per location)   │
└────────┬───────────────────┘
         │
         ├─ All Success
         │      │
         │      ▼
         │  ┌──────────────────┐
         │  │ LOADED STATE     │
         │  │ isLoading: false │
         │  │ data: [...]      │
         │  └────────┬─────────┘
         │           │
         │           ├─ [Manual Refresh]
         │           │       │
         │           │       └─────┐
         │           │             │
         │           └─ [Action: Create/Update/Delete]
         │                         │
         │           ┌─────────────┘
         │           │
         │           ▼
         │     ┌────────────────┐
         │     │ RELOADING      │
         │     │ (Call loadPlatformData)
         │     └────────┬───────┘
         │              │
         │              └─ [Recursively go through cycle]
         │
         ├─ Some Failed
         │      │
         │      ▼
         │  ┌──────────────────┐
         │  │ PARTIAL LOAD     │
         │  │ Show available   │
         │  │ data + Error     │
         │  │ Toast            │
         │  └────────┬─────────┘
         │           │
         │           └─ [User clicks Refresh]
         │                   │
         │                   └─────────┐
         │                             │
         └─────────────────────────────┘
```

## UI State Management Chart

```
┌─────────────────────────────────────────────────────┐
│         UI INTERACTION STATE MANAGEMENT              │
└─────────────────────────────────────────────────────┘

SIDEBAR STATE
    ┌──────────────────┐
    │ EXPANDED         │ ◄──────────┐
    │ (Normal View)    │            │
    └────────┬─────────┘            │
             │                      │
      [Click Collapse]             │
             │                      │
             ▼                      │
    ┌──────────────────┐            │
    │ COLLAPSED        │            │
    │ (Icon Only)      │            │
    └────────┬─────────┘            │
             │                      │
      [Screen > 1024px OR         │
       Click Expand]               │
             │                      │
             └──────────────────────┘

THEME STATE
    ┌──────────────────┐
    │ DARK MODE        │ ◄──────────┐
    │ (Default)        │            │
    └────────┬─────────┘            │
             │                      │
      [Click Toggle]               │
             │                      │
             ▼                      │
    ┌──────────────────┐            │
    │ LIGHT MODE       │            │
    └────────┬─────────┘            │
             │                      │
      [Click Toggle]               │
             │                      │
             └──────────────────────┘
  (Persisted in localStorage)

MODAL STATE
    ┌──────────────────┐
    │ MODAL CLOSED     │ ◄──────────┐
    └────────┬─────────┘            │
             │                      │
      [Click Export Button]        │
             │                      │
             ▼                      │
    ┌──────────────────┐            │
    │ MODAL OPEN       │            │
    │ (Show Export UI) │            │
    └────────┬─────────┘            │
             │                      │
      [Cancel or Complete]         │
             │                      │
             └──────────────────────┘
```

## Toast Notification State System

```
┌─────────────────────────────────────────────────────────┐
│           TOAST NOTIFICATION STATE SYSTEM                │
└─────────────────────────────────────────────────────────┘

Toast Creation Flow:
    
    showToast(title, message, severity)
            │
            ▼
    ┌─────────────────────────┐
    │ Create Toast Object:    │
    │ {                       │
    │   id: 'toast-xxxxx',    │
    │   title,                │
    │   message,              │
    │   severity              │
    │ }                       │
    └────────┬────────────────┘
             │
             ▼
    ┌─────────────────────────┐
    │ Add to toasts array     │
    │ (Triggers re-render)    │
    └────────┬────────────────┘
             │
             ▼
    ┌─────────────────────────┐
    │ Toast Rendered on UI    │
    │                         │
    │ Severity Styling:       │
    │ • HIGH (Red)            │
    │ • MEDIUM (Orange)       │
    │ • LOW (Green)           │
    │ • INFO (Blue)           │
    └────────┬────────────────┘
             │
    ┌────────┴──────────┐
    │                   │
    ▼                   ▼
[Auto-dismiss]     [Manual close]
  5 seconds           [Click X]
    │                   │
    └────────┬──────────┘
             │
             ▼
    ┌─────────────────────────┐
    │ Remove from array       │
    │ removeToast(id)         │
    └─────────────────────────┘

Toast Queue Example:
    
    toasts = [
      {id: 'toast-1', title: 'Welcome', severity: 'INFO'},
      {id: 'toast-2', title: 'Location Added', severity: 'LOW'},
      {id: 'toast-3', title: 'Error Occurred', severity: 'HIGH'}
    ]
    
    Displayed as stack:
    ┌─────────────────────┐
    │ ⚠ Error Occurred    │
    │ Error message...    │
    └─────────────────────┘
    ┌─────────────────────┐
    │ ✓ Location Added    │
    │ Success message...  │
    └─────────────────────┘
    ┌─────────────────────┐
    │ ℹ Welcome           │
    │ Welcome message...  │
    └─────────────────────┘
```

## Location Management State Flow

```
┌─────────────────────────────────────────────────────────┐
│        LOCATION CRUD OPERATIONS STATE FLOW               │
└─────────────────────────────────────────────────────────┘

                    LOCATIONS LIST
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    ┌────────┐      ┌────────┐      ┌────────┐
    │ CREATE │      │ READ   │      │ UPDATE │
    └───┬────┘      └───┬────┘      └───┬────┘
        │                │                │
        ▼                ▼                ▼
    
CREATE FLOW:
    [User fills form]
         │
         ▼
    ┌─────────────────────┐
    │ Validate Input      │
    │ (name, lat, lng)    │
    └────┬────────────────┘
         │
         ├─ Invalid ──────┐
         │        │       │
         │        ▼       │
         │   Show Error   │
         │   Toast        │
         │        │       │
         │        └─ Retry
         │
         ├─ Valid
         │     │
         │     ▼
         │  ┌──────────────────┐
         │  │ Call API         │
         │  │ locationsApi     │
         │  │ .create()        │
         │  └────┬─────────────┘
         │       │
         │       ├─ Success
         │       │     │
         │       │     ▼
         │       │  ┌──────────────────┐
         │       │  │ Show Success     │
         │       │  │ Toast            │
         │       │  └────┬─────────────┘
         │       │       │
         │       │       ▼
         │       │  ┌──────────────────┐
         │       │  │ Reload All Data  │
         │       │  │ (Locations,      │
         │       │  │  Rules, Alerts)  │
         │       │  └────┬─────────────┘
         │       │       │
         │       │       ▼
         │       │  ┌──────────────────┐
         │       │  │ Update State     │
         │       │  │ locations = [...] 
         │       │  └────┬─────────────┘
         │       │       │
         │       │       ▼
         │       │  ┌──────────────────┐
         │       │  │ Close Form       │
         │       │  │ (UI updates)     │
         │       │  └──────────────────┘
         │       │
         │       └─ Error
         │             │
         │             ▼
         │       ┌──────────────────┐
         │       │ Show Error       │
         │       │ Toast            │
         │       └──────────────────┘
         │
         └─────────────────────┘

UPDATE & DELETE FLOWS are similar with minor differences.
```

## Rules Management State Flow

```
┌─────────────────────────────────────────────────────────┐
│         RULES CRUD & TOGGLE STATE FLOW                   │
└─────────────────────────────────────────────────────────┘

                    RULES LIST
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
    ▼                    ▼                    ▼
┌────────┐          ┌────────┐          ┌────────┐
│ CREATE │          │ TOGGLE │          │ DELETE │
└───┬────┘          └───┬────┘          └───┬────┘
    │                   │                    │
    ▼                   ▼                    ▼

CREATE RULE:
    [Select Location]
         │
         ▼
    [Select Metric]
    (temperature, rainfall, wind_speed, humidity)
         │
         ▼
    [Select Operator]
    (>, <, >=, <=, ==)
         │
         ▼
    [Enter Threshold Value]
         │
         ▼
    [Click Create]
         │
         ▼
    ┌──────────────────────┐
    │ Validate All Fields  │
    └────┬─────────────────┘
         │
         ├─ Invalid → Show Error
         │
         ├─ Valid
         │     │
         │     ▼
         │  ┌──────────────────┐
         │  │ API Call         │
         │  │ rulesApi.create()│
         │  └────┬─────────────┘
         │       │
         │       ├─ Success
         │       │     │
         │       │     ▼
         │       │  ┌────────────────────┐
         │       │  │ Show Success Toast │
         │       │  │ "Alert Rule Armed" │
         │       │  └────┬───────────────┘
         │       │       │
         │       │       ▼
         │       │  ┌────────────────────┐
         │       │  │ Reload Platform    │
         │       │  │ Data               │
         │       │  └────┬───────────────┘
         │       │       │
         │       │       ▼
         │       │  ┌────────────────────┐
         │       │  │ Update rules State │
         │       │  └────────────────────┘
         │       │
         │       └─ Error → Show Error Toast
         │
         └──────────────────────┘

TOGGLE RULE:
    [User clicks Active/Inactive toggle]
         │
         ▼
    ┌──────────────────────┐
    │ Optimistic Update    │
    │ (Update UI first)    │
    └────┬─────────────────┘
         │
         ▼
    ┌──────────────────────┐
    │ API Call             │
    │ rulesApi.toggleActive
    └────┬─────────────────┘
         │
         ├─ Success → Confirm in Toast
         │
         ├─ Error
         │     │
         │     ▼
         │  ┌────────────────────┐
         │  │ Revert UI          │
         │  │ Show Error Toast   │
         │  └────────────────────┘
         │
         └──────────────────────┘
```

## Alerts Management State Flow

```
┌─────────────────────────────────────────────────────────┐
│           ALERTS MANAGEMENT STATE FLOW                   │
└─────────────────────────────────────────────────────────┘

                    ALERTS LIST
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    ┌────────┐      ┌────────┐      ┌──────────┐
    │ RESOLVE│      │ FILTER │      │ EXPORT   │
    │ SINGLE │      │ ALERTS │      │ DATA     │
    └───┬────┘      └───┬────┘      └──────┬───┘
        │                │                 │
        ▼                ▼                 ▼

RESOLVE SINGLE ALERT:
    [User clicks Resolve button on alert card]
         │
         ▼
    ┌──────────────────────┐
    │ Show Confirmation    │
    │ (Optional Toast)     │
    └────┬─────────────────┘
         │
         ▼
    ┌──────────────────────┐
    │ API Call             │
    │ alertsApi.resolve()  │
    └────┬─────────────────┘
         │
         ├─ Success
         │     │
         │     ▼
         │  ┌──────────────────┐
         │  │ Show Success     │
         │  │ Toast            │
         │  │ "Incident        │
         │  │ Acknowledged"    │
         │  └────┬─────────────┘
         │       │
         │       ▼
         │  ┌──────────────────┐
         │  │ Reload Alerts    │
         │  │ loadPlatformData()
         │  └────┬─────────────┘
         │       │
         │       ▼
         │  ┌──────────────────┐
         │  │ Remove from UI   │
         │  │ alerts = [...]   │
         │  └──────────────────┘
         │
         ├─ Error
         │     │
         │     ▼
         │  ┌──────────────────┐
         │  │ Show Error Toast │
         │  └──────────────────┘
         │
         └──────────────────────┘

FILTER ALERTS:
    [User selects severity filter: HIGH, MEDIUM, LOW]
         │
         ▼
    [Local filtering of alerts array]
         │
         ▼
    [Re-render AlertsList with filtered data]

EXPORT DATA:
    [User clicks Export button]
         │
         ▼
    ┌──────────────────────┐
    │ Show Export Modal    │
    └────┬─────────────────┘
         │
    [User selects:]
    • Format (CSV/JSON)
    • Data type (Alerts, Rules, Locations)
         │
         ▼
    ┌──────────────────────┐
    │ Format Data          │
    │ (CSV or JSON)        │
    └────┬─────────────────┘
         │
         ▼
    ┌──────────────────────┐
    │ Create Download      │
    │ (Client-side)        │
    └────┬─────────────────┘
         │
         ▼
    ┌──────────────────────┐
    │ Trigger File         │
    │ Download             │
    └────┬─────────────────┘
         │
         ▼
    ┌──────────────────────┐
    │ Show Success Toast   │
    │ File Downloaded      │
    └──────────────────────┘
```

## Component Lifecycle State Management

```
┌─────────────────────────────────────────────────────────┐
│     REACT COMPONENT LIFECYCLE & STATE PATTERNS            │
└─────────────────────────────────────────────────────────┘

App Component (Root):
    
    useEffect Hook 1: Theme Management
    ┌──────────────────────────────────┐
    │ When darkMode changes:           │
    │ 1. Add/Remove 'dark' class       │
    │ 2. Update localStorage           │
    │ Dependencies: [darkMode]          │
    └──────────────────────────────────┘
    
    useEffect Hook 2: Auth & Routing Sync
    ┌──────────────────────────────────┐
    │ On Mount ([]): Check token &     │
    │ set initial page                 │
    │ Setup event listener for         │
    │ hash changes                     │
    │ Dependencies: [user]              │
    └──────────────────────────────────┘
    
    useEffect Hook 3: Responsive Sidebar
    ┌──────────────────────────────────┐
    │ On Mount & Resize event:         │
    │ Set isSidebarCollapsed based on  │
    │ window width < 1024px            │
    │ Dependencies: []                  │
    └──────────────────────────────────┘
    
    useEffect Hook 4: Platform Data Loading
    ┌──────────────────────────────────┐
    │ When user logs in:               │
    │ Call loadPlatformData()          │
    │ Fetch locations, alerts, rules   │
    │ Dependencies: [user]              │
    └──────────────────────────────────┘

LocationsPage Component:
    
    When Component Mounts:
    ┌──────────────────────────────────┐
    │ 1. Receive locations prop        │
    │ 2. Initialize pagination state   │
    │ 3. Setup form state              │
    └──────────────────────────────────┘
    
    When User Adds Location:
    ┌──────────────────────────────────┐
    │ 1. Validate form inputs          │
    │ 2. Call handleAddLocation()      │
    │ 3. Show loading state            │
    │ 4. On success: refresh data &    │
    │    close form                    │
    │ 5. On error: show error message  │
    └──────────────────────────────────┘
```

---

**Key Principles:**

1. **Unidirectional Data Flow**: Data flows down via props, events bubble up
2. **Single Source of Truth**: All global state lives in App component
3. **Derived State**: Page selection from URL hash, not stored separately
4. **Optimistic Updates**: UI updates before API response for better UX
5. **Error Boundaries**: Graceful error handling with user-facing messages
6. **Auto-cleanup**: Effects cleanup subscriptions and event listeners
7. **localStorage Persistence**: Theme and token persist across sessions

**Last Updated**: June 2026  
**Version**: 1.0.0
