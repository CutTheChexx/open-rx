---
name: nextjs-patterns
description: >
  Next.js App Router production patterns for 2026. Server Components by default,
  server-client boundary, Suspense at data boundaries, Server Actions with Zod
  validation, caching and revalidation strategies, generateMetadata for SEO,
  streaming and partial prerendering, and the production deployment checklist.
  Triggers on "Next.js", "App Router", "server component", "server action",
  "RSC", "SSR", "ISR", "revalidate", "generateMetadata", "partial prerendering",
  "streaming", or any Next.js architecture task.
metadata:
  version: "1.0.0"
  sources: "Next.js docs 2026, production patterns, App Router guides"
author: "CutTheChexx"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

# Next.js App Router Production Patterns

## 1. The Mental Model: Server-Client Boundary

Every component in Next.js 13+ App Router is a **Server Component by default**. This is the fundamental shift from Pages Router.

### Server Components (Default)
- Zero JavaScript sent to browser
- Direct database and API access
- Access environment variables, tokens, secrets safely
- Never use `useState`, `useEffect`, `useContext`, or browser APIs
- Cannot be interactive (no onclick, change handlers, etc.)
- Perfect for data fetching and logic

### Client Components ("use client")
- Add ONLY at the leaves where you need interactivity
- Use for: form inputs, event handlers, browser APIs, client-side libraries (charts, animations)
- Never wrap an entire page in "use client"
- Children of a client component are automatically client components

### The Boundary Rule
Start with every component as Server. Move down the tree toward "use client" only where necessary.

```
Page (Server)
├── Header (Server) - displays data
├── Navigation (Server) - static nav
├── MainContent (Server)
│   ├── PostsList (Server) - fetches posts
│   │   └── PostCard (Server) - renders post
│   │       └── LikeButton (Client) ← "use client" only here
│   └── Sidebar (Server)
│       └── TagFilter (Client) ← "use client" only here
└── Footer (Server)
```

The Server Component can pass data as props to Client Components. Never re-fetch in the Client Component.

---

## 2. File-Based Routing: App Router Structure

```
app/
├── layout.tsx          # Root layout - wraps all pages
├── page.tsx            # Home page: GET /
├── loading.tsx         # Loading skeleton for entire route
├── error.tsx           # Error boundary (not catches errors in layout)
├── not-found.tsx       # Custom 404
├── robots.ts           # SEO robots.txt
├── sitemap.ts          # SEO sitemap
├── services/
│   ├── layout.tsx      # Layout for /services/* (inherits root)
│   ├── page.tsx        # Services index: GET /services
│   └── [slug]/
│       ├── page.tsx    # Dynamic service: GET /services/my-service
│       ├── error.tsx   # Error boundary for this segment
│       └── not-found.tsx # 404 for this segment
├── api/
│   ├── auth/
│   │   └── route.ts    # POST /api/auth
│   └── revalidate/
│       └── route.ts    # POST /api/revalidate (on-demand)
└── (marketing)/        # Route group - doesn't affect URL
    ├── about/
    │   └── page.tsx    # GET /about
    └── pricing/
        └── page.tsx    # GET /pricing
```

### Key Concepts

**Layout Nesting**: Each `layout.tsx` wraps its children. Root layout wraps everything. Layouts are NOT re-rendered on navigation if URL stays in same layout.

**template.tsx vs layout.tsx**:
- `layout.tsx`: persists across page changes, state preserved
- `template.tsx`: re-creates on every navigation, fresh state each time

**Route Groups** `(marketing)`: Organize routes without affecting URL structure. Use for grouping pages with different layouts.

**Parallel Routes** (advanced): Render multiple segments simultaneously. Example: `/dashboard` with sidebar, main, analytics all independent.

```
app/dashboard/
├── layout.tsx
├── @sidebar/default.tsx
├── @main/default.tsx
└── @analytics/default.tsx
```

Accessed as: `params: { sidebar: [...], main: [...], analytics: [...] }` in layout.

---

## 3. Data Fetching Patterns

### Async Server Components (Native)

No more `getServerSideProps`. Just use `async/await` in Server Components.

```typescript
// app/posts/page.tsx
export default async function PostsPage() {
  const posts = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 } // Revalidate every hour
  }).then(r => r.json());

  return (
    <div>
      {posts.map(post => (
        <article key={post.id}>{post.title}</article>
      ))}
    </div>
  );
}
```

### Revalidation Strategies

**Static Revalidation** (ISR):
```typescript
// Revalidate every 3600 seconds (1 hour)
fetch(url, { next: { revalidate: 3600 } })
```

**Dynamic Revalidation** (On-Demand):
```typescript
// app/api/revalidate/route.ts
import { revalidateTag, revalidatePath } from 'next/cache';

export async function POST(req: Request) {
  const { tag, secret } = await req.json();

  if (secret !== process.env.REVALIDATION_SECRET) {
    return new Response('Unauthorized', { status: 401 });
  }

  // Option 1: By tag
  revalidateTag('posts');

  // Option 2: By path
  revalidatePath('/posts');

  return Response.json({ revalidated: true });
}
```

Tag your fetches:
```typescript
fetch(url, { next: { tags: ['posts'] } })
```

### Request Deduplication

React automatically deduplicates identical fetch calls within the same render:

```typescript
// Both calls are merged into one network request
const users = await fetch('/api/users').then(r => r.json());
const users2 = await fetch('/api/users').then(r => r.json());
// users === users2, single request
```

### Supabase in Server Components

```typescript
// lib/supabase/server.ts
import { createClient } from '@supabase/supabase-js';

export function createServerClient() {
  return createClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY! // Server-only key
  );
}
```

```typescript
// app/todos/page.tsx
import { createServerClient } from '@/lib/supabase/server';

export default async function TodosPage() {
  const supabase = createServerClient();
  const { data: todos, error } = await supabase
    .from('todos')
    .select('*')
    .eq('user_id', userId);

  if (error) {
    throw error; // Caught by error.tsx
  }

  return <TodosList todos={todos} />;
}
```

---

## 4. Suspense & Streaming

Wrap every async data dependency in its own Suspense boundary. This enables streaming: send UI to browser incrementally, not all-at-once.

### Suspense Boundaries

```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react';
import PostsList from './posts-list'; // async component
import Sidebar from './sidebar';      // async component
import PostsSkeleton from './posts-skeleton';
import SidebarSkeleton from './sidebar-skeleton';

export default function DashboardPage() {
  return (
    <div className="flex gap-4">
      <Suspense fallback={<PostsSkeleton />}>
        <PostsList />
      </Suspense>

      <aside>
        <Suspense fallback={<SidebarSkeleton />}>
          <Sidebar />
        </Suspense>
      </aside>
    </div>
  );
}
```

### loading.tsx: Automatic Suspense

Instead of wrapping in Suspense manually, create `loading.tsx`:

```typescript
// app/posts/loading.tsx
export default function Loading() {
  return (
    <div className="space-y-4">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-20 bg-gray-200 rounded animate-pulse" />
      ))}
    </div>
  );
}
```

This automatically wraps the entire route segment in Suspense.

### Skeleton Components (CLS Prevention)

Skeletons MUST match final dimensions to prevent Cumulative Layout Shift:

```typescript
// components/post-skeleton.tsx
export function PostSkeleton() {
  return (
    <article className="p-4 border rounded-lg">
      <div className="h-6 bg-gray-200 rounded w-3/4 mb-2" /> {/* title */}
      <div className="h-4 bg-gray-100 rounded w-full mb-1" /> {/* excerpt line 1 */}
      <div className="h-4 bg-gray-100 rounded w-5/6" /> {/* excerpt line 2 */}
    </article>
  );
}
```

### Partial Prerendering (PPR)

Mark dynamic parts with `dynamicParams = false` and use Suspense boundaries. PPR renders static shell instantly, streams dynamic holes:

```typescript
// app/products/[id]/page.tsx
export const dynamicParams = false; // Use generateStaticParams only
export const revalidate = 3600;

export async function generateStaticParams() {
  const products = await fetch('/api/products').then(r => r.json());
  return products.map(p => ({ id: p.id.toString() }));
}

export default async function ProductPage({ params }) {
  return (
    <div>
      <h1>{/* Static product list */}</h1>
      <Suspense fallback={<AnalyticsSkeleton />}>
        <Analytics productId={params.id} />
      </Suspense>
    </div>
  );
}
```

---

## 5. Server Actions

Server Actions are async functions marked with "use server" that run on the server. They're the modern replacement for API routes for mutations.

### Basic Server Action

```typescript
// app/actions.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';

const createPostSchema = z.object({
  title: z.string().min(5, 'Title must be at least 5 characters'),
  content: z.string().min(10),
  published: z.boolean().default(false),
});

export async function createPost(formData: FormData) {
  // Parse and validate
  const title = formData.get('title');
  const content = formData.get('content');
  const published = formData.get('published') === 'on';

  const result = createPostSchema.safeParse({
    title,
    content,
    published,
  });

  if (!result.success) {
    return {
      error: result.error.flatten().fieldErrors,
      success: false,
    };
  }

  try {
    const post = await db.posts.create(result.data);
    revalidatePath('/posts');

    return {
      success: true,
      post,
    };
  } catch (e) {
    return {
      error: 'Failed to create post',
      success: false,
    };
  }
}
```

### Form with useActionState (Progressive Enhancement)

```typescript
// app/create-post-form.tsx
'use client';

import { useActionState } from 'react';
import { createPost } from './actions';

export function CreatePostForm() {
  const [state, formAction, isPending] = useActionState(createPost, {
    error: null,
    success: false,
  });

  return (
    <form action={formAction} className="space-y-4">
      <div>
        <label htmlFor="title">Title</label>
        <input
          id="title"
          name="title"
          type="text"
          required
        />
        {state.error?.title && (
          <p className="text-red-500">{state.error.title[0]}</p>
        )}
      </div>

      <div>
        <label htmlFor="content">Content</label>
        <textarea
          id="content"
          name="content"
          required
        />
        {state.error?.content && (
          <p className="text-red-500">{state.error.content[0]}</p>
        )}
      </div>

      <label>
        <input name="published" type="checkbox" />
        Publish immediately
      </label>

      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create Post'}
      </button>

      {state.error && typeof state.error === 'string' && (
        <p className="text-red-500">{state.error}</p>
      )}

      {state.success && (
        <p className="text-green-500">Post created! ID: {state.post.id}</p>
      )}
    </form>
  );
}
```

### Mutation + Revalidation Pattern

Always revalidate after mutations:

```typescript
export async function deletePost(id: string) {
  const result = z.string().uuid().safeParse(id);

  if (!result.success) {
    return { error: 'Invalid ID', success: false };
  }

  try {
    await db.posts.delete(id);

    // Revalidate all posts pages
    revalidateTag('posts');
    revalidatePath('/posts');
    revalidatePath('/dashboard');

    return { success: true };
  } catch (e) {
    return { error: 'Failed to delete post', success: false };
  }
}
```

---

## 6. Metadata & SEO

### generateMetadata Function

Runs on server, deduplicated with page fetch:

```typescript
// app/posts/[slug]/page.tsx
import { Metadata } from 'next';

export async function generateMetadata({
  params,
}: {
  params: { slug: string };
}): Promise<Metadata> {
  const post = await fetch(`/api/posts/${params.slug}`).then(r => r.json());

  return {
    title: post.title,
    description: post.excerpt,
    authors: [{ name: post.author }],
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      url: `https://example.com/posts/${params.slug}`,
      images: [
        {
          url: post.image,
          width: 1200,
          height: 630,
        },
      ],
    },
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
      images: [post.image],
    },
  };
}

export default async function PostPage({ params }) {
  const post = await fetch(`/api/posts/${params.slug}`).then(r => r.json());
  // Single fetch due to deduplication

  return <article>{post.content}</article>;
}
```

### Dynamic Metadata from Database

```typescript
export async function generateMetadata({
  params,
}: {
  params: { id: string };
}): Promise<Metadata> {
  const product = await db.products.findUnique({
    where: { id: params.id },
  });

  if (!product) {
    return { title: 'Product Not Found' };
  }

  return {
    title: product.name,
    description: product.description,
  };
}
```

### Open Graph Images with ImageResponse

```typescript
// app/api/og/route.ts
import { ImageResponse } from 'next/og';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const title = searchParams.get('title') || 'Default Title';

  return new ImageResponse(
    (
      <div
        style={{
          display: 'flex',
          background: 'white',
          width: '100%',
          height: '100%',
          flexDirection: 'column',
          justifyContent: 'center',
          alignItems: 'center',
        }}
      >
        <h1 style={{ fontSize: '72px' }}>{title}</h1>
      </div>
    ),
    {
      width: 1200,
      height: 630,
    }
  );
}
```

### Sitemap & Robots

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await db.posts.findMany({ select: { slug: true, updatedAt: true } });

  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 1,
    },
    ...posts.map(post => ({
      url: `https://example.com/posts/${post.slug}`,
      lastModified: post.updatedAt,
      changeFrequency: 'monthly' as const,
      priority: 0.8,
    })),
  ];
}
```

```typescript
// app/robots.ts
import { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/admin/',
    },
    sitemap: 'https://example.com/sitemap.xml',
  };
}
```

---

## 7. Caching Strategy

Next.js has four caching layers. Understand each:

| Layer | What | Duration | Invalidation |
|-------|------|----------|--------------|
| Request Memoization | Same fetch in same render | Per-request | Automatic |
| Data Cache | Fetch results, DB queries | Persistent until revalidation | `revalidateTag()`, `revalidatePath()`, `revalidate: N` |
| Full Route Cache | HTML + RSC payload | Persistent until revalidation | `revalidate: N`, on-demand |
| Browser Cache | HTML + assets in browser | Per browser cache headers | Browser back/forward |

### When to Use Cache Strategies

**Revalidate with time** (ISR for content that changes predictably):
```typescript
fetch(url, { next: { revalidate: 3600 } }) // 1 hour
```

**Revalidate by tag** (for content that changes unpredictably):
```typescript
// Mark with tag
fetch(url, { next: { tags: ['posts'] } })

// Revalidate on mutation
revalidateTag('posts');
```

**No caching** (user-specific, real-time data):
```typescript
fetch(url, { cache: 'no-store' })
```

**Force dynamic rendering** (when you need fresh data every request):
```typescript
export const dynamic = 'force-dynamic';
```

---

## 8. Production Checklist

Before deploying a Next.js App Router app to production:

- [ ] **Audit "use client"**: Every "use client" component must actually need interactivity. Remove if only rendering data.
- [ ] **Every fetch has revalidation**: No `fetch()` without explicit `revalidate: N` or `cache: 'no-store'`. Default behavior is persistent caching.
- [ ] **Suspense at data boundaries**: Every async component wrapped in Suspense or using loading.tsx.
- [ ] **Error boundaries**: Every route has error.tsx. Root has error.tsx.
- [ ] **Metadata on every page**: Every page.tsx exports generateMetadata or has static metadata.
- [ ] **Image optimization**: Use `next/image` Image component, never `<img>`.
- [ ] **Font optimization**: Use `next/font` for Google Fonts, not `<link>` in HTML.
- [ ] **Environment variables**: Server secrets use plain names. Client vars prefixed with `NEXT_PUBLIC_`.
- [ ] **Bundle analysis**: Run `npm run build` and check `.next/static` size. Code split large components.
- [ ] **Database connection pooling**: Production databases need connection limits. Use Vercel Postgres or Prisma with pooling.
- [ ] **Error logging**: Catch errors in Server Actions and layout/page errors. Log to external service (Sentry, LogRocket).
- [ ] **Performance monitoring**: Set up Core Web Vitals tracking (Vercel Analytics, SpeedInsights).
- [ ] **Security headers**: Configure Content-Security-Policy, HSTS in next.config.js or Vercel.
- [ ] **Rate limiting**: API routes and Server Actions need rate limiting in production.
- [ ] **Testing**: Test Server Actions separately from Client Components. Test dynamic segments with generateStaticParams.

<sub>Open RX by CutTheChexx — The Prescription.</sub>
