---
name: database-ops
description: >
  Production-grade database operations and schema design for Supabase/PostgreSQL.
  Master Row Level Security, query optimization, migrations, indexing strategies,
  and ready-to-use schemas for users, orgs, projects, leads, pipelines, and audit logs.
  Deep coverage of RLS policies, performance tuning, backup patterns, and Edge Function
  database patterns. Write secure, scalable Postgres applications with confidence.
metadata:
  version: "1.0.0"
  author: "CutTheChexx"
  sources: "Supabase docs, PostgreSQL best practices, production database patterns"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

## Database Operations & Schema Design

Supabase provides a PostgreSQL database with first-class JavaScript/TypeScript support. This skill covers the complete lifecycle: schema design, migrations, security (RLS), query patterns, performance optimization, and backup strategies.

---

## 1. Schema Design Principles

### Naming Conventions

**Tables**: Use snake_case, singular names.
```sql
CREATE TABLE users (
  id uuid PRIMARY KEY,
  email text UNIQUE NOT NULL,
  created_at timestamp with time zone DEFAULT now()
);

CREATE TABLE organizations (
  id uuid PRIMARY KEY,
  name text NOT NULL,
  owner_id uuid NOT NULL REFERENCES users(id)
);
```

**Why singular?** `users` is a table of users. A row represents one user. Singular is clearer and aligns with REST conventions.

**Columns**: snake_case for all columns. Use suffixes for clarity:
- `_id`: foreign key reference
- `_at`: timestamp
- `_count`: numeric aggregate
- `_status`: enum values
- `_data`: JSONB columns

### Primary Keys

**UUID vs Serial:**

UUID (recommended):
```sql
id uuid PRIMARY KEY DEFAULT gen_random_uuid()
```
- No guessing: impossible to predict next ID
- Distributed systems: multi-database sync
- Privacy: don't expose usage patterns via sequential IDs
- Cost: 16 bytes vs 4 for serial

Serial (use sparingly):
```sql
id bigserial PRIMARY KEY
```
- Used: legacy systems, extreme disk constraints
- Problem: reveals row counts, predictable IDs

**Always use UUID for user-facing identifiers.**

### Timestamps Pattern

Every table should have audit timestamps:
```sql
CREATE TABLE documents (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  title text NOT NULL,
  owner_id uuid NOT NULL REFERENCES users(id),

  -- Audit timestamps
  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now(),
  deleted_at timestamp with time zone DEFAULT NULL,

  CONSTRAINT documents_owner_fk FOREIGN KEY (owner_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Trigger to auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_documents_updated_at
BEFORE UPDATE ON documents
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();
```

**Soft Deletes**: Use `deleted_at` instead of hard deletes. Allows recovery, maintains referential integrity, supports audit trails.

### Status Enums

Create enum types for status fields:
```sql
CREATE TYPE membership_role AS ENUM ('owner', 'admin', 'member', 'guest');

CREATE TABLE organization_members (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  user_id uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role membership_role NOT NULL DEFAULT 'member',

  created_at timestamp with time zone DEFAULT now(),
  CONSTRAINT org_members_unique UNIQUE (organization_id, user_id)
);
```

Enums prevent invalid states at the database level. Default to a sensible value.

### Polymorphic Associations

When multiple tables reference a common entity, use a tag column + ID:

```sql
CREATE TABLE audit_logs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,

  -- Polymorphic: what changed?
  entity_type text NOT NULL, -- 'user', 'project', 'document'
  entity_id uuid NOT NULL,

  action text NOT NULL, -- 'created', 'updated', 'deleted'
  changes jsonb,
  performed_by uuid NOT NULL REFERENCES users(id),

  created_at timestamp with time zone DEFAULT now()
);

CREATE INDEX idx_audit_logs_entity ON audit_logs(organization_id, entity_type, entity_id);
```

Don't create separate tables for each entity type. One audit table with polymorphic lookups is cleaner and scalable.

---

## 2. Supabase Setup

### Client Configuration

**Server-side (Node.js, Edge Functions):**
```typescript
import { createClient } from "@supabase/supabase-js";

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_SERVICE_ROLE_KEY // Full database access
);
```

**Browser client (React, Vue, etc.):**
```typescript
import { createClient } from "@supabase/supabase-js";

const supabase = createClient(
  process.env.REACT_APP_SUPABASE_URL,
  process.env.REACT_APP_SUPABASE_ANON_KEY // Limited by RLS
);
```

**Key difference:**
- `SERVICE_ROLE_KEY`: Bypasses RLS. Never expose to browser.
- `ANON_KEY`: Public key. Respects RLS policies.

### Environment Variables

```bash
# .env.local
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbGc...
SUPABASE_ANON_KEY=eyJhbGc...
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGc...
```

Only variables prefixed with `NEXT_PUBLIC_` (or framework equivalent) leak to browser. Keep service role key in `.env.local`, not `.env`.

### TypeScript Type Generation

Generate types from your schema automatically:

```bash
npx supabase gen types typescript --project-id your-project > types/database.ts
```

Use in your code:
```typescript
import { Database } from "@/types/database";

type User = Database["public"]["Tables"]["users"]["Row"];
type InsertUser = Database["public"]["Tables"]["users"]["Insert"];
type UpdateUser = Database["public"]["Tables"]["users"]["Update"];

const user: User = { id: "...", email: "..." };
```

Generated types match your schema exactly. Update after migrations.

### Connection Pooling

Supabase provides two connection pools:

- **Session pool** (default): Per-client persistent connection. Use for long-lived clients.
- **Transaction pool**: Lightweight. Use for serverless (Edge Functions, Lambda).

In `supabase/config.toml`:
```toml
db_url = "postgres://user:pass@db.supabase.co:6543/postgres"
db_shadow_database_url = "postgres://user:pass@db.supabase.co:6543/postgres_shadow"
```

For Edge Functions, always use the transaction pool (Supabase's default).

---

## 3. Row Level Security (RLS)

**RLS is mandatory for multi-tenant applications.** Without it, any authenticated user can read/write all data.

### Enable RLS on Every Table

```sql
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
```

Create a helper to enable RLS:
```sql
-- Enable RLS on all tables in public schema
DO $$
DECLARE
  row RECORD;
BEGIN
  FOR row IN
    SELECT tablename FROM pg_tables WHERE schemaname = 'public'
  LOOP
    EXECUTE 'ALTER TABLE ' || row.tablename || ' ENABLE ROW LEVEL SECURITY;';
  END LOOP;
END $$;
```

### Policy Patterns

**Pattern 1: User owns row**
```sql
-- Users can only read their own profile
CREATE POLICY "users_select_own"
ON users
FOR SELECT
USING (auth.uid() = id);

-- Users can only update their own profile
CREATE POLICY "users_update_own"
ON users
FOR UPDATE
USING (auth.uid() = id)
WITH CHECK (auth.uid() = id);
```

**Pattern 2: Org membership**
```sql
-- Users can read projects in organizations they're members of
CREATE POLICY "projects_select_org_members"
ON projects
FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id
    FROM organization_members
    WHERE user_id = auth.uid()
  )
);
```

**Pattern 3: Public read, authenticated write**
```sql
CREATE POLICY "documents_select_public"
ON documents
FOR SELECT
USING (is_public = true OR owner_id = auth.uid());

CREATE POLICY "documents_insert_auth"
ON documents
FOR INSERT
WITH CHECK (owner_id = auth.uid());
```

**Pattern 4: Admin override**
```sql
-- Admins can access everything in their org
CREATE POLICY "projects_admin_access"
ON projects
FOR ALL
USING (
  (SELECT role FROM organization_members
   WHERE organization_id = projects.organization_id
   AND user_id = auth.uid()) = 'admin'
);
```

**Pattern 5: Nested relation access**
```sql
-- Access tasks through project membership
CREATE POLICY "tasks_access_via_project"
ON tasks
FOR SELECT
USING (
  project_id IN (
    SELECT id FROM projects
    WHERE organization_id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid()
    )
  )
);
```

### RLS Testing

Test policies before deploying:

```sql
-- Set a user context
SET "request.jwt.claims" = '{"sub": "user-uuid-here"}';

-- Test SELECT
SELECT * FROM projects; -- Should only return user's projects

-- Test INSERT
INSERT INTO projects (name, organization_id)
VALUES ('Test', 'org-uuid')
RETURNING *;

-- Reset context
RESET "request.jwt.claims";
```

In JavaScript:
```typescript
// Switch auth context
const { data, error } = await supabase
  .from("projects")
  .select("*");
// If no RLS policies match, returns []
```

### Common RLS Mistakes

**Mistake 1: Forgetting deleted_at filter**
```sql
-- WRONG: Deleted records still visible
CREATE POLICY "projects_select"
ON projects
FOR SELECT
USING (owner_id = auth.uid());

-- RIGHT: Exclude soft-deleted rows
CREATE POLICY "projects_select"
ON projects
FOR SELECT
USING (
  owner_id = auth.uid()
  AND deleted_at IS NULL
);
```

**Mistake 2: N+1 subqueries in policies**
```sql
-- SLOW: Subquery runs for every row
CREATE POLICY "tasks_slow"
ON tasks
FOR SELECT
USING (
  project_id IN (
    SELECT id FROM projects
    WHERE organization_id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid()
    )
  )
);

-- FASTER: Use CTE and index
CREATE POLICY "tasks_fast"
ON tasks
FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM projects p
    JOIN organization_members om
      ON p.organization_id = om.organization_id
    WHERE p.id = tasks.project_id
    AND om.user_id = auth.uid()
  )
);
```

**Mistake 3: Updating columns without checking permissions**
```sql
-- WRONG: Any user can bypass permissions via UPDATE
CREATE POLICY "projects_update"
ON projects
FOR UPDATE
USING (owner_id = auth.uid())
WITH CHECK (true); -- INSECURE

-- RIGHT: Check permissions on both old and new data
CREATE POLICY "projects_update"
ON projects
FOR UPDATE
USING (owner_id = auth.uid())
WITH CHECK (owner_id = auth.uid());
```

---

## 4. Migrations

### SQL Migration Patterns

Create migration files in `supabase/migrations/`:

**Naming convention:**
```
YYYY-MM-DD_HHmmss_descriptive_name.sql
```

**Example migration:**
```sql
-- File: supabase/migrations/2026-04-02_120000_create_projects_table.sql

-- Up migration
CREATE TABLE projects (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  name text NOT NULL,
  description text,

  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now(),
  deleted_at timestamp with time zone DEFAULT NULL,

  CONSTRAINT projects_unique_name_per_org UNIQUE (organization_id, name)
);

ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

CREATE INDEX idx_projects_organization ON projects(organization_id)
WHERE deleted_at IS NULL;

CREATE POLICY "projects_select_org_members"
ON projects
FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND deleted_at IS NULL
  )
  AND deleted_at IS NULL
);
```

### Up/Down Migrations

Supabase auto-handles up migrations. For manual reversions:

```sql
-- File: supabase/migrations/2026-04-02_120000_create_projects_table.sql

-- Up
CREATE TABLE projects (...);

-- Down (commented for safety)
/*
DROP POLICY IF EXISTS "projects_select_org_members" ON projects;
DROP INDEX IF EXISTS idx_projects_organization;
DROP TABLE IF EXISTS projects;
*/
```

To revert in development:
```bash
supabase db reset # Applies all migrations from scratch
```

### Seed Data

Create seed files in `supabase/seed.sql`:

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Insert test user
INSERT INTO users (id, email)
VALUES
  ('11111111-1111-1111-1111-111111111111', 'test@example.com')
ON CONFLICT DO NOTHING;

-- Insert test org
INSERT INTO organizations (id, name, owner_id)
VALUES
  ('22222222-2222-2222-2222-222222222222', 'Test Org', '11111111-1111-1111-1111-111111111111')
ON CONFLICT DO NOTHING;

-- Insert membership
INSERT INTO organization_members (organization_id, user_id, role)
VALUES
  ('22222222-2222-2222-2222-222222222222', '11111111-1111-1111-1111-111111111111', 'owner')
ON CONFLICT DO NOTHING;
```

Run with:
```bash
supabase db push
supabase seed restore
```

### Schema Versioning

Track schema version in a table:
```sql
CREATE TABLE schema_versions (
  id serial PRIMARY KEY,
  version text NOT NULL UNIQUE,
  applied_at timestamp with time zone DEFAULT now()
);

INSERT INTO schema_versions (version) VALUES ('1.0.0');
```

Query before running migrations:
```typescript
const { data } = await supabase
  .from("schema_versions")
  .select("version")
  .order("applied_at", { ascending: false })
  .limit(1);

console.log("Current schema version:", data?.[0]?.version);
```

---

## 5. Query Patterns

### Select with Relations

**Simple join:**
```typescript
const { data: projects } = await supabase
  .from("projects")
  .select("*, organization:organizations(id, name)")
  .eq("organization_id", orgId);
```

**Deep nesting:**
```typescript
const { data: tasks } = await supabase
  .from("tasks")
  .select(`
    id,
    title,
    project:projects(
      id,
      name,
      organization:organizations(id, name)
    ),
    assignee:users(id, email)
  `)
  .eq("project_id", projectId);
```

**Many-to-many through junction table:**
```sql
CREATE TABLE project_tags (
  project_id uuid NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  tag_id uuid NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  PRIMARY KEY (project_id, tag_id)
);

-- Query
SELECT p.*, array_agg(t.name) as tags
FROM projects p
LEFT JOIN project_tags pt ON p.id = pt.project_id
LEFT JOIN tags t ON pt.tag_id = t.id
WHERE p.id = $1
GROUP BY p.id;
```

### Filtering

```typescript
// Equals
.eq("status", "active")

// Not equals
.neq("status", "deleted")

// Greater/less than
.gt("price", 100)
.lt("created_at", "2026-01-01")

// IN list
.in("id", ["id1", "id2", "id3"])

// Text search
.ilike("email", "%@example.com%")

// AND (multiple filters)
.eq("status", "active")
.eq("organization_id", orgId)

// OR (use raw SQL)
const { data } = await supabase
  .from("projects")
  .select("*")
  .or("owner_id.eq." + userId + ",is_public.eq.true");
```

### Pagination (Range-based)

```typescript
const PAGE_SIZE = 20;
const pageNum = 1; // 1-indexed

const from = (pageNum - 1) * PAGE_SIZE;
const to = from + PAGE_SIZE - 1;

const { data: projects, count } = await supabase
  .from("projects")
  .select("*", { count: "exact" })
  .range(from, to);

const totalPages = Math.ceil((count || 0) / PAGE_SIZE);
```

### Pagination (Cursor-based)

```typescript
// Fetch next page using last row's ID
const pageSize = 20;

const { data: projects } = await supabase
  .from("projects")
  .select("id, name, created_at")
  .order("created_at", { ascending: false })
  .limit(pageSize + 1); // Fetch extra to detect if more exists

const hasMore = projects.length > pageSize;
const items = projects.slice(0, pageSize);
const lastCursor = items[items.length - 1]?.id;

// Next request
const { data: nextProjects } = await supabase
  .from("projects")
  .select("id, name, created_at")
  .lt("created_at", lastCursor)
  .order("created_at", { ascending: false })
  .limit(pageSize);
```

### Full-Text Search

Enable full-text search:
```sql
ALTER TABLE documents ADD COLUMN search_vector tsvector;

CREATE INDEX idx_documents_search ON documents USING gin(search_vector);

CREATE TRIGGER documents_search_update
BEFORE INSERT OR UPDATE ON documents
FOR EACH ROW
EXECUTE FUNCTION tsvector_update_trigger(search_vector, 'pg_catalog.english', title, content);
```

Query:
```typescript
const { data: results } = await supabase
  .from("documents")
  .select("id, title")
  .textSearch("search_vector", "postgres & database");
```

### Aggregations

```sql
-- Count
SELECT COUNT(*) as total_projects
FROM projects
WHERE organization_id = $1;

-- Sum
SELECT SUM(price) as total_revenue
FROM orders
WHERE status = 'completed';

-- Group by
SELECT
  status,
  COUNT(*) as count,
  AVG(price) as avg_price
FROM orders
GROUP BY status;

-- Window functions (running totals, ranks)
SELECT
  name,
  price,
  SUM(price) OVER (ORDER BY created_at) as running_total,
  ROW_NUMBER() OVER (ORDER BY price DESC) as rank
FROM products;
```

In Supabase (use RPC or raw SQL for aggregations):
```typescript
const { data } = await supabase.rpc("count_projects", { org_id: orgId });
```

### RPC Functions

Define a function in the database:
```sql
CREATE OR REPLACE FUNCTION count_projects(org_id uuid)
RETURNS bigint AS $$
  SELECT COUNT(*)::bigint
  FROM projects
  WHERE organization_id = org_id
  AND deleted_at IS NULL;
$$ LANGUAGE sql STABLE;
```

Call from JavaScript:
```typescript
const { data: count } = await supabase.rpc("count_projects", { org_id: orgId });
console.log("Projects:", count);
```

---

## 6. Indexes

### When to Index

Index on:
- Foreign keys (joined frequently)
- Columns in WHERE clauses
- Columns in ORDER BY
- Columns in GROUP BY
- High-cardinality (distinct values) columns

Don't index:
- Boolean columns (low cardinality)
- Columns updated frequently (index maintenance cost)
- JSONB fields (unless you use specific operators)

### Index Types

**B-tree (default):**
```sql
CREATE INDEX idx_users_email ON users(email);
```
Used for: equality, range queries (`=`, `<`, `>`, `IN`).

**Hash:**
```sql
CREATE INDEX idx_users_email_hash ON users USING hash(email);
```
Used for: exact match only, rarely needed.

**GIN (Generalized Inverted Index):**
```sql
CREATE INDEX idx_documents_search ON documents USING gin(search_vector);
CREATE INDEX idx_documents_tags ON documents USING gin(tags);
```
Used for: full-text search, array containment, JSONB operators.

**GiST (Generalized Search Tree):**
```sql
CREATE INDEX idx_points_spatial ON points USING gist(location);
```
Used for: spatial data, range types, custom data types.

**BRIN (Block Range Index):**
```sql
CREATE INDEX idx_events_time ON events USING brin(created_at);
```
Used for: very large tables with naturally sorted data, compact.

### Composite Indexes

Index multiple columns together:
```sql
CREATE INDEX idx_org_members_lookup ON organization_members(organization_id, user_id);
```

Order matters: most selective columns first.

### Partial Indexes

Index only rows matching a condition:
```sql
-- Only index active projects (exclude soft-deletes)
CREATE INDEX idx_projects_active ON projects(organization_id)
WHERE deleted_at IS NULL;
```

Reduces index size, faster inserts.

### Covering Indexes

Include non-indexed columns to avoid table lookups:
```sql
-- Index includes name so query doesn't need table lookup
CREATE INDEX idx_users_email_covering ON users(email) INCLUDE (name, avatar_url);
```

Query can satisfy entirely from index: "index-only scan".

### EXPLAIN ANALYZE

Analyze query performance:
```sql
EXPLAIN ANALYZE
SELECT p.*, COUNT(t.id) as task_count
FROM projects p
LEFT JOIN tasks t ON p.id = t.project_id
WHERE p.organization_id = 'org-uuid'
GROUP BY p.id;
```

Output shows:
- Sequential scan vs index scan
- Rows estimated vs actual
- Node costs
- Planning/execution time

Look for:
- **Seq Scan** on large tables: add index
- **Index scan loops**: may need different index
- Estimated != actual: stale statistics (`ANALYZE table_name`)

---

## 7. Common Schemas

Ready-to-use schemas included in `references/schema-recipes.md`.

Key schemas:
1. Users + profiles with auth trigger
2. Organizations + members + roles
3. Projects + tasks
4. Leads pipeline
5. Audit log
6. File attachments
7. Notifications

Copy and adapt to your needs.

---

## 8. Edge Functions + Database

### Calling Supabase from Edge Functions

```typescript
// supabase/functions/process-order/index.ts

import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

Deno.serve(async (req) => {
  // Service role key: bypass RLS
  const supabase = createClient(
    Deno.env.get("SUPABASE_URL")!,
    Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")!
  );

  // Insert order
  const { data: order, error } = await supabase
    .from("orders")
    .insert({ user_id: userId, total: 100 })
    .select()
    .single();

  if (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 400,
    });
  }

  return new Response(JSON.stringify(order), { status: 200 });
});
```

Deploy:
```bash
supabase functions deploy process-order
```

### Service Role Key Usage

Service role key bypasses RLS. Use for:
- System-initiated actions (cron jobs, webhooks, admin operations)
- Multi-user transactions
- Operations that need cross-tenant access

Never expose to browser or public APIs.

### Transaction Patterns

PostgreSQL transactions in Edge Functions:

```typescript
const { data: result, error } = await supabase.rpc("create_order_with_items", {
  user_id: userId,
  items: [
    { product_id: "p1", quantity: 2 },
    { product_id: "p2", quantity: 1 },
  ],
});
```

Define the function:
```sql
CREATE OR REPLACE FUNCTION create_order_with_items(
  user_id uuid,
  items jsonb
)
RETURNS uuid AS $$
DECLARE
  order_id uuid;
  item jsonb;
BEGIN
  -- Create order
  INSERT INTO orders (user_id, total)
  VALUES (user_id, 0)
  RETURNING id INTO order_id;

  -- Add items (atomic)
  FOR item IN SELECT * FROM jsonb_array_elements(items)
  LOOP
    INSERT INTO order_items (order_id, product_id, quantity)
    VALUES (order_id, (item->>'product_id')::uuid, (item->>'quantity')::int);
  END LOOP;

  RETURN order_id;
EXCEPTION WHEN OTHERS THEN
  -- Rollback handled automatically
  RAISE;
END;
$$ LANGUAGE plpgsql;
```

---

## 9. Performance

### Connection Pooling

Supabase manages connection pooling. For Edge Functions:
- Connections are pooled at transaction level
- Max 100 concurrent connections per project
- Idle connections close after 5 minutes

Monitor in Supabase dashboard: Settings → Database → Connection pooling.

### Query Optimization

**N+1 Prevention:**

Bad:
```typescript
const projects = await supabase
  .from("projects")
  .select("id, name");

for (const project of projects) {
  const tasks = await supabase
    .from("tasks")
    .select("id, title")
    .eq("project_id", project.id); // One query per project!
}
```

Good:
```typescript
const projectIds = projects.map(p => p.id);

const { data: tasks } = await supabase
  .from("tasks")
  .select("id, title, project_id")
  .in("project_id", projectIds); // Single query
```

### Materialized Views

Cache expensive queries:
```sql
CREATE MATERIALIZED VIEW project_stats AS
SELECT
  p.id,
  p.name,
  COUNT(t.id) as task_count,
  COUNT(t.id) FILTER (WHERE t.status = 'completed') as completed_count,
  AVG(EXTRACT(EPOCH FROM (t.completed_at - t.created_at))) / 3600 as avg_hours
FROM projects p
LEFT JOIN tasks t ON p.id = t.project_id
WHERE p.deleted_at IS NULL
GROUP BY p.id, p.name;

CREATE INDEX idx_project_stats_id ON project_stats(id);

-- Refresh when data changes
REFRESH MATERIALIZED VIEW CONCURRENTLY project_stats;
```

Query materialized views like tables:
```typescript
const { data: stats } = await supabase
  .from("project_stats")
  .select("*")
  .eq("id", projectId);
```

Refresh on-demand or with cron.

### Scheduled Jobs (pg_cron)

Enable the extension:
```sql
CREATE EXTENSION IF NOT EXISTS pg_cron;
```

Schedule tasks:
```sql
-- Refresh materialized views every hour
SELECT cron.schedule('refresh-project-stats', '0 * * * *',
  'REFRESH MATERIALIZED VIEW CONCURRENTLY project_stats');

-- Clean up old audit logs every week
SELECT cron.schedule('cleanup-old-logs', '0 3 * * 0',
  'DELETE FROM audit_logs WHERE created_at < now() - INTERVAL ''1 year''');

-- Send daily digest emails
SELECT cron.schedule('send-daily-digest', '0 6 * * *',
  'SELECT send_daily_digest()');
```

View scheduled jobs:
```sql
SELECT * FROM cron.job;
```

---

## 10. Backup & Recovery

### Point-in-Time Recovery (PITR)

Supabase maintains 7-day PITR for Pro/Team plans.

Recover to a specific timestamp:
```bash
# In Supabase dashboard: Settings → Backups → Restore
# Select date/time and restore to new project
```

Or use Postgres directly:
```bash
# Get WAL archive
aws s3 cp s3://supabase-backups/project_id/ ./backups --recursive

# Restore to specific time
pg_basebackup -D ./backup_data -h backup.supabase.co -U postgres
```

### pg_dump Patterns

Export full database:
```bash
pg_dump postgres://user:pass@db.supabase.co/postgres > backup.sql
```

Export specific schema:
```bash
pg_dump -n public postgres://user:pass@db.supabase.co/postgres > schema.sql
```

Export data only (no DDL):
```bash
pg_dump --data-only postgres://user:pass@db.supabase.co/postgres > data.sql
```

Export with custom format (faster, compressible):
```bash
pg_dump -F custom postgres://user:pass@db.supabase.co/postgres > backup.dump
pg_restore backup.dump > restored.sql
```

Restore:
```bash
psql postgres://user:pass@db.supabase.co/postgres < backup.sql
```

### Data Export

Export table as CSV:
```bash
psql postgres://user:pass@db.supabase.co/postgres -c "COPY projects TO STDOUT CSV HEADER" > projects.csv
```

Or in application:
```typescript
const { data: projects } = await supabase
  .from("projects")
  .select("*")
  .then(({ data }) => {
    const csv = [Object.keys(data[0]).join(',')];
    csv.push(...data.map(row => Object.values(row).join(',')));
    return csv.join('\n');
  });
```

---

## Testing RLS in Development

Enable verbose logging:
```sql
ALTER SYSTEM SET log_rls_statements = on;
SELECT pg_reload_conf();
```

Check logs:
```bash
supabase logs --follow
```

Test with different users:
```typescript
// Simulate user context
const { data: myProjects } = await supabase
  .from("projects")
  .select("*");

// Simulate different user
const otherUser = createClient(url, anonKey);
const { data: theirProjects } = await otherUser
  .from("projects")
  .select("*");

// Should be empty (RLS blocks access)
console.assert(theirProjects?.length === 0);
```

---

## Debugging Common Issues

**Error: Row level security policy violated**
- Check RLS policies exist for the operation (SELECT, INSERT, UPDATE, DELETE)
- Verify `auth.uid()` matches expected user ID
- Check JWT token is valid and current

**Error: Foreign key constraint violated**
- Parent row doesn't exist
- Cascading delete removed parent before child deleted
- Check referential integrity with: `SELECT * FROM orders WHERE user_id = $1 AND user_id NOT IN (SELECT id FROM users)`

**Error: Unique constraint violated**
- Duplicate value already exists
- Use `ON CONFLICT DO UPDATE` for upserts: `INSERT INTO users (id, email) VALUES ($1, $2) ON CONFLICT (id) DO UPDATE SET email = $2`

**Slow queries**
- Run `EXPLAIN ANALYZE` to identify missing indexes
- Check connection pool: `SELECT count(*) FROM pg_stat_activity;`
- Monitor slow logs in Supabase dashboard

**Out of disk space**
- Check `SELECT pg_database_size('postgres');` in Dashboard → SQL Editor
- Archive old data to cold storage
- Increase disk space in Plan settings

---

## Best Practices Checklist

- [ ] Enable RLS on all tables
- [ ] Use UUIDs for all PKs
- [ ] Add `created_at`, `updated_at`, `deleted_at` timestamps
- [ ] Index foreign keys and WHERE/ORDER BY columns
- [ ] Test RLS policies before deployment
- [ ] Use migrations for schema changes
- [ ] Generate TypeScript types from schema
- [ ] Implement soft deletes with `deleted_at` filter
- [ ] Use service role key only in backend
- [ ] Monitor query performance with EXPLAIN ANALYZE
- [ ] Set up automated backups
- [ ] Document schema with comments

---

<sub>Open RX by CutTheChexx — The Prescription.</sub>
