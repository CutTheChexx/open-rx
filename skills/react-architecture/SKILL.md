---
name: react-architecture
description: >
  Elite React architecture patterns distilled from bulletproof-react (29k+ stars),
  clean architecture repos, and production-grade patterns. Use this skill whenever
  building React components, structuring projects, creating features, setting up
  state management, writing hooks, or organizing any React/Next.js codebase.
  Also triggers on "project structure", "folder structure", "architecture",
  "component patterns", "clean code", "refactor", or any React development task
  where code quality matters.
metadata:
  version: "1.0.0"
  sources: "bulletproof-react, react-clean-architecture, React TypeScript Cheatsheet"
author: "CutTheChexx"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

# React Architecture — Bulletproof Patterns

Distilled from [bulletproof-react](https://github.com/alan2207/bulletproof-react) and
production-grade React codebases.

## Project Structure

```
src/
├── app/              # Application layer — routes, providers, root component
│   ├── routes/       # Page components / route definitions
│   ├── app.tsx       # Root component
│   └── provider.tsx  # Global providers (auth, theme, query, etc.)
├── components/       # Shared UI components (buttons, inputs, modals)
├── config/           # Environment variables, API URLs, feature flags
├── features/         # Feature modules (the core of your app)
│   └── [feature]/
│       ├── api/      # API calls & React Query hooks for this feature
│       ├── components/  # Feature-specific components
│       ├── hooks/    # Feature-specific hooks
│       ├── stores/   # Feature state (Zustand, Jotai, etc.)
│       ├── types/    # Feature TypeScript types
│       └── utils/    # Feature utility functions
├── hooks/            # Shared hooks (useDebounce, useMediaQuery, etc.)
├── lib/              # Pre-configured libraries (axios instance, supabase client)
├── stores/           # Global state stores
├── types/            # Shared TypeScript types
└── utils/            # Shared utilities
```

## The Three Laws

### 1. Feature Independence
Never import across features. If Feature A needs something from Feature B,
lift it to `components/` or `hooks/` (shared), or compose them at the `app/` layer.

### 2. Unidirectional Flow
`shared → features → app`. Code flows in ONE direction:
- `app/` can import from `features/` and shared modules
- `features/` can import from shared modules only
- Shared modules (`components/`, `hooks/`, `lib/`, `utils/`) import from nothing in the app

### 3. Colocation
Keep related code together. A feature's API calls, components, hooks, types, and
utils live in the same directory. Don't scatter them across `api/`, `components/`,
`hooks/` at the root level.

## Component Patterns

### Composition over Configuration
```tsx
// Bad — prop explosion
<Card title="..." subtitle="..." icon={...} footer={...} onClick={...} />

// Good — composition
<Card>
  <Card.Header>
    <Card.Icon icon={Star} />
    <Card.Title>Project Alpha</Card.Title>
  </Card.Header>
  <Card.Body>Content here</Card.Body>
  <Card.Footer>
    <Button onClick={handleClick}>Action</Button>
  </Card.Footer>
</Card>
```

### Container/Presentation Split
```tsx
// Container — handles data & logic
function ProjectListContainer() {
  const { data, isLoading } = useProjects();
  if (isLoading) return <Skeleton />;
  return <ProjectList projects={data} />;
}

// Presentation — pure rendering
function ProjectList({ projects }: { projects: Project[] }) {
  return (
    <div className="grid gap-4">
      {projects.map(p => <ProjectCard key={p.id} project={p} />)}
    </div>
  );
}
```

### Custom Hook Extraction
Extract logic into hooks when a component exceeds ~50 lines of logic:
```tsx
function useProjectForm(projectId?: string) {
  const [form, setForm] = useState<ProjectForm>(defaultForm);
  const mutation = useCreateProject();

  const handleSubmit = async () => {
    await mutation.mutateAsync(form);
  };

  return { form, setForm, handleSubmit, isSubmitting: mutation.isPending };
}
```

## TypeScript Patterns

```tsx
// Props with children
type CardProps = {
  variant?: 'default' | 'outlined' | 'elevated';
  className?: string;
  children: React.ReactNode;
};

// Discriminated unions for state
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

// Polymorphic components
type ButtonProps<T extends React.ElementType = 'button'> = {
  as?: T;
  variant?: 'primary' | 'secondary' | 'ghost';
} & React.ComponentPropsWithoutRef<T>;
```

## Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Files & folders | kebab-case | `project-list.tsx`, `use-auth.ts` |
| Components | PascalCase | `ProjectList`, `AuthProvider` |
| Hooks | camelCase with `use` prefix | `useAuth`, `useProjects` |
| Types/Interfaces | PascalCase | `Project`, `UserProfile` |
| Constants | SCREAMING_SNAKE | `API_BASE_URL`, `MAX_RETRIES` |
| Utils | camelCase | `formatCurrency`, `validateEmail` |

## State Management Decision Tree

```
Is the state only used in one component?
  → useState / useReducer

Is it shared between siblings?
  → Lift state to parent, or use context for deep trees

Is it server data (API responses)?
  → React Query / TanStack Query (NOT in global state)

Is it global UI state (theme, sidebar, modals)?
  → Zustand or Jotai (lightweight) over Redux

Is it form state?
  → React Hook Form + Zod validation
```

## Error Handling Pattern

```tsx
// Error boundary at feature level
<ErrorBoundary fallback={<FeatureError />}>
  <Suspense fallback={<FeatureSkeleton />}>
    <Feature />
  </Suspense>
</ErrorBoundary>
```

## For detailed patterns, read `references/advanced-patterns.md`

<sub>Open RX by CutTheChexx — The Prescription.</sub>
