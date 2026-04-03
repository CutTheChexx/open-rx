---
name: testing-ops
description: >
  Testing strategy and patterns for modern web applications. Master Vitest configuration, unit testing with mocks and snapshots, React Testing Library for component behavior verification, integration testing with MSW, Playwright for E2E automation across browsers, Next.js App Router testing including Server Components and Server Actions, and strategic test patterns for forms, tables, auth flows, and real-time features. TDD approach with coverage thresholds, CI/CD pipeline integration, and avoiding common testing pitfalls. Covers test strategy pyramid, vitest, playwright, unit test, e2e, integration test, coverage metrics, testing best practices.
metadata:
  version: "1.0.0"
  author: "CutTheChexx"
  sources: "Vitest docs, Playwright docs, Testing Library, Kent C. Dodds testing philosophy"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

## Testing Philosophy

The testing trophy, not pyramid: **Static > Unit > Integration > E2E**

Static analysis catches the most issues per effort (TypeScript, ESLint). Unit tests verify isolated functions. Integration tests verify feature flows within the app. E2E tests verify real user workflows across the browser.

Test behavior, not implementation. When you test implementation details, your tests break on refactors even when the behavior is correct. Use React Testing Library's "query as users do" approach. Aim for confidence, not coverage numbers—80% of the right tests beat 100% of the wrong ones.

## Vitest Setup

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'dist/', '**/*.d.ts'],
      statements: 80,
      branches: 75,
      functions: 80,
      lines: 80,
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@utils': path.resolve(__dirname, './src/utils'),
    },
  },
})
```

Configure `vitest.setup.ts` for cleanup and global utilities:

```typescript
// vitest.setup.ts
import { afterEach, vi } from 'vitest'
import { cleanup } from '@testing-library/react'

afterEach(() => {
  cleanup()
  vi.clearAllMocks()
})

global.matchMedia = global.matchMedia || function() {
  return {
    addListener: vi.fn(),
    removeListener: vi.fn(),
  }
}
```

For monorepos, use workspace config:

```typescript
// vitest.workspace.ts
export default [
  { test: { include: ['packages/ui/**/*.test.ts'] } },
  { test: { include: ['packages/api/**/*.test.ts'] } },
  { test: { include: ['packages/web/**/*.test.ts'] } },
]
```

## Unit Testing Patterns

Test pure functions, utilities, hooks, and logic in isolation.

```typescript
// sum.test.ts
import { describe, it, expect } from 'vitest'
import { sum } from './sum'

describe('sum', () => {
  it('adds two positive numbers', () => {
    expect(sum(2, 3)).toBe(5)
  })

  it('handles negative numbers', () => {
    expect(sum(-1, 5)).toBe(4)
  })
})
```

Mocking with Vitest:

```typescript
// api.test.ts
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest'
import { fetchUser } from './api'

vi.mock('axios')

describe('fetchUser', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('fetches user data', async () => {
    const mockData = { id: 1, name: 'John' }
    vi.mocked(axios.get).mockResolvedValueOnce({ data: mockData })

    const result = await fetchUser(1)
    expect(result).toEqual(mockData)
    expect(axios.get).toHaveBeenCalledWith('/api/users/1')
  })
})
```

Spying on module functions:

```typescript
// hook.test.ts
import { describe, it, expect, vi } from 'vitest'
import { useLocalStorage } from './useLocalStorage'

describe('useLocalStorage', () => {
  it('syncs value to localStorage', () => {
    const spy = vi.spyOn(Storage.prototype, 'setItem')
    const { result } = renderHook(() => useLocalStorage('key', 'value'))

    expect(spy).toHaveBeenCalledWith('key', 'value')
  })
})
```

Snapshot testing for complex outputs:

```typescript
// component.test.ts
it('renders form correctly', () => {
  const { container } = render(<ContactForm />)
  expect(container).toMatchSnapshot()
})
```

## React Testing Library

Test components as users interact with them, not implementation details.

```typescript
// Button.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Button } from './Button'

describe('Button', () => {
  it('calls onClick handler on click', async () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>Click me</Button>)

    await userEvent.click(screen.getByRole('button', { name: /click me/i }))
    expect(handleClick).toHaveBeenCalled()
  })
})
```

Testing forms with validation:

```typescript
// LoginForm.test.tsx
describe('LoginForm', () => {
  it('validates email before submit', async () => {
    const user = userEvent.setup()
    render(<LoginForm onSubmit={vi.fn()} />)

    const submitButton = screen.getByRole('button', { name: /submit/i })
    await user.click(submitButton)

    expect(screen.getByText(/invalid email/i)).toBeInTheDocument()
  })

  it('submits form with valid data', async () => {
    const user = userEvent.setup()
    const handleSubmit = vi.fn()
    render(<LoginForm onSubmit={handleSubmit} />)

    await user.type(screen.getByLabelText(/email/i), 'test@example.com')
    await user.type(screen.getByLabelText(/password/i), 'password123')
    await user.click(screen.getByRole('button', { name: /submit/i }))

    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123',
    })
  })
})
```

Testing async components with waitFor:

```typescript
describe('UserProfile', () => {
  it('loads and displays user data', async () => {
    vi.mocked(fetchUser).mockResolvedValueOnce({ id: 1, name: 'Alice' })
    render(<UserProfile userId={1} />)

    expect(screen.getByText(/loading/i)).toBeInTheDocument()

    await waitFor(() => {
      expect(screen.getByText('Alice')).toBeInTheDocument()
    })
  })
})
```

Testing with providers (QueryClient, auth context):

```typescript
// queries.test.ts
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const createTestQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: { retry: false },
  },
})

const wrapper = ({ children }: { children: React.ReactNode }) => (
  <QueryClientProvider client={createTestQueryClient()}>
    {children}
  </QueryClientProvider>
)

describe('useUser', () => {
  it('fetches user data', async () => {
    const { result } = renderHook(() => useUser(), { wrapper })

    await waitFor(() => {
      expect(result.current.data).toBeDefined()
    })
  })
})
```

## Integration Testing

Test features end-to-end within your app using Mock Service Worker (MSW) for API mocking.

```typescript
// server.ts
import { setupServer } from 'msw/node'
import { http, HttpResponse } from 'msw'

export const server = setupServer(
  http.get('/api/users/:id', async ({ params }) => {
    return HttpResponse.json({ id: params.id, name: 'John Doe' })
  }),
  http.post('/api/users', async ({ request }) => {
    const body = await request.json()
    return HttpResponse.json({ id: 1, ...body }, { status: 201 })
  }),
)

// setup-tests.ts
beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())
```

Testing data flows:

```typescript
// Feature.test.tsx
describe('User creation flow', () => {
  it('creates user and updates list', async () => {
    const user = userEvent.setup()
    render(<App />)

    await user.type(screen.getByLabelText(/name/i), 'Alice')
    await user.click(screen.getByRole('button', { name: /create/i }))

    await waitFor(() => {
      expect(screen.getByText('Alice')).toBeInTheDocument()
    })
  })
})
```

## Playwright E2E Testing

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  webServer: {
    command: 'npm run dev',
    reuseExistingServer: !process.env.CI,
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
})
```

Page Object Model:

```typescript
// pages/login.page.ts
export class LoginPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/login')
  }

  async login(email: string, password: string) {
    await this.page.fill('input[type="email"]', email)
    await this.page.fill('input[type="password"]', password)
    await this.page.click('button[type="submit"]')
  }

  async getErrorMessage() {
    return this.page.textContent('.error-message')
  }
}

// e2e/auth.spec.ts
import { test, expect } from '@playwright/test'
import { LoginPage } from '../pages/login.page'

test.describe('Authentication', () => {
  test('logs in with valid credentials', async ({ page }) => {
    const loginPage = new LoginPage(page)
    await loginPage.goto()
    await loginPage.login('user@example.com', 'password')
    await expect(page).toHaveURL('/dashboard')
  })
})
```

## Testing Next.js App Router

Testing Server Components (run in Node):

```typescript
// app/components/UserCard.server.test.tsx
import { render } from '@testing-library/react'
import { UserCard } from './UserCard'
import * as userModule from '@/lib/user'

vi.mock('@/lib/user')

describe('UserCard (Server Component)', () => {
  it('renders user name', async () => {
    vi.mocked(userModule.getUser).mockResolvedValueOnce({
      id: 1,
      name: 'Alice',
    })

    const { container } = render(await UserCard({ id: 1 }))
    expect(container).toHaveTextContent('Alice')
  })
})
```

Testing Server Actions:

```typescript
// app/actions.test.ts
import { createUser } from './actions'

vi.mock('@/db', () => ({
  db: {
    user: {
      create: vi.fn(),
    },
  },
}))

describe('Server Actions', () => {
  it('creates user via action', async () => {
    await createUser('test@example.com')
    expect(db.user.create).toHaveBeenCalledWith({
      data: { email: 'test@example.com' },
    })
  })
})
```

## Testing Patterns by Component Type

Form testing:

```typescript
describe('DynamicForm', () => {
  it('validates required fields', async () => {
    const user = userEvent.setup()
    render(<DynamicForm fields={[{ name: 'email', required: true }]} />)
    await user.click(screen.getByRole('button', { name: /submit/i }))
    expect(screen.getByText(/required/i)).toBeInTheDocument()
  })
})
```

Modal testing:

```typescript
describe('Modal', () => {
  it('closes on confirm', async () => {
    const user = userEvent.setup()
    const onConfirm = vi.fn()
    render(<Modal onConfirm={onConfirm} />)
    await user.click(screen.getByRole('button', { name: /confirm/i }))
    expect(onConfirm).toHaveBeenCalled()
  })
})
```

## CI/CD Testing

GitHub Actions workflow:

```yaml
# .github/workflows/test.yml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit -- --run
      - run: npm run test:integration -- --run
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx playwright install
      - run: npm run build
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
```

## What NOT to Test

Don't test implementation details. Don't test that a component calls a specific hook—test the behavior it produces. Don't test library internals (React rendering, array methods). Don't aim for 100% coverage; aim for meaningful coverage that reduces production bugs.

Bad test:
```typescript
expect(useState).toHaveBeenCalled()
expect(component.instance.state.count).toBe(0)
```

Good test:
```typescript
expect(screen.getByText(/count: 0/i)).toBeInTheDocument()
```

<sub>Open RX by CutTheChexx — The Prescription.</sub>
