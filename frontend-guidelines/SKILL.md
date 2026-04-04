---
name: frontend-guidelines
description: Frontend development patterns, architecture, and best practices for React, TypeScript, state management, performance optimization, and UI development
---

# Frontend Development Guidelines

Comprehensive frontend development patterns, architecture, and best practices.

## Architecture Principles

**Component-Based Architecture**
- Build UI as composition of reusable components
- Single Responsibility Principle for components
- Clear separation of concerns (UI, logic, state)
- Component-driven development (build in isolation)

**State Management Strategy**
- **Local state**: Component-level UI state (useState, useReducer)
- **Shared state**: Cross-component state (Context, Zustand)
- **Server state**: Backend data (React Query, SWR)
- **URL state**: Navigation state (search params, routes)

**Directory Structure (Feature-Based)**
```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types/
│   ├── products/
│   └── checkout/
├── shared/
│   ├── components/
│   ├── hooks/
│   ├── utils/
│   └── types/
├── layouts/
├── pages/
└── config/
```

## Code Quality Standards

**TypeScript Usage**
```typescript
// Define prop types
interface ButtonProps {
  variant: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
  children: React.ReactNode;
  disabled?: boolean;
}

// Use proper types for hooks
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Item[]>([]);

// Type API responses
interface ApiResponse<T> {
  data: T;
  error?: string;
  status: number;
}

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

**Naming Conventions**
- **Components**: PascalCase (UserProfile, ProductCard)
- **Files**: kebab-case for files (user-profile.tsx)
- **Functions**: camelCase (handleSubmit, fetchUsers)
- **Constants**: UPPER_SNAKE_CASE (API_BASE_URL)
- **CSS classes**: kebab-case (user-profile, product-card)
- **Custom hooks**: usePrefixed (useAuth, useFetchData)

**Code Organization**
```typescript
// Component structure order:
// 1. Imports
import { useState, useEffect } from 'react';
import { Button } from '@/components/ui';

// 2. Types/Interfaces
interface Props {
  userId: string;
}

// 3. Component
export function UserProfile({ userId }: Props) {
  // 4. Hooks (in order)
  const [user, setUser] = useState<User | null>(null);
  const { data } = useQuery('user', () => fetchUser(userId));

  // 5. Event handlers
  const handleEdit = () => {
    // ...
  };

  // 6. Effects
  useEffect(() => {
    // ...
  }, [userId]);

  // 7. Render helpers
  const renderActions = () => {
    // ...
  };

  // 8. Early returns
  if (!user) return <Loading />;

  // 9. Main render
  return (
    <div>
      {/* ... */}
    </div>
  );
}
```

## Performance Patterns

**Code Splitting**
```javascript
// Route-based splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// Component-based splitting
const HeavyChart = lazy(() => import('./components/HeavyChart'));
```

**List Virtualization**
```javascript
import { FixedSizeList } from 'react-window';

function LargeList({ items }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>
          {items[index].name}
        </div>
      )}
    </FixedSizeList>
  );
}
```

**Image Optimization**
```javascript
// Lazy loading
<img
  src="image.jpg"
  loading="lazy"
  alt="Description"
/>

// Responsive images
<picture>
  <source
    srcSet="image-large.webp"
    media="(min-width: 1024px)"
    type="image/webp"
  />
  <source
    srcSet="image-medium.webp"
    media="(min-width: 768px)"
    type="image/webp"
  />
  <img src="image-small.jpg" alt="Description" />
</picture>

// Modern formats with fallback
<img
  srcSet="image.avif, image.webp, image.jpg"
  alt="Description"
/>
```

**Debouncing & Throttling**
```javascript
import { debounce } from 'lodash-es';

function SearchInput() {
  const [query, setQuery] = useState('');

  const debouncedSearch = useMemo(
    () => debounce((value: string) => {
      performSearch(value);
    }, 300),
    []
  );

  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };

  return <input value={query} onChange={handleChange} />;
}
```

## Accessibility (A11y)

**Semantic HTML**
```javascript
// ✓ GOOD - Semantic elements
<header>
  <nav>
    <ul>
      <li><a href="/home">Home</a></li>
    </ul>
  </nav>
</header>
<main>
  <article>
    <h1>Title</h1>
    <p>Content</p>
  </article>
</main>
<footer>Footer content</footer>

// ✗ BAD - Generic divs
<div className="header">
  <div className="nav">
    <div onClick={goHome}>Home</div>
  </div>
</div>
```

**ARIA Attributes**
```javascript
// Labels and descriptions
<button aria-label="Close dialog">×</button>
<input aria-describedby="password-hint" />
<span id="password-hint">Must be 8+ characters</span>

// Live regions
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>

// Hidden content
<span className="sr-only">For screen readers only</span>
<div aria-hidden="true">Decorative icon</div>
```

**Keyboard Navigation**
```javascript
function Dialog({ onClose }) {
  const dialogRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();

      // Trap focus within dialog
      if (e.key === 'Tab') {
        const focusable = dialogRef.current?.querySelectorAll(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        // Handle tab focus trapping
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [onClose]);

  return (
    <div
      ref={dialogRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="dialog-title"
    >
      <h2 id="dialog-title">Dialog Title</h2>
      {/* Content */}
    </div>
  );
}
```

**Focus Management**
```javascript
function Modal({ isOpen, onClose }) {
  const closeButtonRef = useRef<HTMLButtonElement>(null);
  const previousActiveElement = useRef<HTMLElement>();

  useEffect(() => {
    if (isOpen) {
      previousActiveElement.current = document.activeElement as HTMLElement;
      closeButtonRef.current?.focus();
    } else {
      previousActiveElement.current?.focus();
    }
  }, [isOpen]);

  return (
    <div role="dialog">
      <button ref={closeButtonRef} onClick={onClose}>Close</button>
      {/* Content */}
    </div>
  );
}
```

## Error Handling

**Error Boundaries**
```typescript
class ErrorBoundary extends React.Component<
  { children: ReactNode; fallback?: ReactNode },
  { hasError: boolean; error?: Error }
> {
  state = { hasError: false, error: undefined };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Log to error tracking service
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <ErrorFallback error={this.state.error} />;
    }
    return this.props.children;
  }
}
```

**Async Error Handling**
```typescript
function useAsyncError() {
  const [error, setError] = useState<Error | null>(null);

  const executeAsync = async <T,>(promise: Promise<T>) => {
    try {
      setError(null);
      return await promise;
    } catch (err) {
      setError(err as Error);
      throw err;
    }
  };

  return { error, executeAsync, clearError: () => setError(null) };
}

// Usage
function DataComponent() {
  const { error, executeAsync } = useAsyncError();

  const loadData = () => {
    executeAsync(fetchData())
      .then(data => setData(data))
      .catch(err => {
        // Error already set by hook
        showNotification('Failed to load data');
      });
  };

  if (error) return <ErrorMessage error={error} />;
  return <Content />;
}
```

## Testing Strategy

**Component Tests**
```typescript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('LoginForm', () => {
  it('submits form with user credentials', async () => {
    const onSubmit = jest.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    await userEvent.type(screen.getByLabelText(/email/i), 'user@example.com');
    await userEvent.type(screen.getByLabelText(/password/i), 'password123');

    fireEvent.click(screen.getByRole('button', { name: /login/i }));

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'user@example.com',
        password: 'password123'
      });
    });
  });

  it('shows validation errors', async () => {
    render(<LoginForm onSubmit={jest.fn()} />);

    fireEvent.click(screen.getByRole('button', { name: /login/i }));

    expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
  });
});
```

**Integration Tests**
```typescript
import { render, screen } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { setupServer } from 'msw/node';
import { rest } from 'msw';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json([{ id: 1, name: 'John' }]));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test('displays user list from API', async () => {
  const queryClient = new QueryClient();

  render(
    <QueryClientProvider client={queryClient}>
      <UserList />
    </QueryClientProvider>
  );

  expect(await screen.findByText('John')).toBeInTheDocument();
});
```

## Security Best Practices

**Input Validation**
```typescript
// Sanitize user input
import DOMPurify from 'dompurify';

function SafeContent({ html }: { html: string }) {
  const sanitized = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
    ALLOWED_ATTR: ['href']
  });

  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}

// Validate form inputs
const emailSchema = z.string().email();
const passwordSchema = z.string().min(8);
```

**Authentication & Authorization**
```typescript
// Protected route component
function ProtectedRoute({ children, requiredRole }) {
  const { user, loading } = useAuth();

  if (loading) return <Loading />;
  if (!user) return <Navigate to="/login" />;
  if (requiredRole && user.role !== requiredRole) {
    return <Navigate to="/unauthorized" />;
  }

  return children;
}

// API request with auth
async function authenticatedFetch(url: string, options = {}) {
  const token = getAuthToken();

  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    }
  });
}
```

**CSRF Protection**
```typescript
// Include CSRF token in requests
const csrfToken = document.querySelector('meta[name="csrf-token"]')?.content;

fetch('/api/action', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': csrfToken,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data)
});
```

## CSS Best Practices

**CSS Modules**
```css
/* Button.module.css */
.button {
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
}

.primary {
  background: blue;
  color: white;
}
```

```typescript
import styles from './Button.module.css';

function Button({ variant = 'primary' }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      Click me
    </button>
  );
}
```

**CSS-in-JS (Styled Components)**
```typescript
import styled from 'styled-components';

const Button = styled.button<{ variant: 'primary' | 'secondary' }>`
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  background: ${props => props.variant === 'primary' ? 'blue' : 'gray'};
  color: white;

  &:hover {
    opacity: 0.9;
  }
`;
```

**Tailwind CSS**
```typescript
import clsx from 'clsx';

function Button({ variant, className }) {
  return (
    <button
      className={clsx(
        'px-4 py-2 rounded',
        variant === 'primary' && 'bg-blue-500 text-white',
        variant === 'secondary' && 'bg-gray-500 text-white',
        className
      )}
    >
      Click me
    </button>
  );
}
```

## Quick Reference

**Component Checklist**
- [ ] TypeScript types defined
- [ ] Props validated
- [ ] Accessible (ARIA, keyboard nav)
- [ ] Responsive design
- [ ] Error states handled
- [ ] Loading states shown
- [ ] Tests written
- [ ] Performance optimized
- [ ] Security reviewed

**Performance Checklist**
- [ ] Code splitting implemented
- [ ] Images optimized
- [ ] Lists virtualized (if large)
- [ ] Expensive operations memoized
- [ ] Bundle size monitored
- [ ] Lighthouse score > 90

**Accessibility Checklist**
- [ ] Semantic HTML used
- [ ] ARIA labels added
- [ ] Keyboard navigation works
- [ ] Focus management implemented
- [ ] Color contrast sufficient
- [ ] Screen reader tested
