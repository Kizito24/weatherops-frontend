# WeatherOps Frontend - Component Documentation

## Component Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    APP (Root)                            │
│  - Global State Manager                                 │
│  - Authentication Handler                               │
│  - Route Coordinator                                     │
└────────────┬──────────────────────────────────────────┬─┘
             │                                          │
             │ (conditional render)                     │
             │                                          │
             ▼                                          ▼
    ┌──────────────────┐                   ┌──────────────────┐
    │   AuthPage       │                   │  Dashboard       │
    │  (Login/Reg)     │                   │  Layout          │
    └──────────────────┘                   │  - Sidebar       │
                                           │  - Navbar        │
                                           │  - Pages         │
                                           │  - Modals        │
                                           │  - Toasts        │
                                           └──────────────────┘
```

## Page Components (Route Views)

### 1. AuthPage

**Purpose**: Authenticate users via login or registration

**State**:
```typescript
interface AuthPageProps {
  id: string;
  onAuthSuccess: (user: UserProfile) => void;
  initialMode: 'login' | 'register';
}
```

**Child Components**:
- LoginForm
  - FormInput (email)
  - FormInput (password)
  - Submit Button
  - Register Link

- RegisterForm
  - FormInput (name)
  - FormInput (email)
  - FormInput (password)
  - FormSelect (role)
  - Submit Button
  - Login Link

**Data Flow**:
```
User Input → Validation → API Call → Success/Error → onAuthSuccess callback
```

---

### 2. OverviewPage

**Purpose**: Display dashboard with KPIs and summary metrics

**Props**:
```typescript
interface OverviewPageProps {
  locations: Location[];
  rules: Rule[];
  alerts: Alert[];
  onNavigateToPage: (page: string) => void;
  onRefreshAll: () => Promise<void>;
  isLoading: boolean;
}
```

**Features**:
- Total Locations KPI
- Active Rules Count
- Unresolved Alerts Count
- Recent Alerts Summary
- Quick Navigation Cards

**Child Components**:
- KPICard (x3)
  - Icon
  - Title
  - Value
  - Trend Indicator

- AlertsSummary
  - Recent alerts list
  - Severity badges

**Rendering Logic**:
```
if (isLoading) → Show Skeletons
else if (data.length === 0) → Show EmptyState
else → Render KPI Cards + Summary
```

---

### 3. WeatherPage

**Purpose**: Display current weather and forecast for locations

**Props**:
```typescript
interface WeatherPageProps {
  locations: Location[];
  onNavigateToPage: (page: string) => void;
}
```

**Features**:
- Location selector dropdown
- Current weather conditions
- Hourly forecast (next 24h)
- Daily forecast (next 7 days)
- AI weather summary
- API usage statistics

**Child Components**:
- LocationSelector
  - Dropdown list of locations
- WeatherDisplay
  - Current metrics
  - Forecast grid
  - Chart visualization
- WeatherUsagePanel
  - Usage bar
  - Remaining quota

**Data Fetching**:
```typescript
useEffect(() => {
  if (selectedLocation) {
    fetchWeatherData(selectedLocation.id);
  }
}, [selectedLocation]);
```

---

### 4. LocationsPage

**Purpose**: Manage locations (CRUD operations)

**Props**:
```typescript
interface LocationsPageProps {
  locations: Location[];
  onAddLocation: (name: string, lat: number, lng: number) => Promise<Location>;
  onEditLocation: (id: string, name: string, lat: number, lng: number) => Promise<Location>;
  onDeleteLocation: (id: string) => Promise<void>;
  isLoading: boolean;
}
```

**Features**:
- Add new location form
- Location list with edit/delete actions
- Pagination support
- Map integration (optional)
- Validation errors

**Child Components**:
- LocationForm
  - FormInput (name)
  - FormInput (latitude)
  - FormInput (longitude)
  - Submit Button

- LocationCard (repeated per location)
  - Location name & coordinates
  - Edit button
  - Delete button
  - Created date

- Pagination
  - Page numbers
  - Next/Previous buttons

- EmptyState
  - "No locations" message
  - CTA to add first location

**Data Handling**:
```
Add Location Flow:
Form Submit → Validation → onAddLocation() → API Call → 
Reload Data → Update locations[] → Close Form → Show Toast

Delete Location Flow:
Click Delete → Confirmation → onDeleteLocation() → API Call → 
Reload Data → Update locations[] → Show Toast
```

---

### 5. RulesPage

**Purpose**: Create and manage weather alert rules

**Props**:
```typescript
interface RulesPageProps {
  rules: Rule[];
  locations: Location[];
  onCreateRule: (locationId: string, metric: WeatherMetric, 
                 operator: RuleOperator, threshold: number) => Promise<Rule>;
  onToggleRule: (id: string) => Promise<Rule>;
  onDeleteRule: (id: string) => Promise<void>;
  isLoading: boolean;
}
```

**Features**:
- Rule creation form
- Rules list with status indicators
- Enable/disable toggle
- Delete with confirmation
- Pagination

**Child Components**:
- RuleForm
  - FormSelect (location)
  - FormSelect (metric: temperature, rainfall, wind_speed, humidity)
  - FormSelect (operator: >, <, >=, <=, ==)
  - FormInput (threshold value)
  - Submit Button

- RuleCard (repeated per rule)
  - Location name
  - Metric type
  - Operator & threshold
  - Active/Inactive toggle
  - Delete button

- Pagination
- EmptyState

**Rule Creation Logic**:
```typescript
const handleCreateRule = async (formData) => {
  // Validate all fields
  if (!formData.locationId || !formData.metric || !formData.threshold) {
    showError('All fields required');
    return;
  }
  
  // API call
  const rule = await onCreateRule(
    formData.locationId,
    formData.metric,
    formData.operator,
    formData.threshold
  );
  
  // Success handling
  showToast(`Alert Rule Armed for ${location.name}`);
  resetForm();
};
```

---

### 6. AlertsPage

**Purpose**: View and resolve weather alert incidents

**Props**:
```typescript
interface AlertsPageProps {
  alerts: Alert[];
  locations: Location[];
  onDeleteAlert: (id: string) => Promise<boolean>;
  onClearAllAlerts: () => Promise<boolean>;
  isLoading: boolean;
}
```

**Features**:
- Alerts list with details
- Severity color coding (HIGH/MEDIUM/LOW)
- Resolve/Acknowledge action
- Filter by severity
- Pagination
- Bulk clear option

**Child Components**:
- AlertCard (repeated per alert)
  - Location name
  - Metric information
  - Current value vs threshold
  - Severity badge (red/orange/green)
  - Timestamp
  - Resolve button

- SeverityFilter
  - Checkbox filters
  - Reset button

- Pagination
- EmptyState

**Severity Color Mapping**:
```typescript
const severityColors = {
  HIGH: 'bg-red-100 text-red-800 border-red-300',
  MEDIUM: 'bg-yellow-100 text-yellow-800 border-yellow-300',
  LOW: 'bg-green-100 text-green-800 border-green-300'
};
```

---

### 7. TreesPage

**Purpose**: Tree health analysis from satellite imagery

**Props**: (minimal props)

**Features**:
- Image upload form
- Analysis results display
- Tree health breakdown
- Recommendations
- Tree species detection
- Confidence scoring

**Child Components**:
- TreeUploadForm
  - File input
  - Metadata inputs (county, acres, etc)
  - Submit button

- TreeAnalysisResult
  - Tree health metrics
  - Canopy coverage
  - Species information
  - Satellite overlay image
  - Recommendations list

---

### 8. SettingsPage

**Purpose**: User profile and notification preferences

**Props**:
```typescript
interface SettingsPageProps {
  user: UserProfile;
  onUpdateUser: (updates: Partial<UserProfile>) => Promise<UserProfile>;
}
```

**Features**:
- View user profile (name, email, role)
- API key display/generation
- Notification preferences (email, SMS, system)
- Account settings
- Logout option

**Child Components**:
- ProfileSection
  - Name display
  - Email display
  - Role display

- APIKeySection
  - Key display (masked)
  - Copy button
  - Regenerate button

- NotificationPreferences
  - Toggle email notifications
  - Toggle SMS notifications
  - Toggle system notifications
  - Save button

---

## Shared/Utility Components

### Navigation Components

#### Sidebar
**Purpose**: Main navigation and user menu

**Props**:
```typescript
interface SidebarProps {
  activePage: string;
  setActivePage: (page: string) => void;
  user: UserProfile;
  onLogout: () => void;
  isCollapsed: boolean;
  setIsCollapsed: (collapsed: boolean) => void;
  alertCount: number;
}
```

**Features**:
- Navigation links to all pages
- Alert badge with count
- User profile section
- Logout button
- Collapse/expand toggle
- Responsive behavior

**Navigation Structure**:
```
├─ Overview
├─ Weather
├─ Locations
├─ Rules
├─ Alerts (with badge)
├─ Trees
├─ Settings
└─ Logout
```

**Responsive Behavior**:
```typescript
if (window.innerWidth < 1024) {
  autoCollapse = true;
}
```

---

#### Navbar
**Purpose**: Top navigation bar with actions

**Props**:
```typescript
interface NavbarProps {
  activePage: string;
  user: UserProfile;
  sidebarCollapsed: boolean;
  setSidebarCollapsed: (value: boolean) => void;
  darkMode: boolean;
  setDarkMode: (value: boolean) => void;
  onSimulateIncident: () => void;
  isSimulating: boolean;
  onExport: () => void;
}
```

**Features**:
- Page title display
- Dark mode toggle
- Simulate incident button
- Export button
- User dropdown (profile, settings, logout)

---

### Form Components

#### FormInput
**Purpose**: Reusable text input field

**Props**:
```typescript
interface FormInputProps {
  label: string;
  type: string;
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
  error?: string;
  required?: boolean;
}
```

#### FormSelect
**Purpose**: Reusable dropdown/select field

**Props**:
```typescript
interface FormSelectProps {
  label: string;
  options: Array<{label: string, value: string}>;
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
  error?: string;
  required?: boolean;
}
```

---

### Display Components

#### KPICard
**Purpose**: Display key performance indicators

**Props**:
```typescript
interface KPICardProps {
  icon: React.ReactNode;
  title: string;
  value: string | number;
  trend?: number;
  trendDirection?: 'up' | 'down';
  color?: string;
}
```

**Example**:
```
┌──────────────────────┐
│ 📍 Total Locations   │
│      15              │
│      ↑ +3 this week  │
└──────────────────────┘
```

---

#### Badge
**Purpose**: Display severity/status indicators

**Props**:
```typescript
interface BadgeProps {
  severity: 'LOW' | 'MEDIUM' | 'HIGH';
  label: string;
}
```

**Rendering**:
```
HIGH:   [🔴 HIGH SEVERITY]
MEDIUM: [🟠 MEDIUM SEVERITY]
LOW:    [🟢 LOW SEVERITY]
```

---

#### EmptyState
**Purpose**: Show when no data available

**Props**:
```typescript
interface EmptyStateProps {
  icon: React.ReactNode;
  title: string;
  message: string;
  actionLabel?: string;
  onAction?: () => void;
}
```

---

#### Pagination
**Purpose**: Navigate large lists

**Props**:
```typescript
interface PaginationProps {
  currentPage: number;
  totalPages: number;
  onPageChange: (page: number) => void;
}
```

---

### Modal & Notification Components

#### ExportModal
**Purpose**: Export data in various formats

**Props**:
```typescript
interface ExportModalProps {
  isOpen: boolean;
  onClose: () => void;
  alerts: Alert[];
  rules: Rule[];
  locations: Map<string, string>;
}
```

**Features**:
- Format selection (CSV, JSON)
- Data type selection (Alerts, Rules, Locations)
- Download trigger
- Loading state

**Export Flow**:
```
Format Selection → Data Preparation → CSV/JSON Generation → Download
```

---

#### Toast System
**Purpose**: Display notifications

**Toast Types**:
```typescript
type Toast = {
  id: string;
  title: string;
  message: string;
  severity: 'LOW' | 'MEDIUM' | 'HIGH' | 'INFO';
};
```

**Severity Styling**:
```
HIGH:   Red background, fire icon
MEDIUM: Orange background, warning icon
LOW:    Green background, checkmark icon
INFO:   Blue background, info icon
```

**Auto-dismiss**: 5 seconds
**Manual close**: Click X button

---

#### ErrorBoundary
**Purpose**: Catch and display component errors

**Props**:
```typescript
interface ErrorBoundaryProps {
  children: React.ReactNode;
}
```

**Features**:
- Catch React errors
- Display error message
- Retry button
- Fallback UI

---

## Component Communication Patterns

### Props Flow (Parent → Child)
```
App
  ├─ locations prop → LocationsPage
  │                → RulesPage
  │                → AlertsPage
  │                → OverviewPage
  │
  ├─ rules prop → RulesPage
  │            → OverviewPage
  │
  ├─ alerts prop → AlertsPage
  │             → OverviewPage
  │
  └─ user prop → Sidebar
               → Navbar
               → SettingsPage
```

### Event Callbacks (Child → Parent)
```
LocationsPage
  └─ onAddLocation() → App.handleAddLocation()
                    → API Call
                    → loadPlatformData()
                    → State Update
                    → Re-render with new data

RulesPage
  ├─ onCreateRule() → App.handleCreateRule()
  ├─ onToggleRule() → App.handleToggleRuleActive()
  └─ onDeleteRule() → App.handleDeleteRule()

AlertsPage
  └─ onDeleteAlert() → App.handleDeleteAlert()

SettingsPage
  └─ onUpdateUser() → App.handleUpdateUserProfile()
```

---

## Data Flow Example: Creating a Location

```
User Types Location Name & Coordinates
    │
    ▼
[Form Submit Event]
    │
    ▼
LocationsPage.handleSubmit()
    │
    ├─ Validate input
    │     │
    │     └─ Invalid? → Show error toast → Return
    │
    ├─ Valid? → Call onAddLocation()
    │
    ▼
App.handleAddLocation()
    │
    ├─ Show toast "Creating location..."
    │
    ├─ Call locationsApi.create(name, lat, lng)
    │
    ▼
API Request to Backend
    │
    ├─ Success Response
    │     │
    │     ▼
    │  App.handleAddLocation()
    │     │
    │     ├─ Show success toast "Location Provisioned"
    │     │
    │     ├─ Call loadPlatformData()
    │     │
    │     ├─ Fetch new locations[]
    │     │
    │     ├─ Update state: setLocations(newLocations)
    │     │
    │     └─ Component re-renders with new data
    │
    └─ Error Response
          │
          ▼
       Show error toast with message
```

---

**Last Updated**: June 2026  
**Version**: 1.0.0
