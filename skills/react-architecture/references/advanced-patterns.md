# Advanced React Patterns

## Compound Components Pattern

Gives users full control over rendering while maintaining internal state.

```tsx
const TabsContext = React.createContext<{
  active: string;
  setActive: (id: string) => void;
} | null>(null);

function Tabs({ defaultValue, children }: { defaultValue: string; children: React.ReactNode }) {
  const [active, setActive] = React.useState(defaultValue);
  return (
    <TabsContext.Provider value={{ active, setActive }}>
      {children}
    </TabsContext.Provider>
  );
}

function TabList({ children }: { children: React.ReactNode }) {
  return <div role="tablist" className="flex gap-1">{children}</div>;
}

function Tab({ value, children }: { value: string; children: React.ReactNode }) {
  const ctx = React.useContext(TabsContext);
  if (!ctx) throw new Error('Tab must be inside Tabs');
  return (
    <button
      role="tab"
      aria-selected={ctx.active === value}
      onClick={() => ctx.setActive(value)}
      className={ctx.active === value ? 'active' : ''}
    >
      {children}
    </button>
  );
}

function TabPanel({ value, children }: { value: string; children: React.ReactNode }) {
  const ctx = React.useContext(TabsContext);
  if (!ctx) throw new Error('TabPanel must be inside Tabs');
  if (ctx.active !== value) return null;
  return <div role="tabpanel">{children}</div>;
}

Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;
```

## Render Props with Hooks

When you need maximum flexibility in how data is rendered:

```tsx
function useSearchableList<T>(items: T[], searchKey: keyof T) {
  const [query, setQuery] = React.useState('');
  const filtered = React.useMemo(() =>
    items.filter(item =>
      String(item[searchKey]).toLowerCase().includes(query.toLowerCase())
    ),
    [items, query, searchKey]
  );
  return { query, setQuery, filtered, count: filtered.length };
}
```

## Generic Data Table Component

```tsx
type Column<T> = {
  key: keyof T;
  label: string;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
  sortable?: boolean;
};

function DataTable<T extends { id: string }>({
  data,
  columns,
  onRowClick,
}: {
  data: T[];
  columns: Column<T>[];
  onRowClick?: (row: T) => void;
}) {
  const [sortKey, setSortKey] = React.useState<keyof T | null>(null);
  const [sortDir, setSortDir] = React.useState<'asc' | 'desc'>('asc');

  const sorted = React.useMemo(() => {
    if (!sortKey) return data;
    return [...data].sort((a, b) => {
      const aVal = a[sortKey], bVal = b[sortKey];
      const cmp = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
      return sortDir === 'asc' ? cmp : -cmp;
    });
  }, [data, sortKey, sortDir]);

  return (
    <table>
      <thead>
        <tr>
          {columns.map(col => (
            <th
              key={String(col.key)}
              onClick={() => {
                if (!col.sortable) return;
                if (sortKey === col.key) setSortDir(d => d === 'asc' ? 'desc' : 'asc');
                else { setSortKey(col.key); setSortDir('asc'); }
              }}
              style={{ cursor: col.sortable ? 'pointer' : 'default' }}
            >
              {col.label}
              {sortKey === col.key && (sortDir === 'asc' ? ' ↑' : ' ↓')}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {sorted.map(row => (
          <tr key={row.id} onClick={() => onRowClick?.(row)}>
            {columns.map(col => (
              <td key={String(col.key)}>
                {col.render ? col.render(row[col.key], row) : String(row[col.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

## Error Boundary Pattern

```tsx
class ErrorBoundary extends React.Component<
  { children: React.ReactNode; fallback: React.ReactNode },
  { hasError: boolean; error: Error | null }
> {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    console.error('Error boundary caught:', error, info);
    // Send to error tracking service
  }

  render() {
    if (this.state.hasError) return this.props.fallback;
    return this.props.children;
  }
}

// Usage with Suspense
function FeatureWrapper({ children }: { children: React.ReactNode }) {
  return (
    <ErrorBoundary fallback={<ErrorFallback />}>
      <Suspense fallback={<FeatureSkeleton />}>
        {children}
      </Suspense>
    </ErrorBoundary>
  );
}
```

## Custom Hook Patterns

### useLocalStorage
```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = React.useState<T>(() => {
    try {
      const stored = window.localStorage.getItem(key);
      return stored ? JSON.parse(stored) : initialValue;
    } catch { return initialValue; }
  });

  React.useEffect(() => {
    window.localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}
```

### useDebounce
```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = React.useState(value);
  React.useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}
```

### useMediaQuery
```tsx
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = React.useState(false);
  React.useEffect(() => {
    const mql = window.matchMedia(query);
    setMatches(mql.matches);
    const handler = (e: MediaQueryListEvent) => setMatches(e.matches);
    mql.addEventListener('change', handler);
    return () => mql.removeEventListener('change', handler);
  }, [query]);
  return matches;
}

// Usage
const isMobile = useMediaQuery('(max-width: 768px)');
```

## Form Pattern (React Hook Form + Zod)

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email'),
  role: z.enum(['admin', 'member']),
});
type FormData = z.infer<typeof schema>;

function CreateUserForm({ onSubmit }: { onSubmit: (data: FormData) => void }) {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormData>({
    resolver: zodResolver(schema),
    defaultValues: { role: 'member' },
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
      {errors.name && <span>{errors.name.message}</span>}
      {/* ... */}
      <button type="submit" disabled={isSubmitting}>Create</button>
    </form>
  );
}
```
