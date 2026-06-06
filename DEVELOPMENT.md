# WeatherOps Frontend - Development Guide

## Coding Standards & Best Practices

This guide establishes consistent coding standards across the WeatherOps frontend codebase.

## Table of Contents

1. [TypeScript Guidelines](#typescript-guidelines)
2. [React Component Guidelines](#react-component-guidelines)
3. [State Management](#state-management)
4. [API Integration](#api-integration)
5. [Error Handling](#error-handling)
6. [Styling](#styling)
7. [Performance Optimization](#performance-optimization)
8. [Code Review Checklist](#code-review-checklist)

---

## TypeScript Guidelines

### 1. Type Definitions

**✅ DO**: Define explicit types for all values

```typescript
// Good
interface Location {
  id: string;
  name: string;
  latitude: number;
  longitude: number;
  createdAt: string;
}

type WeatherMetric = 'temperature' | 'rainfall' | 'wind_speed' | 'humidity';

// Use types, not interfaces, for unions
type Alert = ActiveAlert | ResolvedAlert;

// Explicit function return types
function calculateDistance(lat1: number, lng1: number, lat2: number, lng2: number): number {
  // ...
}
```

**❌ DON'T**: Use `any` type

```typescript
// Bad - avoids type safety
const data: any = response.data;

// Good - explicit type
const data: WeatherData = response.data;
```

### 2. Strict Null Checking

```typescript
// Good - handle both cases
const user: UserProfile | null = getUser();

if (user) {
  console.log(user.email);
} else {
  console.log('No user');
}

// Good - optional chaining
const email = user?.email;

// Good - nullish coalescing
const name = user?.name ?? 'Anonymous';
```

### 3. Generic Types

```typescript
// Good - reusable generic API response
interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
}

// Usage
const locResponse: ApiResponse<Location[]> = await api.get('/locations');
const ruleResponse: ApiResponse<Rule> = await api.get('/rules/1');
```

### 4. Avoid Type Assertion Abuse

```typescript
// Bad - bypasses type checking
const value = someUnknownValue as string;

// Good - validate before using
if (typeof someUnknownValue === 'string') {
  const value = someUnknownValue;
}

// Good - use type guard
function isLocation(obj: unknown): obj is Location {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj &&
    'latitude' in obj &&
    'longitude' in obj
  );
}
```

---

## React Component Guidelines

### 1. Functional Components with Hooks

```typescript
// Good - modern React functional component
interface OverviewPageProps {
  locations: Location[];
  alerts: Alert[];
  isLoading: boolean;
}

export default function OverviewPage({
  locations,
  alerts,
  isLoading,
}: OverviewPageProps) {
  const [expandedCard, setExpandedCard] = useState<string | null>(null);

  useEffect(() => {
    // Side effect
  }, [locations]);

  return (
    <div className="space-y-6">
      {isLoading ? (
        <Skeleton />
      ) : (
        <KPICard value={locations.length} title="Total Locations" />
      )}
    </div>
  );
}

// Export with display name for debugging
OverviewPage.displayName = 'OverviewPage';
```

### 2. Component Props Patterns

```typescript
// Good - explicit props interface
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary';
  className?: string;
}

export function Button({
  label,
  onClick,
  disabled = false,
  variant = 'primary',
  className,
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={cn(
        'px-4 py-2 rounded font-semibold',
        variant === 'primary' ? 'bg-blue-600 text-white' : 'bg-gray-200',
        className
      )}
    >
      {label}
    </button>
  );
}
```

### 3. Hooks Best Practices

```typescript
// Good - custom hook for reusable logic
function useWeatherData(locationId: string) {
  const [data, setData] = useState<WeatherData | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let isMounted = true;

    const fetchData = async () => {
      setLoading(true);
      try {
        const result = await weatherApi.getCurrent(locationId);
        if (isMounted) {
          setData(result);
          setError(null);
        }
      } catch (err) {
        if (isMounted) {
          setError(err instanceof Error ? err : new Error('Unknown error'));
          setData(null);
        }
      } finally {
        if (isMounted) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      isMounted = false;
    };
  }, [locationId]);

  return { data, loading, error };
}

// Usage
export function WeatherDisplay({ locationId }: { locationId: string }) {
  const { data, loading, error } = useWeatherData(locationId);

  if (loading) return <Skeleton />;
  if (error) return <ErrorState message={error.message} />;
  if (!data) return <EmptyState />;

  return <WeatherCard data={data} />;
}
```

### 4. Event Handler Patterns

```typescript
// Good - explicit handler types
function LocationForm() {
  const [name, setName] = useState('');

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    // Submit logic
  };

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setName(e.target.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={handleInputChange}
        placeholder="Location name"
      />
      <button type="submit">Add Location</button>
    </form>
  );
}
```

### 5. Conditional Rendering

```typescript
// Good - clear and readable
function AlertCard({ alert }: { alert: Alert }) {
  if (alert.status === 'resolved') {
    return <ResolvedAlertCard alert={alert} />;
  }

  return <ActiveAlertCard alert={alert} />;
}

// Good - compound pattern for complex UI
function WeatherDisplay({ data, loading, error }: Props) {
  if (loading) return <WeatherSkeleton />;
  if (error) return <WeatherError />;
  if (!data) return <EmptyWeatherState />;

  return (
    <div className="space-y-4">
      <CurrentWeather data={data.current} />
      <ForecastSection data={data.daily} />
      <HourlyChart data={data.hourly} />
    </div>
  );
}

// Bad - ternary hell
return isLoading ? (
  <Skeleton />
) : error ? (
  <Error />
) : !data ? (
  <Empty />
) : (
  <Display data={data} />
);
```

---

## State Management

### 1. useState Pattern

```typescript
// Good - separate concerns
function LocationForm() {
  const [name, setName] = useState('');
  const [latitude, setLatitude] = useState('');
  const [longitude, setLongitude] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError(null);
    setIsSubmitting(true);

    try {
      await onAddLocation(name, parseFloat(latitude), parseFloat(longitude));
      setName('');
      setLatitude('');
      setLongitude('');
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Failed to add location');
    } finally {
      setIsSubmitting(false);
    }
  };

  // Render form...
}
```

### 2. useEffect Dependencies

```typescript
// Good - explicit dependencies
useEffect(() => {
  if (!userId) return;

  let isMounted = true;

  const fetchUser = async () => {
    try {
      const data = await userApi.get(userId);
      if (isMounted) {
        setUser(data);
      }
    } catch (err) {
      if (isMounted) {
        setError(err);
      }
    }
  };

  fetchUser();

  return () => {
    isMounted = false;
  };
}, [userId]);

// Bad - missing dependencies (will cause bugs)
useEffect(() => {
  loadData(userId, pageSize); // pageSize is missing from deps!
}, [userId]);
```

### 3. State Initialization

```typescript
// Good - lazy initialization for expensive operations
const [user, setUser] = useState<UserProfile | null>(() => {
  const stored = localStorage.getItem('user');
  return stored ? JSON.parse(stored) : null;
});

// Good - callback function for complex initial state
const [filters, setFilters] = useState(() => ({
  severity: 'HIGH' as const,
  dateRange: getLastWeek(),
  locationIds: [] as string[],
}));
```

---

## API Integration

### 1. API Client Pattern

```typescript
// Good - type-safe API client
class LocationApi {
  private baseUrl = import.meta.env.VITE_API_BASE_URL;
  private timeout = import.meta.env.VITE_API_TIMEOUT;

  async list(): Promise<Location[]> {
    const response = await axios.get<Location[]>(
      `${this.baseUrl}/locations`,
      { timeout: this.timeout }
    );
    return response.data;
  }

  async create(
    name: string,
    latitude: number,
    longitude: number
  ): Promise<Location> {
    const response = await axios.post<Location>(
      `${this.baseUrl}/locations`,
      { name, latitude, longitude },
      { timeout: this.timeout }
    );
    return response.data;
  }

  async update(
    id: string,
    name: string,
    latitude: number,
    longitude: number
  ): Promise<Location> {
    const response = await axios.put<Location>(
      `${this.baseUrl}/locations/${id}`,
      { name, latitude, longitude },
      { timeout: this.timeout }
    );
    return response.data;
  }

  async delete(id: string): Promise<void> {
    await axios.delete(`${this.baseUrl}/locations/${id}`, {
      timeout: this.timeout,
    });
  }
}

export const locationsApi = new LocationApi();
```

### 2. API Error Handling

```typescript
// Good - custom error class
class ApiError extends Error {
  constructor(
    public statusCode: number,
    public data: any,
    message: string
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

// Good - error interceptor
axios.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response) {
      // Server responded with error status
      const { status, data } = error.response;
      const message = data?.detail || data?.message || 'Request failed';

      if (status === 401) {
        // Handle unauthorized
        tokenManager.clear();
        window.location.href = '/login';
      }

      throw new ApiError(status, data, message);
    }

    if (error.request) {
      // Request made but no response
      throw new Error('No response from server');
    }

    // Request setup error
    throw error;
  }
);
```

### 3. Loading States

```typescript
// Good - explicit loading state management
async function handleCreateLocation(name: string, lat: number, lng: number) {
  try {
    setIsLoading(true);
    setError(null);

    const newLocation = await locationsApi.create(name, lat, lng);

    showToast('Location added successfully', 'LOW');
    await loadPlatformData(); // Reload all data
    resetForm();

    return newLocation;
  } catch (err) {
    const message =
      err instanceof Error ? err.message : 'Failed to add location';
    setError(message);
    showToast(message, 'MEDIUM');
    throw err;
  } finally {
    setIsLoading(false);
  }
}
```

---

## Error Handling

### 1. Error Boundaries

```typescript
// Good - comprehensive error boundary
interface ErrorBoundaryProps {
  children: React.ReactNode;
}

interface ErrorBoundaryState {
  error: Error | null;
  errorInfo: React.ErrorInfo | null;
}

export class ErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error: Error) {
    return { error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log error details
    console.error('Error caught:', error, errorInfo);

    this.setState({
      error,
      errorInfo,
    });

    // Send to error tracking service
    if (import.meta.env.VITE_APP_ENV === 'production') {
      // Sentry.captureException(error);
    }
  }

  render() {
    if (this.state.error) {
      return (
        <div className="min-h-screen flex items-center justify-center bg-red-50">
          <div className="max-w-md w-full space-y-4">
            <h1 className="text-2xl font-bold text-red-800">
              Something went wrong
            </h1>
            <p className="text-red-700">{this.state.error.toString()}</p>
            <button
              onClick={() => window.location.reload()}
              className="w-full px-4 py-2 bg-red-600 text-white rounded"
            >
              Reload Page
            </button>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 2. Try-Catch Patterns

```typescript
// Good - specific error handling
async function loadWeatherData(locationId: string) {
  try {
    const data = await weatherApi.getCurrent(locationId);
    setWeatherData(data);
  } catch (err) {
    if (err instanceof ApiError) {
      if (err.statusCode === 404) {
        showToast('Location not found', 'MEDIUM');
      } else if (err.statusCode === 429) {
        showToast('Rate limit exceeded - try again later', 'HIGH');
      } else {
        showToast(err.message, 'MEDIUM');
      }
    } else if (err instanceof Error) {
      showToast('Network error - check your connection', 'MEDIUM');
    }
  }
}
```

---

## Styling

### 1. Tailwind CSS Classes

```typescript
// Good - organized Tailwind classes
function Card({ children }: { children: React.ReactNode }) {
  return (
    <div
      className={`
        p-6
        rounded-lg
        border border-slate-200
        dark:border-slate-800
        bg-white
        dark:bg-slate-900
        shadow-sm
        hover:shadow-md
        transition-shadow
        duration-200
      `}
    >
      {children}
    </div>
  );
}

// Or use cn utility (classnames)
import cn from 'classnames';

function Button({ variant, className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        'px-4 py-2 rounded-lg font-medium transition-colors',
        {
          'bg-blue-600 text-white hover:bg-blue-700': variant === 'primary',
          'bg-gray-200 text-gray-900 hover:bg-gray-300': variant === 'secondary',
        },
        className
      )}
      {...props}
    />
  );
}
```

### 2. Dark Mode Support

```typescript
// Good - dark mode classes
<div
  className={`
    text-slate-900
    dark:text-slate-100
    bg-white
    dark:bg-slate-900
    border border-slate-200
    dark:border-slate-800
  `}
>
  Content
</div>
```

### 3. Responsive Design

```typescript
// Good - mobile-first responsive design
<div
  className={`
    grid
    grid-cols-1
    sm:grid-cols-2
    lg:grid-cols-3
    xl:grid-cols-4
    gap-4
  `}
>
  {items.map((item) => (
    <Card key={item.id}>{item.name}</Card>
  ))}
</div>
```

---

## Performance Optimization

### 1. Memoization

```typescript
// Good - memoize expensive components
interface LocationCardProps {
  location: Location;
  onEdit: (id: string) => void;
}

const LocationCard = React.memo(
  ({ location, onEdit }: LocationCardProps) => (
    <Card>
      <h3>{location.name}</h3>
      <button onClick={() => onEdit(location.id)}>Edit</button>
    </Card>
  ),
  (prev, next) => {
    // Custom comparison if needed
    return prev.location.id === next.location.id;
  }
);
```

### 2. useCallback for Event Handlers

```typescript
// Good - memoize callbacks passed to children
function LocationsPage({ locations }: { locations: Location[] }) {
  const handleEdit = useCallback((id: string) => {
    // Edit logic
  }, []);

  const handleDelete = useCallback(async (id: string) => {
    // Delete logic
  }, []);

  return (
    <>
      {locations.map((loc) => (
        <LocationCard
          key={loc.id}
          location={loc}
          onEdit={handleEdit}
          onDelete={handleDelete}
        />
      ))}
    </>
  );
}
```

### 3. useMemo for Expensive Computations

```typescript
// Good - memoize expensive calculations
function AlertsList({ alerts }: { alerts: Alert[] }) {
  const groupedAlerts = useMemo(() => {
    return alerts.reduce(
      (acc, alert) => {
        const key = alert.severity;
        if (!acc[key]) acc[key] = [];
        acc[key].push(alert);
        return acc;
      },
      {} as Record<string, Alert[]>
    );
  }, [alerts]);

  return (
    <>
      {Object.entries(groupedAlerts).map(([severity, items]) => (
        <div key={severity}>
          <h3>{severity}</h3>
          {items.map((alert) => (
            <AlertCard key={alert.id} alert={alert} />
          ))}
        </div>
      ))}
    </>
  );
}
```

### 4. Lazy Loading & Code Splitting

```typescript
// Good - lazy load pages
const OverviewPage = React.lazy(() =>
  import('./components/OverviewPage').then((m) => ({
    default: m.OverviewPage,
  }))
);

const WeatherPage = React.lazy(() =>
  import('./components/WeatherPage').then((m) => ({
    default: m.WeatherPage,
  }))
);

// In App.tsx
<Suspense fallback={<PageSkeleton />}>
  {activePage === 'overview' && <OverviewPage {...props} />}
  {activePage === 'weather' && <WeatherPage {...props} />}
</Suspense>;
```

---

## Code Review Checklist

Before submitting a pull request, verify:

### Types & TypeScript
- [ ] All `any` types are justified
- [ ] Functions have explicit return types
- [ ] Props interfaces are defined
- [ ] No TypeScript errors on `npm run lint`

### React & Components
- [ ] Components use functional syntax with hooks
- [ ] Props are properly typed and documented
- [ ] Conditional rendering is clear and readable
- [ ] No unnecessary re-renders (use memo if needed)
- [ ] Error states are handled

### State Management
- [ ] useState dependencies are correct
- [ ] useEffect has proper cleanup functions
- [ ] No stale closures in callbacks
- [ ] Global state updates are proper

### API Integration
- [ ] All API calls are wrapped in try-catch
- [ ] Loading states are shown
- [ ] Error messages are user-friendly
- [ ] API tokens are properly handled

### Error Handling
- [ ] Error boundaries are in place
- [ ] User receives feedback on failures
- [ ] Network errors are handled gracefully
- [ ] No console errors in dev tools

### Styling
- [ ] Dark mode support is implemented
- [ ] Responsive design is tested
- [ ] No hardcoded colors (use Tailwind)
- [ ] Proper spacing and alignment

### Performance
- [ ] Large lists use pagination or virtualization
- [ ] No unnecessary re-renders
- [ ] Images are optimized
- [ ] Bundle size impact is minimal

### Testing
- [ ] Manual testing in browser
- [ ] E2E tests pass: `npm run test:e2e`
- [ ] Different browsers tested
- [ ] Mobile responsive tested

### Code Quality
- [ ] Code follows established patterns
- [ ] Variable names are descriptive
- [ ] Functions are not too long
- [ ] Comments explain "why", not "what"
- [ ] No console logs left in production code

### Git
- [ ] Commit messages are descriptive
- [ ] Branch is up-to-date with main
- [ ] No merge conflicts
- [ ] No accidental files committed

---

**Last Updated**: June 2026  
**Version**: 1.0.0

**Next Steps**: Read [TESTING.md](./TESTING.md) for testing strategies and [DEPLOYMENT.md](./DEPLOYMENT.md) for deployment procedures.
