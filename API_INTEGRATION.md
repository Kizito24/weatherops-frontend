# WeatherOps Frontend - API Integration & Data Flow

## API Layer Architecture

```
┌────────────────────────────────────────────────────────┐
│           FRONTEND APPLICATION                         │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │         REACT COMPONENTS                         │  │
│  │                                                  │  │
│  │  (LocationsPage, RulesPage, AlertsPage, etc)   │  │
│  └────────────┬─────────────────────────────────────┘  │
│               │                                        │
│               │ Uses                                   │
│               ▼                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │      API CLIENT LAYER (/lib/api)                │  │
│  │                                                  │  │
│  │  • authApi          - Authentication            │  │
│  │  • locationsApi     - Location CRUD             │  │
│  │  • rulesApi         - Rule Management           │  │
│  │  • alertsApi        - Alert Management          │  │
│  │  • weatherApi       - Weather Data              │  │
│  │  • treeApi          - Tree Analysis             │  │
│  │                                                  │  │
│  └────────────┬─────────────────────────────────────┘  │
│               │                                        │
│               │ Uses                                   │
│               ▼                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │      UTILITY LAYER                              │  │
│  │                                                  │  │
│  │  • tokenManager     - JWT Token Management      │  │
│  │  • validation       - Input Validation          │  │
│  │  • axios-instance   - HTTP Client Setup         │  │
│  │                                                  │  │
│  └────────────┬─────────────────────────────────────┘  │
│               │                                        │
└───────────────┼────────────────────────────────────────┘
                │ HTTP/HTTPS
                │ REST API Calls
                ▼
┌────────────────────────────────────────────────────────┐
│        BACKEND API SERVER                              │
│  (Handles all business logic & database operations)   │
└────────────────────────────────────────────────────────┘
```

## API Client Structure

```typescript
// src/lib/api/auth.ts
export const authApi = {
  login(email: string, password: string): Promise<{token: string}>
  register(email: string, password: string, name: string): Promise<{token: string}>
  logout(): Promise<void>
  getCurrentUser(): Promise<UserProfile>
};

// src/lib/api/locations.ts
export const locationsApi = {
  list(): Promise<Location[]>
  get(id: string): Promise<Location>
  create(name: string, lat: number, lng: number): Promise<Location>
  update(id: string, name: string, lat: number, lng: number): Promise<Location>
  delete(id: string): Promise<void>
};

// src/lib/api/rules.ts
export const rulesApi = {
  list(locationId: string): Promise<Rule[]>
  get(id: string): Promise<Rule>
  create(locationId: string, metric: WeatherMetric, operator: RuleOperator, threshold: number): Promise<Rule>
  toggleActive(id: string, isActive: boolean): Promise<Rule>
  delete(id: string): Promise<void>
};

// src/lib/api/alerts.ts
export const alertsApi = {
  list(): Promise<Alert[]>
  get(id: string): Promise<Alert>
  resolve(id: string): Promise<Alert>
  delete(id: string): Promise<void>
};

// src/lib/api/weather.ts
export const weatherApi = {
  getCurrent(locationId: string): Promise<WeatherData>
  getForecast(locationId: string): Promise<WeatherData>
  getUsage(): Promise<WeatherUsage>
};

// src/lib/api/tree.ts
export const treeApi = {
  analyze(imageUrl: string, metadata: TreeMetadata): Promise<TreeAnalysis>
  getUsage(): Promise<TreeUsage>
};
```

## Authentication Flow

### Login Request-Response Cycle

```
┌──────────────┐
│ User Input   │
│ - Email      │
│ - Password   │
└──────┬───────┘
       │
       ▼
┌──────────────────────────────┐
│  authApi.login(email, pwd)   │
│                              │
│  POST /api/auth/login        │
│  {                           │
│    "email": "user@mail.com", │
│    "password": "securepass"  │
│  }                           │
└──────┬───────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│  Backend Validation          │
│  - Check user exists         │
│  - Verify password           │
│  - Generate JWT token        │
└──────┬───────────────────────┘
       │
       ├─ Invalid Credentials
       │      │
       │      ▼
       │  Response 401:
       │  {
       │    "error": "Invalid credentials"
       │  }
       │      │
       │      └─ Show Error Toast
       │
       ├─ Valid Credentials
       │      │
       │      ▼
       │  Response 200:
       │  {
       │    "token": "eyJhbGc...",
       │    "user": {
       │      "id": "user-123",
       │      "email": "user@mail.com",
       │      "name": "John Doe",
       │      "role": "User",
       │      "apiKey": "sk-...",
       │      "notifications": {...}
       │    }
       │  }
       │      │
       │      ▼
       │  Store token in localStorage
       │      │
       │      ▼
       │  Create user profile object
       │      │
       │      ▼
       │  Load initial platform data
       │      │
       │      ▼
       │  Navigate to Overview page
       │
       └─────────────────────────
```

### Token Management Flow

```
┌────────────────────────────────────────┐
│      TOKEN LIFECYCLE MANAGEMENT          │
└────────────────────────────────────────┘

Token Storage:
    localStorage.weatherops_token = "eyJhbGc..."

Token Usage:
    Every API request adds header:
    Authorization: Bearer eyJhbGc...

Axios Interceptor:
    
    request interceptor:
        └─ Add token to headers
    
    response interceptor:
        ├─ Token Valid
        │     └─ Return response
        │
        ├─ Token Expired (401)
        │     ├─ Clear token from storage
        │     ├─ Clear user from state
        │     └─ Redirect to login
        │
        └─ Other Errors
              └─ Pass to error handler

Token Expiration Check:
    
    useEffect(() => {
      const checkToken = () => {
        if (tokenManager.isAuthenticated()) {
          if (tokenManager.isExpired()) {
            logout();
          }
        }
      };
      
      const interval = setInterval(checkToken, 60000); // Check every minute
      return () => clearInterval(interval);
    }, []);
```

## Location API Endpoints

### Sequence Diagram: Create Location

```
┌──────────────┐                    ┌───────────────┐
│   Frontend   │                    │   Backend     │
└──────┬───────┘                    └───────┬───────┘
       │                                    │
       │ POST /api/locations               │
       │ {                                 │
       │   "name": "New York",            │
       │   "latitude": 40.7128,           │
       │   "longitude": -74.0060          │
       │ }                                 │
       ├───────────────────────────────────>│
       │                                    │
       │                         Validate  │
       │                         Input     │
       │                                    │
       │                         Save to   │
       │                         Database  │
       │                                    │
       │  200 OK                           │
       │  {                                │
       │    "id": "loc-xyz",              │
       │    "name": "New York",           │
       │    "latitude": 40.7128,          │
       │    "longitude": -74.0060,        │
       │    "createdAt": "2026-06-06T..." │
       │  }                                │
       │<───────────────────────────────────┤
       │                                    │
       ├─ Show Toast                       │
       │  "Location Provisioned"           │
       │                                    │
       ├─ Call loadPlatformData()         │
       │                                    │
       └─ Update locations state           │
```

### CRUD Operations Timeline

```
Locations List Request:
    
    App mounts / User loads Locations Page
        │
        ▼
    GET /api/locations
        │
        ├─ Returns: Location[]
        │
        ├─ Filter for pagination
        │
        ├─ Display 10 per page
        │
        └─ Set Pagination state

Create Location:
    
    User submits form
        │
        ▼
    POST /api/locations
        │
        ├─ Success: Reload locations
        │
        └─ Error: Show error message

Update Location:
    
    User clicks edit & submits
        │
        ▼
    PUT /api/locations/{id}
        │
        ├─ Success: Reload locations
        │
        └─ Error: Show error message

Delete Location:
    
    User clicks delete
        │
        ▼
    Show confirmation dialog
        │
        ├─ Cancel: Close dialog
        │
        └─ Confirm:
            │
            ▼
            DELETE /api/locations/{id}
            │
            ├─ Success: Reload locations + Rules + Alerts
            │
            └─ Error: Show error message
```

## Rules API Endpoints

### Rule Creation Data Flow

```
┌──────────────────────────────────────────────────────┐
│           RULE CREATION PROCESS                       │
└──────────────────────────────────────────────────────┘

Step 1: Select Location
    └─ Locations already loaded in state

Step 2: Select Metric
    Options:
    ├─ temperature (°C)
    ├─ rainfall (mm)
    ├─ wind_speed (km/h)
    └─ humidity (%)

Step 3: Select Operator
    Options:
    ├─ > (greater than)
    ├─ < (less than)
    ├─ >= (greater than or equal)
    ├─ <= (less than or equal)
    └─ == (equal to)

Step 4: Enter Threshold Value
    └─ Numeric input with metric-specific validation

Step 5: Form Validation
    
    POST /api/rules
    {
      "location_id": "loc-123",
      "metric": "temperature",
      "operator": ">",
      "threshold": 30
    }
        │
        ├─ Backend validates threshold range
        │
        ├─ Checks location exists
        │
        ├─ Creates rule record
        │
        └─ Returns: Rule object

Step 6: Response Handling
    
    Success:
    ├─ Show success toast
    ├─ Reload all rules
    └─ Clear form & close modal
    
    Error:
    ├─ Show error toast
    └─ Keep form open for retry

Step 7: Rule Activation
    
    Backend starts monitoring:
    └─ Continuously checks weather data
       against this rule's threshold
       
    When breached:
    └─ Creates Alert automatically
```

### Rule Toggle (Enable/Disable) Flow

```
Current State: Rule is ACTIVE
    │
    ▼
User clicks toggle
    │
    ▼
Optimistic Update: UI shows INACTIVE immediately
    │
    ▼
PUT /api/rules/{id}/toggle
{
  "is_active": false
}
    │
    ├─ Success
    │     │
    │     ├─ Show toast "Incident Trigger Paused"
    │     │
    │     └─ Reload rules to get latest state
    │
    └─ Error
          │
          ├─ Revert UI to ACTIVE
          │
          └─ Show error toast
```

## Alerts API Endpoints

### Alert Resolution Workflow

```
┌──────────────────────────────────────────────────────┐
│           ALERT RESOLUTION WORKFLOW                   │
└──────────────────────────────────────────────────────┘

Alert Created (Backend):
    
    Rule: Temperature > 30°C triggered
    Current Temp: 32°C
        │
        ▼
    Create Alert Record:
    {
      "id": "alert-xyz",
      "location_id": "loc-123",
      "rule_id": "rule-456",
      "metric": "temperature",
      "value": 32,
      "threshold": 30,
      "operator": ">",
      "severity": "HIGH",
      "status": "active",
      "timestamp": "2026-06-06T10:30:00Z"
    }
        │
        ▼
    Frontend polls/subscribed to alerts
        │
        ├─ Fetch alert from API
        │
        ├─ Add to alerts state
        │
        └─ Alert badge updates

User Views Alert:
    
    Alerts Page loads:
    ├─ GET /api/alerts
    │   Returns: Alert[]
    │
    ├─ Display in list with:
    │   - Location name
    │   - Metric & current value
    │   - Threshold info
    │   - Severity badge (RED for HIGH)
    │   - Timestamp
    │   - Resolve button
    │
    └─ Unresolved alerts shown first

User Resolves Alert:
    
    Clicks "Resolve" button
        │
        ▼
    POST /api/alerts/{id}/resolve
        │
        ├─ Backend marks as "resolved"
        │
        ├─ Sets resolvedAt timestamp
        │
        └─ Returns updated Alert
        
    Frontend handling:
    ├─ Show success toast
    ├─ Reload alerts list
    └─ Update UI (alert disappears or grayed out)
```

### Alert Filtering & Display Logic

```
All Alerts Retrieved from API
    │
    ├─ Filter by status: Only "active" or "recent" (< 24h)
    │
    ├─ Group by severity
    │
    ├─ Sort by timestamp (newest first)
    │
    └─ Apply user filters (if any)
            │
            ├─ Filter by location
            │
            ├─ Filter by severity
            │
            └─ Filter by metric

Displayed Result:
    
    ┌─────────────────────────────────┐
    │ 🔴 HIGH SEVERITY ALERT          │
    │ Location: New York              │
    │ Temperature: 35°C > 30°C        │
    │ Time: 10 minutes ago            │
    │ [RESOLVE] Button                │
    └─────────────────────────────────┘
    
    ┌─────────────────────────────────┐
    │ 🟠 MEDIUM SEVERITY ALERT        │
    │ Location: London                │
    │ Rainfall: 25mm > 20mm           │
    │ Time: 2 hours ago               │
    │ [RESOLVE] Button                │
    └─────────────────────────────────┘
```

## Weather API Endpoints

### Weather Data Fetching Flow

```
┌──────────────────────────────────────────────────────┐
│         WEATHER DATA REQUEST WORKFLOW                 │
└──────────────────────────────────────────────────────┘

User Navigates to Weather Page
    │
    ▼
Component Mounts
    │
    ├─ Display skeleton loaders
    │
    ├─ Fetch available locations
    │
    └─ Auto-select first location

User Selects Location
    │
    ├─ Set selected location state
    │
    └─ Trigger data fetch

Parallel API Calls:
    
    Promise.all([
      weatherApi.getCurrent(locationId),
      weatherApi.getForecast(locationId),
      weatherApi.getUsage()
    ])
    
    ├─ GET /api/weather/{id}/current
    │  Returns: {
    │    temperature: 28,
    │    humidity: 65,
    │    wind_speed: 12,
    │    rainfall: 0,
    │    condition: "Partly Cloudy",
    │    pressure: 1013,
    │    visibility: 10,
    │    uv_index: 6,
    │    timestamp: "2026-06-06T10:30:00Z"
    │  }
    │
    ├─ GET /api/weather/{id}/forecast
    │  Returns: {
    │    daily: [{...}, {...}], // 7 days
    │    hourly: [{...}, {...}]  // 24 hours
    │  }
    │
    └─ GET /api/weather/usage
       Returns: {
         plan: "pro",
         requests_used: 120,
         requests_remaining: 130,
         requests_limit: 250
       }

Data Processing:
    
    ├─ Format temperature for display
    │
    ├─ Build hourly chart data
    │
    ├─ Build daily forecast cards
    │
    ├─ Calculate usage percentage
    │
    └─ Show warning if usage > 80%

Display Weather Data:
    
    ┌──────────────────────────────┐
    │  Current Weather             │
    │                              │
    │  28°C  ☁️ Partly Cloudy      │
    │  Humidity: 65%  Wind: 12 km/h
    │                              │
    └──────────────────────────────┘
    
    ┌──────────────────────────────┐
    │  Hourly Forecast (24h)       │
    │  [Chart visualization]       │
    └──────────────────────────────┘
    
    ┌──────────────────────────────┐
    │  7-Day Forecast              │
    │  [Daily forecast cards]      │
    └──────────────────────────────┘
    
    ┌──────────────────────────────┐
    │  API Usage                   │
    │  [████████░] 120/250 used    │
    └──────────────────────────────┘

AI Summary (Optional):
    
    GET /api/weather/{id}/summary?ai=true
        │
        └─ Returns AI-generated weather summary
```

## Error Handling Strategy

### API Error Response Handling

```
┌──────────────────────────────────────────────────────┐
│           ERROR HANDLING FLOW                         │
└──────────────────────────────────────────────────────┘

API Call Made
    │
    ▼
Network Response Received
    │
    ├─ Status 2xx (Success)
    │     └─ Process and return data
    │
    └─ Status 4xx-5xx (Error)
         │
         ▼
      Check Status Code
      │
      ├─ 400 Bad Request
      │     │
      │     ├─ User validation error
      │     │
      │     └─ Show: "Invalid input - field required"
      │
      ├─ 401 Unauthorized
      │     │
      │     ├─ Token invalid/expired
      │     │
      │     ├─ Clear token from storage
      │     │
      │     ├─ Clear user state
      │     │
      │     └─ Redirect to login page
      │
      ├─ 403 Forbidden
      │     │
      │     ├─ User lacks permissions
      │     │
      │     └─ Show: "You don't have permission"
      │
      ├─ 404 Not Found
      │     │
      │     ├─ Resource doesn't exist
      │     │
      │     └─ Show: "Resource not found"
      │
      ├─ 409 Conflict
      │     │
      │     ├─ Data conflict (e.g., duplicate)
      │     │
      │     └─ Show: "This item already exists"
      │
      ├─ 429 Too Many Requests
      │     │
      │     ├─ Rate limited
      │     │
      │     └─ Show: "Too many requests - please wait"
      │
      ├─ 500+ Server Error
      │     │
      │     ├─ Backend error
      │     │
      │     ├─ Retry logic (exponential backoff)
      │     │
      │     └─ Show: "Server error - please try again"
      │
      └─ Network Error
            │
            ├─ No internet connection
            │
            └─ Show: "Connection failed - check internet"

Toast Display:
    
    severity = ERROR or MEDIUM/HIGH based on type
    
    showToast(
      title: "Operation Failed",
      message: errorMessage,
      severity: "MEDIUM"
    );

Error Recovery Options:
    
    ├─ Automatic: Retry after delay
    │
    ├─ User-triggered: Retry button in toast
    │
    ├─ Manual: User fixes issue and retries
    │
    └─ Fallback: Show cached data or empty state
```

### Request Retry Strategy

```
Initial Request
    │
    ├─ Success → Return data
    │
    └─ Network/Server Error
         │
         ├─ Attempt 1 (wait 1s)
         │     │
         │     ├─ Success → Return data
         │     │
         │     └─ Error → Continue
         │
         ├─ Attempt 2 (wait 2s)
         │     │
         │     ├─ Success → Return data
         │     │
         │     └─ Error → Continue
         │
         ├─ Attempt 3 (wait 4s)
         │     │
         │     ├─ Success → Return data
         │     │
         │     └─ Error → Continue
         │
         └─ Max Attempts Reached
              │
              ├─ Show error toast
              │
              └─ Offer manual retry
```

## Data Validation

### Frontend Validation Rules

```
┌──────────────────────────────────────────────────────┐
│      FRONTEND DATA VALIDATION                         │
└──────────────────────────────────────────────────────┘

Location Form:
    ├─ Name
    │  ├─ Required: Yes
    │  ├─ Length: 1-100 characters
    │  ├─ Type: String
    │  └─ Show error if empty or too long
    │
    ├─ Latitude
    │  ├─ Required: Yes
    │  ├─ Range: -90 to 90
    │  ├─ Type: Number
    │  └─ Show error if outside range
    │
    └─ Longitude
       ├─ Required: Yes
       ├─ Range: -180 to 180
       ├─ Type: Number
       └─ Show error if outside range

Rule Form:
    ├─ Location
    │  ├─ Required: Yes
    │  ├─ Must exist in locations array
    │  └─ Show error if not selected
    │
    ├─ Metric
    │  ├─ Required: Yes
    │  ├─ Options: temperature | rainfall | wind_speed | humidity
    │  └─ Show error if not selected
    │
    ├─ Operator
    │  ├─ Required: Yes
    │  ├─ Options: > | < | >= | <= | ==
    │  └─ Show error if not selected
    │
    └─ Threshold
       ├─ Required: Yes
       ├─ Type: Number
       ├─ Metric-specific ranges:
       │  ├─ temperature: -50 to 60
       │  ├─ rainfall: 0 to 500
       │  ├─ wind_speed: 0 to 300
       │  └─ humidity: 0 to 100
       └─ Show error if outside range

Authentication:
    ├─ Email
    │  ├─ Required: Yes
    │  ├─ Format: valid email
    │  └─ Show error if invalid
    │
    └─ Password
       ├─ Required: Yes
       ├─ Length: 8+ characters
       ├─ Must contain: uppercase, lowercase, number
       └─ Show error if requirements not met
```

---

**Last Updated**: June 2026  
**Version**: 1.0.0
