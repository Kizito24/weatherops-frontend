# WeatherOps: Intelligent Weather Monitoring & Alerting Platform

> **Stop reacting to weather. Start planning for it.**

WeatherOps is an intelligent, automated weather monitoring platform that continuously watches weather conditions across your locations and alerts your team when action is needed. Built for businesses that depend on weather intelligence to make critical decisions.

**Live Demo:** https://weatherops-frontend.vercel.app

---

## 🎯 What is WeatherOps?

WeatherOps transforms weather from a **reactive problem** into a **proactive advantage**. Instead of checking forecasts manually or discovering weather impacts after they happen, WeatherOps automatically:

- ✅ **Monitors** weather 24/7 across unlimited locations
- ✅ **Evaluates** your custom rules every 5 minutes
- ✅ **Alerts** your team via SMS, Email, or Webhooks
- ✅ **Tracks** complete alert history for analysis
- ✅ **Provides** real-time weather data with 7-day forecasts
- ✅ **Analyzes** farm imagery for tree health (satellite analysis)

### Perfect For:
- 🚚 **Logistics & Transportation** - Optimize routes, reduce delays
- 🏗️ **Construction** - Ensure worker safety, protect equipment
- 🌾 **Agriculture** - Optimize irrigation, prevent crop damage
- 🎪 **Events** - Plan contingencies, manage weather risks
- ⚡ **Utilities** - Pre-position crews, minimize outages
- 🏥 **Healthcare** - Ensure continuity of operations
- 🛒 **Retail** - Manage deliveries, customer expectations

---

## 🌟 Core Features

### 📍 Location Management
- Monitor **unlimited locations** simultaneously
- Add locations by GPS coordinates (latitude/longitude)
- Search and filter locations
- Edit location details anytime
- Location-specific weather data and rules

### 🎯 Intelligent Rule Engine
- Create unlimited **weather-based alert rules**
- Monitor 4 weather metrics:
  - 🌡️ **Temperature** (°C)
  - 🌧️ **Rainfall** (mm)
  - 💨 **Wind Speed** (m/s)
  - 💧 **Humidity** (%)
- Use 5 flexible operators: `>`, `<`, `>=`, `<=`, `==`
- No coding required - simple point-and-click interface
- Toggle rules on/off without deletion
- Rules automatically evaluated every 5 minutes

### 🚨 Smart Alert System
- **Automatic Alert Generation** when rules trigger
- **Severity Classification** (LOW/MEDIUM/HIGH) auto-calculated
- **Alert Status Tracking** (active/resolved)
- **Deduplication** prevents alert spam
- **Alert History** with complete audit trail
- **Filtering & Search** by location, severity, status, date
- **Resolution Tracking** for acknowledgment

### 🔔 Multi-Channel Notifications
- **SMS Alerts** via Twilio (field teams, mobile alerts)
- **Email Alerts** via SendGrid (team distribution)
- **Webhooks** (integrate with any system)
- Real-time notification delivery
- Alert details included in all channels

### 🌤️ Comprehensive Weather Data
- **Current Conditions:**
  - Temperature, Humidity, Wind Speed, Rainfall
  - Pressure, Visibility, UV Index, Condition
- **7-Day Forecast:**
  - Daily high/low temperatures
  - Rainfall prediction
  - Wind speed forecast
  - Weather conditions
- **Hourly Breakdown:**
  - Hour-by-hour temperature
  - Hourly precipitation
  - Wind conditions
- **AI Weather Summary** (powered by Weather-AI)
- **Usage Tracking** (API requests, remaining quota)

### 🌳 Tree Analysis (Satellite Image Analysis)
- Upload satellite or aerial farm images
- Automatic tree detection and analysis:
  - Total tree count
  - Tree density per acre
  - Canopy coverage percentage
  - Tree health assessment (healthy/needs care/needs replacement)
  - Tree species identification
  - Confidence scores
- Custom metadata (farmer ID, county, acreage, location)
- Analysis history and results
- Usage tracking and plan management

### 📊 Dashboard & Analytics
- **Overview Dashboard** with KPI cards:
  - Total monitored locations
  - Active rules count
  - Alerts in last 24 hours
  - Alerts in last hour
  - Active (unresolved) alerts
  - System health and uptime
- **Real-time Metrics** automatically recalculated
- **Location Performance** view
- **Alert Trends** tracking

### 🔐 User Management
- Secure JWT-based authentication
- User registration and login
- Password encryption
- Session management
- User preferences and settings

### 🎨 User Experience
- **Responsive Design** - Works on desktop, tablet, mobile
- **Dark/Light Mode** - Toggle theme preference
- **Loading Skeletons** - Professional loading states
- **Toast Notifications** - Real-time feedback
- **Modal Dialogs** - Intuitive forms and confirmations
- **Empty States** - Helpful guidance when no data
- **Error Boundaries** - Graceful error handling

### 📤 Export Functionality
- Export alerts data
- Export rules configuration
- Export location data
- CSV/JSON formats

---

## 🏗️ Architecture

### Technology Stack

**Frontend:**
- **Framework:** React 19.0.1 (latest)
- **Language:** TypeScript 5.8
- **Styling:** Tailwind CSS 4.1
- **Build Tool:** Vite 6.2 (lightning-fast HMR)
- **HTTP Client:** Axios 1.6
- **Icons:** Lucide React (546+ icons)
- **Testing:** Playwright (e2e tests)
- **Animation:** Motion 12.23
- **Deployment:** Vercel (auto-deploy from git)

**Backend (Python):**
- **API:** FastAPI 0.115 (high-performance async)
- **ASGI Server:** Uvicorn 0.32
- **ORM:** SQLAlchemy 2.0 (async support)
- **Database:** PostgreSQL 15 (asyncpg driver)
- **Cache:** Redis 5.2 (real-time data)
- **Task Queue:** Celery 5.4 + Celery Beat
- **Authentication:** JWT tokens (python-jose)
- **Password Hashing:** Argon2 (passlib)
- **Migrations:** Alembic 1.14
- **Deployment:** Render (containerized)

**External APIs:**
- **Weather Data:** Weather-AI API (real-time + forecasts)
- **SMS:** Twilio 9.10 (alert delivery)
- **Email:** SendGrid 6.12 (email notifications)

### Data Flow

```
User Creates Rule
     ↓
Frontend sends to Backend API
     ↓
Rule stored in PostgreSQL
     ↓
Celery Beat scheduler (every 5 min)
     ↓
Weather Monitor task:
  1. Fetch all locations
  2. Get current weather from Weather-AI
  3. Evaluate rules via Rule Engine
  4. Create alerts for triggered rules
  5. Send notifications (SMS, Email, Webhooks)
     ↓
User receives alert instantly
     ↓
User dismisses alert in UI
     ↓
Alert marked as resolved
```

---

## 📸 Screenshots & Pages

### 1. **Overview Dashboard**
- At-a-glance KPI metrics
- System health status
- Alert trends
- Quick navigation to other sections

### 2. **Weather Page**
- Current conditions for selected location
- Interactive weather cards
- 7-day forecast view
- Hourly breakdown
- AI-generated weather summary
- API usage statistics

### 3. **Locations Page**
- Table of all monitored locations
- Search locations by name
- Add new location (modal form with lat/lon)
- Edit existing locations
- Delete locations with confirmation
- Location creation timestamp

### 4. **Rules Page**
- All active rules in table format
- Rules grouped by location
- Metric icons (temperature, rainfall, wind, humidity)
- Operator display (>, <, >=, <=, ==)
- Toggle rule on/off
- Delete rule
- Create new rule (modal form)
- Validation feedback

### 5. **Alerts Page**
- Alert history with all triggered alerts
- Filter by severity (LOW/MEDIUM/HIGH)
- Filter by location
- Sort by recency
- Expand alert details
- Dismiss/resolve individual alerts
- Purge all alerts
- Empty state when no alerts

### 6. **Trees Page**
- Upload satellite/aerial images
- Image preview
- Form fields:
  - Farmer ID
  - County
  - Land acreage
  - Location
  - Notes
- Analysis results:
  - Tree count
  - Tree density
  - Canopy coverage
  - Health status
  - Species guess
  - Confidence score
  - Observations & recommendations
- Analysis history
- Usage tracking (remaining analyses)

### 7. **Settings Page**
- User profile information
- Email and name
- API key management
- Notification preferences:
  - Email notifications toggle
  - SMS notifications toggle
  - System notifications toggle
- Profile update form
- Logout button

### 8. **Auth Pages**
- Login page
- Registration page
- Form validation
- Error messages
- Toggle between login/register

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- npm or yarn

### Installation

```bash
# Clone repository
git clone https://github.com/Kizito24/weatherops-frontend.git
cd weatherops-frontend

# Install dependencies
npm install

# Copy environment file
cp .env.example .env

# Update .env with your values:
# VITE_API_BASE_URL=https://your-backend-api.com/api/v1
```

### Development

```bash
# Start dev server (runs on port 3000)
npm run dev

# Open http://localhost:3000 in browser
```

### Building

```bash
# Build for production
npm run build

# Preview production build
npm run preview
```

### Testing

```bash
# Run E2E tests
npm run test:e2e

# Run tests in UI mode
npm run test:e2e:ui

# Debug tests
npm run test:e2e:debug

# Run tests with browser visible
npm run test:e2e:headed
```

---

## 🔗 API Integration

The frontend communicates with the backend API at `/api/v1`:

### Endpoints Used

**Authentication:**
- `POST /auth/register` - Register new user
- `POST /auth/login` - Login and get JWT token
- `POST /auth/logout` - Logout user

**Locations:**
- `GET /locations` - List all locations
- `POST /locations` - Create location
- `PUT /locations/{id}` - Update location
- `DELETE /locations/{id}` - Delete location

**Rules:**
- `GET /rules/location/{locationId}` - Get rules for location
- `POST /rules` - Create rule
- `PUT /rules/{id}` - Update rule
- `PUT /rules/{id}/toggle` - Toggle rule active status
- `DELETE /rules/{id}` - Delete rule

**Alerts:**
- `GET /alerts` - Get alerts (with filtering)
- `GET /alerts/location/{locationId}` - Get location alerts
- `POST /alerts/{id}/resolve` - Resolve alert
- `GET /alerts/count/critical` - Get high severity count
- `GET /alerts/count/by-severity` - Get severity breakdown

**Weather:**
- `GET /weather` - Get weather data (current + forecast + hourly + AI summary)
- `GET /weather/usage` - Get API usage statistics

**Trees:**
- `POST /trees/analyze` - Upload and analyze image
- `GET /trees/analyses` - List previous analyses
- `GET /trees/usage` - Get tree analysis usage

**Preferences:**
- `GET /preferences` - Get user preferences
- `PUT /preferences` - Update preferences

**Health:**
- `GET /health` - API health check

---

## 🎨 Design System

### Color Scheme
- **Primary:** Indigo (`indigo-600`, `indigo-400`)
- **Success:** Emerald (`emerald-500`)
- **Warning:** Amber (`amber-500`)
- **Danger:** Rose/Red (`rose-500`)
- **Background:** Slate (`slate-950` dark, `#F9FAFB` light)

### Component Library
- **Buttons** - Primary, secondary, ghost variants
- **Inputs** - Text, email, password, number
- **Selects** - Dropdown menus
- **Cards** - Data display containers
- **Modals** - Forms and confirmations
- **Badges** - Status indicators
- **Skeletons** - Loading placeholders
- **Tables** - Data lists
- **Toast** - Notifications

### Icons
From Lucide React:
- Weather: Cloud, CloudRain, Sun, Wind, Droplets
- Status: AlertTriangle, CheckCircle, Info, AlertCircle
- Navigation: MapPin, Sliders, Bell, Settings, LayoutDashboard
- Actions: Plus, Trash2, Edit2, X, RefreshCw
- Data: TrendingUp, TrendingDown, Activity, ZapOff

---

## 📊 State Management

Uses React Hooks for state:
- `useState` - Local component state
- `useEffect` - Side effects and data fetching
- Context/Props for app-level state
- No external state management library (kept simple)

Key state patterns:
- Form state (local modals)
- Loading states (isLoading, isSubmitting)
- Error handling (errorMessage, validationErrors)
- Data caching (locations, rules, alerts)

---

## 🔐 Security Features

- **JWT Authentication** - Secure token-based auth
- **Password Encryption** - Backend bcrypt hashing
- **HTTPS/TLS** - All communications encrypted
- **CORS Protection** - Cross-origin security
- **Input Validation** - Client and server-side
- **Error Boundaries** - Graceful error handling
- **XSS Prevention** - React escapes by default

---

## 📈 Performance Features

- **Code Splitting** - Vite bundling optimization
- **Lazy Loading** - Route-based code splitting
- **Image Optimization** - Efficient icon SVGs
- **Caching** - Browser cache headers
- **Debouncing** - Form input validation
- **Memoization** - React.memo for components
- **Skeleton Loading** - Better perceived performance

---

## 🌐 Deployment

### Vercel (Current)

```bash
# One-click deploy from GitHub
# Auto-deploys on push to main
```

Environment variables needed on Vercel:
```
VITE_API_BASE_URL=https://your-backend-api.com/api/v1
```

### Other Platforms

```bash
# Build
npm run build

# Output in ./dist folder
# Deploy dist folder to any static host
```

---

## 📚 Documentation

- **[USE_CASES.md](USE_CASES.md)** - Real-world scenarios and ROI
- **[Backend README](https://github.com/Kizito24/weatherops-backend)** - API documentation
- **[API Docs](https://your-api.onrender.com/docs)** - Interactive Swagger UI

---

## 🤝 Contributing

### Development Workflow
1. Create feature branch: `git checkout -b feature/your-feature`
2. Make changes and test locally
3. Commit with clear message: `git commit -m "Add feature"`
4. Push and create Pull Request

### Code Standards
- TypeScript for type safety
- Tailwind CSS for styling
- Functional components with hooks
- Meaningful variable names
- Comments for complex logic

### Testing
Always test:
- Form validation
- Error states
- Empty states
- Loading states
- API integration
- Dark mode toggle

---

## 🐛 Troubleshooting

### API Connection Issues
- Check `VITE_API_BASE_URL` in `.env`
- Ensure backend is running
- Check CORS configuration
- Verify JWT token

### Dark Mode Not Working
- Check localStorage for `weatherops_theme`
- Clear browser cache
- Verify Tailwind dark mode in CSS

### Weather Data Not Loading
- Check Weather-AI API key
- Verify location coordinates
- Check API usage quota
- Review backend logs

### Alerts Not Triggering
- Verify rules are created and active
- Check Celery worker is running
- Review backend alert service logs
- Confirm weather data is being fetched

---

## 📄 License

MIT - See LICENSE file

---

## 🔗 Quick Links

### Live
- 🌐 **Frontend:** https://weatherops-frontend.vercel.app
- 🔌 **Backend API:** https://weatherops-backend-[id].onrender.com
- 📖 **API Docs:** https://weatherops-backend-[id].onrender.com/docs

### Repositories
- 📦 **Frontend:** https://github.com/Kizito24/weatherops-frontend
- 📦 **Backend:** https://github.com/Kizito24/weatherops-backend

### Resources
- 📖 **Use Cases:** [USE_CASES.md](USE_CASES.md)
- 📖 **Weather-AI:** https://weather-ai.co/docs
- 📖 **React Docs:** https://react.dev
- 📖 **Tailwind CSS:** https://tailwindcss.com

### Contact
- 💌 **Email:** donate.mydonation@gmail.com
- 🐙 **GitHub:** [@Kizito24](https://github.com/Kizito24)

---

## 🎉 Get Started Today

1. **Visit:** https://weatherops-frontend.vercel.app
2. **Register:** Create your account
3. **Add Location:** Set first monitoring location
4. **Create Rule:** Define alert condition
5. **Receive Alerts:** Get automatic weather notifications

**That's it. Weather intelligence is now automated for your business.**

---

**WeatherOps: Because weather shouldn't surprise you.** 🌤️
