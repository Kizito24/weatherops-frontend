# WeatherOps Frontend - System Architecture

## Overview

WeatherOps is a comprehensive weather monitoring and alert management system that enables users to create rules based on weather metrics across multiple locations and receive alerts when those rules are triggered.

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        USER BROWSER / CLIENT                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     REACT APPLICATION                        │   │
│  │                                                              │   │
│  │  ┌─────────────────────────────────────────────────────┐    │   │
│  │  │              APP ROOT COMPONENT                     │    │   │
│  │  │  - Global State Management                          │    │   │
│  │  │  - Route/Page Management                            │    │   │
│  │  │  - Authentication Gate                              │    │   │
│  │  └─────────────────────────────────────────────────────┘    │   │
│  │                           │                                   │   │
│  │    ┌──────────────────────┼──────────────────────┐           │   │
│  │    │                      │                      │           │   │
│  │    ▼                      ▼                      ▼           │   │
│  │  ┌──────────┐      ┌─────────────┐      ┌──────────────┐   │   │
│  │  │ Auth     │      │  Sidebar    │      │   Navbar     │   │   │
│  │  │ Page     │      │ Navigation  │      │   Top Bar    │   │   │
│  │  └──────────┘      └─────────────┘      └──────────────┘   │   │
│  │                                                              │   │
│  │          ┌──────────────────────────────────┐              │   │
│  │          │       PAGE ROUTER OUTLET         │              │   │
│  │          │  - Overview                      │              │   │
│  │          │  - Weather                       │              │   │
│  │          │  - Locations                     │              │   │
│  │          │  - Rules                         │              │   │
│  │          │  - Alerts                        │              │   │
│  │          │  - Trees                         │              │   │
│  │          │  - Settings                      │              │   │
│  │          └──────────────────────────────────┘              │   │
│  │                                                              │   │
│  │  ┌────────────────────────────────────────────────────┐    │   │
│  │  │              TOAST NOTIFICATION SYSTEM             │    │   │
│  │  │  - INFO / LOW / MEDIUM / HIGH severity            │    │   │
│  │  │  - Auto-dismiss after 5 seconds                   │    │   │
│  │  └────────────────────────────────────────────────────┘    │   │
│  │                                                              │   │
│  │  ┌────────────────────────────────────────────────────┐    │   │
│  │  │            MODAL SYSTEM                            │    │   │
│  │  │  - Export Modal                                    │    │   │
│  │  │  - Confirmation Dialogs                           │    │   │
│  │  └────────────────────────────────────────────────────┘    │   │
│  │                                                              │   │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              STATE MANAGEMENT & HOOKS                        │   │
│  │                                                              │   │
│  │  • User Authentication State                                │   │
│  │  • Locations State                                          │   │
│  │  • Rules State                                              │   │
│  │  • Alerts State                                             │   │
│  │  • UI State (Dark Mode, Sidebar Collapse)                  │   │
│  │  • Loading States                                           │   │
│  │                                                              │   │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              API INTEGRATION LAYER                           │   │
│  │                                                              │   │
│  │  • authApi - Authentication & User Management              │   │
│  │  • locationsApi - CRUD Operations for Locations            │   │
│  │  • rulesApi - Create/Update/Delete Rules                   │   │
│  │  • alertsApi - Manage Weather Alerts                       │   │
│  │  • weatherApi - Fetch Current & Forecast Data              │   │
│  │  • treeApi - Tree Health Analysis                          │   │
│  │                                                              │   │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              UTILITY & HELPER MODULES                        │   │
│  │                                                              │   │
│  │  • tokenManager - JWT Token Management                      │   │
│  │  • validation - Form & Data Validation                      │   │
│  │  • export - Data Export (CSV, JSON)                         │   │
│  │                                                              │   │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTP/HTTPS
                              │ REST API Calls
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    BACKEND WEATHER OPS API SERVER                   │
│                                                                      │
│  • Authentication Endpoints                                        │
│  • Location Management Endpoints                                   │
│  • Weather Rules Endpoints                                         │
│  • Alert Management Endpoints                                      │
│  • Weather Data Endpoints                                          │
│  • Tree Analysis Endpoints                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   EXTERNAL SERVICES & DATABASES                     │
│                                                                      │
│  • Weather Data Provider (OpenWeather API / Similar)               │
│  • Geolocation Services                                            │
│  • Google Generative AI (for AI summaries)                         │
│  • User Database                                                    │
│  • Alert History Database                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Data Flow Architecture

```
┌─────────────┐
│   User      │
│ Interacts   │
└──────┬──────┘
       │
       ▼
┌──────────────────────┐
│  Browser Event       │
│  - Click             │
│  - Form Submit       │
│  - Route Change      │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│  Event Handler       │
│  (App Component)     │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│  Validation & Error Handling         │
│  - Input Validation                  │
│  - Error Boundary Catch              │
└──────┬───────────────────────────────┘
       │
       ├─────────────────────────────────┐
       │                                 │
       ▼                                 ▼
┌──────────────────────┐     ┌──────────────────────┐
│  Update Local State  │     │  API Call            │
│  (Optimistic)        │     │  (Send to Backend)   │
└──────┬───────────────┘     └──────┬───────────────┘
       │                            │
       │                            ▼
       │                    ┌──────────────────────┐
       │                    │  Backend Processing  │
       │                    │  - Database Query    │
       │                    │  - Business Logic    │
       │                    │  - Data Validation   │
       │                    └──────┬───────────────┘
       │                           │
       │                           ▼
       │                    ┌──────────────────────┐
       │                    │  API Response        │
       │                    │  - Data              │
       │                    │  - Status Code       │
       │                    │  - Errors            │
       │                    └──────┬───────────────┘
       │                           │
       └───────────────┬───────────┘
                       │
                       ▼
            ┌──────────────────────────┐
            │  Handle API Response     │
            │  - Update State          │
            │  - Show Toast Notification
            │  - Handle Errors         │
            └──────┬───────────────────┘
                   │
                   ▼
            ┌──────────────────────────┐
            │  Refresh Related Data    │
            │  - Reload Locations      │
            │  - Reload Rules          │
            │  - Reload Alerts         │
            └──────┬───────────────────┘
                   │
                   ▼
            ┌──────────────────────────┐
            │  Component Re-render     │
            │  (Via State Update)       │
            └──────┬───────────────────┘
                   │
                   ▼
            ┌──────────────────────────┐
            │  User Sees Changes       │
            └──────────────────────────┘
```

## Component Hierarchy

```
App
├── AuthPage
│   ├── LoginForm
│   │   ├── FormInput
│   │   └── FormSelect
│   └── RegisterForm
│       ├── FormInput
│       └── FormSelect
│
├── Sidebar
│   ├── Navigation Links
│   ├── User Profile Section
│   └── Logout Button
│
├── Navbar
│   ├── Page Title
│   ├── Theme Toggle (Dark/Light)
│   ├── Simulate Incident Button
│   └── Export Button
│
├── OverviewPage
│   ├── KPICard
│   │   ├── Metric Value
│   │   └── Trend Indicator
│   ├── AlertsSummary
│   └── QuickStats
│
├── WeatherPage
│   ├── LocationSelector
│   ├── CurrentWeather
│   │   └── WeatherMetrics
│   ├── ForecastSection
│   │   └── ForecastCard
│   └── WeatherUsage
│
├── LocationsPage
│   ├── LocationForm
│   │   ├── FormInput (Name, Lat, Lng)
│   │   └── Submit Button
│   ├── LocationsList
│   │   ├── LocationCard
│   │   │   ├── Edit Button
│   │   │   └── Delete Button
│   │   └── Pagination
│   └── EmptyState (when no locations)
│
├── RulesPage
│   ├── RuleForm
│   │   ├── LocationSelect
│   │   ├── MetricSelect
│   │   ├── OperatorSelect
│   │   ├── ThresholdInput
│   │   └── CreateButton
│   ├── RulesList
│   │   ├── RuleCard
│   │   │   ├── Rule Details
│   │   │   ├── Active/Inactive Toggle
│   │   │   └── Delete Button
│   │   └── Pagination
│   └── EmptyState (when no rules)
│
├── AlertsPage
│   ├── AlertsFilter
│   ├── AlertsList
│   │   ├── AlertCard
│   │   │   ├── Location Name
│   │   │   ├── Metric & Value
│   │   │   ├── Severity Badge
│   │   │   └── Resolve Button
│   │   └── Pagination
│   ├── ClearAllButton
│   └── EmptyState (when no alerts)
│
├── TreesPage
│   ├── TreeUpload
│   └── TreeAnalysisResults
│
├── SettingsPage
│   ├── UserProfile Section
│   │   ├── Name Input
│   │   ├── Email Display
│   │   └── Role Display
│   └── Notification Preferences
│       ├── Email Toggle
│       ├── SMS Toggle
│       └── System Toggle
│
├── ExportModal
│   ├── Export Format Selector
│   ├── Data Selection Checkboxes
│   └── Export Button
│
└── Toast System
    └── ToastContainer
        ├── Toast (INFO)
        ├── Toast (LOW)
        ├── Toast (MEDIUM)
        └── Toast (HIGH)
```

## Authentication Flow

```
START
  │
  ▼
┌──────────────────────┐
│  Check Token in      │
│  Local Storage       │
└──────┬───────────────┘
       │
       ├─ Token Found ──────┐
       │                    │
       ├─ Token Not Found   │
       │       │            │
       │       ▼            │
       │  ┌─────────────┐   │
       │  │  Redirect   │   │
       │  │  to Login   │   │
       │  └─────────────┘   │
       │                    │
       ▼                    │
┌──────────────────────┐   │
│  Validate Token      │   │
│  with Backend        │   │
└──────┬───────────────┘   │
       │                    │
       ├─ Valid ────────────┤
       │                    │
       ├─ Invalid/Expired   │
       │      │             │
       │      ▼             │
       │  ┌─────────────┐   │
       │  │  Clear      │   │
       │  │  Token &    │   │
       │  │  Redirect   │   │
       │  │  to Login   │   │
       │  └─────────────┘   │
       │                    │
       ▼                    │
┌──────────────────────┐   │
│  User Submits        │   │
│  Login Credentials   │   │
└──────┬───────────────┘   │
       │                    │
       ▼                    │
┌──────────────────────┐   │
│  Validate Input      │   │
│  (Email & Password)  │   │
└──────┬───────────────┘   │
       │                    │
       ├─ Invalid ──┐       │
       │      │     │       │
       │      ▼     │       │
       │  Show Error│       │
       │  Message   │       │
       │      │     │       │
       │      └─────┘       │
       │                    │
       ├─ Valid             │
       │       │            │
       │       ▼            │
       │  ┌─────────────┐   │
       │  │  Send to    │   │
       │  │  Backend    │   │
       │  │  API        │   │
       │  └─────────────┘   │
       │        │           │
       └────────┼───────────┘
                │
                ▼
      ┌──────────────────────┐
      │  Backend Returns     │
      │  Access Token        │
      └──────┬───────────────┘
             │
             ├─ Success
             │     │
             │     ▼
             │ ┌──────────────┐
             │ │ Store Token  │
             │ │ in Local     │
             │ │ Storage      │
             │ └──────┬───────┘
             │        │
             │        ▼
             │ ┌──────────────┐
             │ │ Load User    │
             │ │ Profile      │
             │ └──────┬───────┘
             │        │
             │        ▼
             │ ┌──────────────┐
             │ │ Fetch Initial│
             │ │ Data (Locs,  │
             │ │ Rules, etc)  │
             │ └──────┬───────┘
             │        │
             │        ▼
             │ ┌──────────────┐
             │ │ Redirect to  │
             │ │ Overview     │
             │ │ Page         │
             │ └──────────────┘
             │
             ├─ Failure
             │     │
             │     ▼
             │ ┌──────────────┐
             │ │ Show Error   │
             │ │ Toast        │
             │ └──────────────┘
             │
             ▼
          END
```

## Storage Strategy

```
┌──────────────────────────────────────────────────────┐
│                 BROWSER STORAGE                       │
├──────────────────────────────────────────────────────┤
│                                                       │
│  LOCAL STORAGE (Persistent)                         │
│  ├─ weatherops_token - JWT Authentication Token     │
│  ├─ weatherops_theme - Theme Preference (dark/light)│
│  └─ weatherops_user_* - User Profile Cache          │
│                                                       │
│  SESSION STORAGE (Session-scoped)                   │
│  ├─ weatherops_temp_filters - Temporary Filters    │
│  └─ weatherops_scroll_position - UI State           │
│                                                       │
│  MEMORY (React State)                               │
│  ├─ User Profile Object                             │
│  ├─ Locations Array                                 │
│  ├─ Rules Array                                     │
│  ├─ Alerts Array                                    │
│  ├─ Active Page State                               │
│  ├─ UI States (Sidebar, Loading, etc)               │
│  └─ Toast Queue                                     │
│                                                       │
│  COOKIES (If needed)                                │
│  └─ weatherops_session - Session Cookie             │
│                                                       │
└──────────────────────────────────────────────────────┘
```

## Error Handling & Recovery

```
┌─────────────────────────────────────┐
│     ERROR OCCURS                     │
└──────────┬──────────────────────────┘
           │
           ▼
┌─────────────────────────────────────┐
│  ErrorBoundary Component Catches    │
│  React Component Errors             │
└──────────┬──────────────────────────┘
           │
           ├─ Caught
           │     │
           │     ▼
           │  ┌──────────────────────┐
           │  │  Display Error UI    │
           │  │  with Retry Button   │
           │  └──────────────────────┘
           │
           ├─ Not Caught (API Error)
           │     │
           │     ▼
           │  ┌──────────────────────┐
           │  │  Check Error Type    │
           │  └──────┬───────────────┘
           │         │
           │         ├─ 401 Unauthorized
           │         │    │
           │         │    ▼
           │         │ ┌──────────────┐
           │         │ │ Clear Token  │
           │         │ │ Redirect to  │
           │         │ │ Login        │
           │         │ └──────────────┘
           │         │
           │         ├─ 404 Not Found
           │         │    │
           │         │    ▼
           │         │ ┌──────────────┐
           │         │ │ Show "Not    │
           │         │ │ Found"       │
           │         │ │ Toast        │
           │         │ └──────────────┘
           │         │
           │         ├─ 500+ Server Error
           │         │    │
           │         │    ▼
           │         │ ┌──────────────┐
           │         │ │ Show Error   │
           │         │ │ Toast +      │
           │         │ │ Retry Option │
           │         │ └──────────────┘
           │         │
           │         └─ Network Error
           │              │
           │              ▼
           │           ┌──────────────┐
           │           │ Show Network │
           │           │ Error Toast  │
           │           └──────────────┘
           │
           ▼
       RECOVERY
       (Retry / Manual Action)
```

## Key Features & Workflows

### 1. Location Management
- **Create**: User adds a location with name, latitude, and longitude
- **Read**: Fetch and display all locations with weather summary
- **Update**: Edit location coordinates
- **Delete**: Remove location and all associated rules/alerts

### 2. Weather Rules
- **Define**: Create rules (e.g., Temperature > 30°C)
- **Manage**: Toggle rules on/off, delete rules
- **Monitor**: Backend continuously checks if rules are triggered
- **Alert**: When rule triggers, alert is created automatically

### 3. Alert Management
- **Receive**: Alerts generated when rules trigger
- **View**: Display alerts with severity levels
- **Resolve**: Mark alerts as resolved/acknowledged
- **Export**: Export alerts for reporting

### 4. Data Export
- **Format**: CSV or JSON export
- **Filter**: Export specific alerts/rules
- **Download**: Client-side file download

### 5. User Preferences
- **Theme**: Dark/Light mode toggle (persisted)
- **Notifications**: Enable/disable email, SMS, system alerts
- **Profile**: View and manage user information

---

**Last Updated**: June 2026  
**Version**: 1.0.0
