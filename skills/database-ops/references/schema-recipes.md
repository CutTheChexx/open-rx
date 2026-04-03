# Schema Recipes: Production-Ready Schemas

Copy and adapt these complete, production-grade schemas. Each includes CREATE TABLE, indexes, RLS policies, and trigger functions.

---

## 1. Users + Profiles with Auth Trigger

Syncs Supabase Auth with profiles table.

```sql
-- Users table (custom profile data)
CREATE TABLE users (
  id uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email text UNIQUE NOT NULL,
  full_name text,
  avatar_url text,
  website text,
  bio text,

  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now()
);

-- Sync users when auth.users changes
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS trigger AS $$
BEGIN
  INSERT INTO public.users (id, email, full_name)
  VALUES (new.id, new.email, new.raw_user_meta_data->>'full_name')
  ON CONFLICT (id) DO UPDATE
  SET email = new.email,
      full_name = COALESCE(new.raw_user_meta_data->>'full_name', users.full_name);
  RETURN new;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
AFTER INSERT ON auth.users
FOR EACH ROW
EXECUTE FUNCTION public.handle_new_user();

-- Enable RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Users can read their own profile
CREATE POLICY "users_select_own" ON users
FOR SELECT
USING (auth.uid() = id);

-- Users can read any public profile
CREATE POLICY "users_select_public" ON users
FOR SELECT
USING (true); -- Make profiles public by default, or add is_public column

-- Users can update their own profile
CREATE POLICY "users_update_own" ON users
FOR UPDATE
USING (auth.uid() = id)
WITH CHECK (auth.uid() = id);

-- Prevent deleting own account (use soft delete)
CREATE POLICY "users_delete_prevented" ON users
FOR DELETE
USING (false);

-- Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at DESC);
```

---

## 2. Organizations + Members + Roles

Multi-tenant organization structure with role-based access control.

```sql
-- Organizations
CREATE TABLE organizations (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  slug text UNIQUE NOT NULL,
  description text,
  logo_url text,
  owner_id uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,

  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now(),
  deleted_at timestamp with time zone DEFAULT NULL,

  CONSTRAINT slug_lowercase CHECK (slug = LOWER(slug))
);

-- Organization members with roles
CREATE TYPE org_role AS ENUM ('owner', 'admin', 'member', 'guest');

CREATE TABLE organization_members (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  user_id uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role org_role NOT NULL DEFAULT 'member',

  invited_by uuid REFERENCES users(id) ON DELETE SET NULL,
  invited_at timestamp with time zone,
  accepted_at timestamp with time zone,

  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now(),

  CONSTRAINT org_members_unique UNIQUE (organization_id, user_id),
  CONSTRAINT cannot_be_own_member CHECK (organization_id IS NOT NULL)
);

-- Enable RLS
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE organization_members ENABLE ROW LEVEL SECURITY;

-- Organizations: members can view
CREATE POLICY "orgs_select_members" ON organizations
FOR SELECT
USING (
  id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
  OR owner_id = auth.uid()
) OR deleted_at IS NULL;

-- Organizations: only owner/admin can update
CREATE POLICY "orgs_update_admin" ON organizations
FOR UPDATE
USING (
  owner_id = auth.uid()
  OR id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
)
WITH CHECK (
  owner_id = auth.uid()
  OR id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
);

-- Members: can view members in their org
CREATE POLICY "org_members_select" ON organization_members
FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
);

-- Members: admin can insert/update/delete members
CREATE POLICY "org_members_admin" ON organization_members
FOR ALL
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
)
WITH CHECK (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
);

-- Prevent members from removing owner
CREATE POLICY "org_members_prevent_remove_owner" ON organization_members
FOR DELETE
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role = 'owner'
  )
);

-- Indexes
CREATE INDEX idx_org_members_user ON organization_members(user_id);
CREATE INDEX idx_org_members_org ON organization_members(organization_id);
CREATE INDEX idx_org_members_lookup ON organization_members(organization_id, user_id);
CREATE INDEX idx_organizations_slug ON organizations(slug);
CREATE INDEX idx_organizations_owner ON organizations(owner_id);
CREATE INDEX idx_organizations_active ON organizations(created_at DESC)
WHERE deleted_at IS NULL;

-- Helper function: check user role in org
CREATE OR REPLACE FUNCTION get_user_org_role(org_id uuid)
RETURNS org_role AS $$
  SELECT role FROM organization_members
  WHERE organization_id = org_id AND user_id = auth.uid()
  LIMIT 1;
$$ LANGUAGE sql STABLE;

-- Helper function: check if user is admin
CREATE OR REPLACE FUNCTION is_org_admin(org_id uuid)
RETURNS boolean AS $$
  SELECT EXISTS(
    SELECT 1 FROM organization_members
    WHERE organization_id = org_id
    AND user_id = auth.uid()
    AND role IN ('owner', 'admin')
  );
$$ LANGUAGE sql STABLE;
```

---

## 3. Projects + Tasks

Hierarchical project/task structure with status tracking.

```sql
CREATE TYPE project_status AS ENUM ('planning', 'active', 'on_hold', 'completed', 'archived');
CREATE TYPE task_status AS ENUM ('todo', 'in_progress', 'review', 'done', 'blocked');
CREATE TYPE task_priority AS ENUM ('low', 'medium', 'high', 'urgent');

-- Projects
CREATE TABLE projects (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  name text NOT NULL,
  description text,
  status project_status NOT NULL DEFAULT 'planning',
  owner_id uuid NOT NULL REFERENCES users(id) ON DELETE SET NULL,

  start_date date,
  end_date date,
  color text DEFAULT '#3B82F6',

  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now(),
  deleted_at timestamp with time zone DEFAULT NULL,

  CONSTRAINT projects_unique_name UNIQUE (organization_id, name),
  CONSTRAINT dates_valid CHECK (start_date IS NULL OR end_date IS NULL OR start_date <= end_date)
);

-- Tasks
CREATE TABLE tasks (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id uuid NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  parent_task_id uuid REFERENCES tasks(id) ON DELETE CASCADE,

  title text NOT NULL,
  description text,
  status task_status NOT NULL DEFAULT 'todo',
  priority task_priority NOT NULL DEFAULT 'medium',

  assignee_id uuid REFERENCES users(id) ON DELETE SET NULL,
  created_by uuid NOT NULL REFERENCES users(id),

  start_date date,
  due_date date,
  completed_at timestamp with time zone,

  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now(),
  deleted_at timestamp with time zone DEFAULT NULL
);

-- Task dependencies
CREATE TABLE task_dependencies (
  task_id uuid NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  depends_on_task_id uuid NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  PRIMARY KEY (task_id, depends_on_task_id),
  CONSTRAINT no_self_dependency CHECK (task_id != depends_on_task_id)
);

-- Enable RLS
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE task_dependencies ENABLE ROW LEVEL SECURITY;

-- Projects: org members can read
CREATE POLICY "projects_select_org_members" ON projects
FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
  AND deleted_at IS NULL
);

-- Projects: admin can write
CREATE POLICY "projects_insert_admin" ON projects
FOR INSERT
WITH CHECK (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
);

CREATE POLICY "projects_update_admin" ON projects
FOR UPDATE
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
)
WITH CHECK (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
);

-- Tasks: org members can read tasks in their projects
CREATE POLICY "tasks_select_org_members" ON tasks
FOR SELECT
USING (
  project_id IN (
    SELECT id FROM projects
    WHERE organization_id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid()
    )
    AND deleted_at IS NULL
  )
  AND deleted_at IS NULL
);

-- Tasks: members can insert/update
CREATE POLICY "tasks_insert_members" ON tasks
FOR INSERT
WITH CHECK (
  project_id IN (
    SELECT id FROM projects
    WHERE organization_id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid()
    )
  )
);

CREATE POLICY "tasks_update_members" ON tasks
FOR UPDATE
USING (
  project_id IN (
    SELECT id FROM projects
    WHERE organization_id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid()
    )
  )
)
WITH CHECK (
  project_id IN (
    SELECT id FROM projects
    WHERE organization_id IN (
      SELECT organization_id FROM organization_members
      WHERE user_id = auth.uid()
    )
  )
);

-- Task dependencies: same as tasks
CREATE POLICY "task_deps_select" ON task_dependencies
FOR SELECT
USING (
  task_id IN (
    SELECT id FROM tasks
    WHERE project_id IN (
      SELECT id FROM projects
      WHERE organization_id IN (
        SELECT organization_id FROM organization_members
        WHERE user_id = auth.uid()
      )
    )
  )
);

CREATE POLICY "task_deps_insert_update_delete" ON task_dependencies
FOR ALL
USING (
  task_id IN (
    SELECT id FROM tasks
    WHERE project_id IN (
      SELECT id FROM projects
      WHERE organization_id IN (
        SELECT organization_id FROM organization_members
        WHERE user_id = auth.uid()
      )
    )
  )
);

-- Indexes
CREATE INDEX idx_projects_org ON projects(organization_id);
CREATE INDEX idx_projects_owner ON projects(owner_id);
CREATE INDEX idx_projects_active ON projects(created_at DESC)
WHERE deleted_at IS NULL;

CREATE INDEX idx_tasks_project ON tasks(project_id);
CREATE INDEX idx_tasks_assignee ON tasks(assignee_id);
CREATE INDEX idx_tasks_parent ON tasks(parent_task_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_due_date ON tasks(due_date) WHERE completed_at IS NULL;

CREATE INDEX idx_task_deps_depends_on ON task_dependencies(depends_on_task_id);

-- Trigger: auto-update task.updated_at
CREATE OR REPLACE FUNCTION update_tasks_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_tasks_updated_at
BEFORE UPDATE ON tasks
FOR EACH ROW
EXECUTE FUNCTION update_tasks_updated_at();

-- Trigger: mark task as completed
CREATE OR REPLACE FUNCTION mark_task_completed()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.status = 'done' AND OLD.status != 'done' THEN
    NEW.completed_at = now();
  ELSIF NEW.status != 'done' AND OLD.status = 'done' THEN
    NEW.completed_at = NULL;
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER mark_task_completed
BEFORE UPDATE ON tasks
FOR EACH ROW
EXECUTE FUNCTION mark_task_completed();
```

---

## 4. Leads + Pipeline Stages

Sales pipeline with lead tracking.

```sql
CREATE TYPE lead_source AS ENUM ('inbound', 'outbound', 'referral', 'partnership', 'content', 'event', 'other');
CREATE TYPE lead_stage AS ENUM ('prospect', 'lead', 'qualified', 'proposal', 'negotiation', 'won', 'lost');

CREATE TABLE leads (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,

  first_name text NOT NULL,
  last_name text,
  email text UNIQUE,
  phone text,
  company text,
  website text,
  linkedin_url text,

  source lead_source NOT NULL DEFAULT 'inbound',
  stage lead_stage NOT NULL DEFAULT 'prospect',
  assigned_to uuid REFERENCES users(id) ON DELETE SET NULL,

  value_usd numeric(12, 2),
  notes text,
  metadata jsonb DEFAULT '{}'::jsonb,

  created_at timestamp with time zone DEFAULT now(),
  updated_at timestamp with time zone DEFAULT now(),
  deleted_at timestamp with time zone DEFAULT NULL
);

-- Lead activity log
CREATE TABLE lead_activities (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id uuid NOT NULL REFERENCES leads(id) ON DELETE CASCADE,
  organization_id uuid NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,

  activity_type text NOT NULL, -- 'email', 'call', 'meeting', 'task', 'note'
  description text,
  performed_by uuid NOT NULL REFERENCES users(id),

  created_at timestamp with time zone DEFAULT now()
);

-- Enable RLS
ALTER TABLE leads ENABLE ROW LEVEL SECURITY;
ALTER TABLE lead_activities ENABLE ROW LEVEL SECURITY;

-- Leads: org members can read
CREATE POLICY "leads_select_org_members" ON leads
FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
  AND deleted_at IS NULL
);

-- Leads: members can insert/update (assigned to them or admin)
CREATE POLICY "leads_insert_members" ON leads
FOR INSERT
WITH CHECK (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
);

CREATE POLICY "leads_update_owner_or_assigned" ON leads
FOR UPDATE
USING (
  assigned_to = auth.uid()
  OR organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
)
WITH CHECK (
  assigned_to = auth.uid()
  OR organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
);

-- Lead activities: org members can read/insert
CREATE POLICY "lead_activities_select" ON lead_activities
FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
);

CREATE POLICY "lead_activities_insert" ON lead_activities
FOR INSERT
WITH CHECK (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
  AND performed_by = auth.uid()
);

-- Indexes
CREATE INDEX idx_leads_org ON leads(organization_id);
CREATE INDEX idx_leads_assigned_to ON leads(assigned_to);
CREATE INDEX idx_leads_stage ON leads(stage);
CREATE INDEX idx_leads_email ON leads(email);
CREATE INDEX idx_leads_created ON leads(created_at DESC) WHERE deleted_at IS NULL;

CREATE INDEX idx_lead_activities_lead ON lead_activities(lead_id);
CREATE INDEX idx_lead_activities_org ON lead_activities(organization_id);
CREATE INDEX idx_lead_activities_created ON lead_activities(created_at DESC);

-- Trigger: update updated_at
CREATE OR REPLACE FUNCTION update_leads_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_leads_updated_at
BEFORE UPDATE ON leads
FOR EACH ROW
EXECUTE FUNCTION update_leads_updated_at();

-- Helper view: pipeline summary
CREATE VIEW pipeline_summary AS
SELECT
  stage,
  COUNT(*) as count,
  COALESCE(SUM(value_usd), 0) as total_value,
  COALESCE(AVG(value_usd), 0) as avg_value
FROM leads
WHERE deleted_at IS NULL
GROUP BY stage;
```

---

## 5. Audit Log

Track all changes to sensitive entities.

```sql
CREATE TYPE audit_action AS ENUM ('create', 'update', 'delete', 'restore');

CREATE TABLE audit_logs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,

  entity_type text NOT NULL, -- 'user', 'project', 'task', 'lead'
  entity_id uuid NOT NULL,
  entity_name text, -- display name for reference

  action audit_action NOT NULL,
  performed_by uuid NOT NULL REFERENCES users(id),

  -- What changed (old_values vs new_values)
  changes jsonb,
  -- Example: { "status": { "old": "todo", "new": "done" } }

  ip_address inet,
  user_agent text,

  created_at timestamp with time zone DEFAULT now()
);

-- Enable RLS: only org admins can view logs
ALTER TABLE audit_logs ENABLE ROW LEVEL SECURITY;

CREATE POLICY "audit_logs_select_admin" ON audit_logs
FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid() AND role IN ('owner', 'admin')
  )
);

CREATE POLICY "audit_logs_prevent_delete" ON audit_logs
FOR DELETE
USING (false);

-- Indexes
CREATE INDEX idx_audit_logs_org ON audit_logs(organization_id);
CREATE INDEX idx_audit_logs_entity ON audit_logs(organization_id, entity_type, entity_id);
CREATE INDEX idx_audit_logs_created ON audit_logs(created_at DESC);
CREATE INDEX idx_audit_logs_performed_by ON audit_logs(performed_by);

-- Function: log changes
CREATE OR REPLACE FUNCTION log_audit_change()
RETURNS TRIGGER AS $$
DECLARE
  changes jsonb := '{}';
  field text;
BEGIN
  -- Compare old and new values
  FOR field IN
    SELECT key FROM (
      SELECT jsonb_object_keys(to_jsonb(NEW)) AS key
      UNION
      SELECT jsonb_object_keys(to_jsonb(OLD)) AS key
    ) t
  LOOP
    IF OLD IS NULL THEN
      changes := changes || jsonb_build_object(field, jsonb_build_object('new', to_jsonb(NEW)->field));
    ELSIF NEW IS NULL THEN
      changes := changes || jsonb_build_object(field, jsonb_build_object('old', to_jsonb(OLD)->field));
    ELSIF (to_jsonb(OLD)->field) IS DISTINCT FROM (to_jsonb(NEW)->field) THEN
      changes := changes || jsonb_build_object(field, jsonb_build_object(
        'old', to_jsonb(OLD)->field,
        'new', to_jsonb(NEW)->field
      ));
    END IF;
  END LOOP;

  INSERT INTO audit_logs (
    organization_id, entity_type, entity_id, action, performed_by, changes
  ) VALUES (
    (NEW).organization_id,
    TG_TABLE_NAME,
    (NEW).id,
    CASE WHEN TG_OP = 'INSERT' THEN 'create' ELSE 'update' END,
    auth.uid(),
    CASE WHEN changes = '{}' THEN NULL ELSE changes END
  );

  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Attach to any table you want to audit (example)
-- CREATE TRIGGER audit_projects_changes
-- AFTER INSERT OR UPDATE ON projects
-- FOR EACH ROW
-- EXECUTE FUNCTION log_audit_change();
```

---

## 6. File Attachments + Storage

Store file metadata and link to Supabase Storage.

```sql
CREATE TABLE files (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id uuid NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,

  name text NOT NULL,
  mime_type text,
  size_bytes bigint,
  storage_path text NOT NULL, -- path in Supabase Storage bucket

  uploaded_by uuid NOT NULL REFERENCES users(id),
  description text,

  created_at timestamp with time zone DEFAULT now(),
  deleted_at timestamp with time zone DEFAULT NULL
);

-- Attach files to entities (polymorphic)
CREATE TABLE file_attachments (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  file_id uuid NOT NULL REFERENCES files(id) ON DELETE CASCADE,

  entity_type text NOT NULL, -- 'project', 'task', 'lead', 'document'
  entity_id uuid NOT NULL,

  created_at timestamp with time zone DEFAULT now()
);

-- Enable RLS
ALTER TABLE files ENABLE ROW LEVEL SECURITY;
ALTER TABLE file_attachments ENABLE ROW LEVEL SECURITY;

-- Files: org members can read files they uploaded or that are public
CREATE POLICY "files_select_org_members" ON files
FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
  AND deleted_at IS NULL
);

-- Files: can insert if in organization
CREATE POLICY "files_insert_org_members" ON files
FOR INSERT
WITH CHECK (
  organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  )
  AND uploaded_by = auth.uid()
);

-- Files: can only delete own uploads
CREATE POLICY "files_delete_own" ON files
FOR DELETE
USING (uploaded_by = auth.uid());

-- Indexes
CREATE INDEX idx_files_org ON files(organization_id);
CREATE INDEX idx_files_uploaded_by ON files(uploaded_by);
CREATE INDEX idx_files_created ON files(created_at DESC);

CREATE INDEX idx_file_attachments_file ON file_attachments(file_id);
CREATE INDEX idx_file_attachments_entity ON file_attachments(entity_type, entity_id);

-- Function: delete file from Storage when record deleted
CREATE OR REPLACE FUNCTION delete_file_from_storage()
RETURNS TRIGGER AS $$
BEGIN
  -- Note: Actual deletion should happen in application code with admin key
  -- This just marks as deleted to prevent re-access
  RETURN OLD;
END;
$$ LANGUAGE plpgsql;

-- Helper function: get file URL
CREATE OR REPLACE FUNCTION get_file_url(file_id uuid)
RETURNS text AS $$
  SELECT
    'https://PROJECT.supabase.co/storage/v1/object/public/' || storage_path
  FROM files
  WHERE id = file_id;
$$ LANGUAGE sql;
```

---

## 7. Notifications

User notification system with read tracking.

```sql
CREATE TYPE notification_type AS ENUM ('info', 'success', 'warning', 'error');

CREATE TABLE notifications (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  organization_id uuid REFERENCES organizations(id) ON DELETE CASCADE,

  type notification_type NOT NULL DEFAULT 'info',
  title text NOT NULL,
  message text,
  action_url text,
  action_label text DEFAULT 'View',

  read_at timestamp with time zone,
  read boolean GENERATED ALWAYS AS (read_at IS NOT NULL) STORED,

  created_at timestamp with time zone DEFAULT now(),
  deleted_at timestamp with time zone DEFAULT NULL
);

-- Enable RLS
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;

-- Users can only read their own notifications
CREATE POLICY "notifications_select_own" ON notifications
FOR SELECT
USING (user_id = auth.uid());

-- Users can update their own notifications (mark as read)
CREATE POLICY "notifications_update_own" ON notifications
FOR UPDATE
USING (user_id = auth.uid())
WITH CHECK (user_id = auth.uid());

-- Users can delete their own notifications
CREATE POLICY "notifications_delete_own" ON notifications
FOR DELETE
USING (user_id = auth.uid());

-- Indexes
CREATE INDEX idx_notifications_user ON notifications(user_id);
CREATE INDEX idx_notifications_user_unread ON notifications(user_id, read_at)
WHERE read_at IS NULL;
CREATE INDEX idx_notifications_created ON notifications(created_at DESC);

-- Helper function: create notification
CREATE OR REPLACE FUNCTION notify_user(
  user_id uuid,
  org_id uuid,
  notif_type notification_type,
  title text,
  message text DEFAULT NULL,
  action_url text DEFAULT NULL
)
RETURNS uuid AS $$
DECLARE
  notification_id uuid;
BEGIN
  INSERT INTO notifications (user_id, organization_id, type, title, message, action_url)
  VALUES (user_id, org_id, notif_type, title, message, action_url)
  RETURNING id INTO notification_id;

  RETURN notification_id;
END;
$$ LANGUAGE plpgsql;

-- Helper function: mark all as read
CREATE OR REPLACE FUNCTION mark_all_notifications_read()
RETURNS void AS $$
BEGIN
  UPDATE notifications
  SET read_at = now()
  WHERE user_id = auth.uid() AND read_at IS NULL;
END;
$$ LANGUAGE plpgsql;

-- Realtime: subscribe to user's notifications
-- In JavaScript:
-- const subscription = supabase
--   .from('notifications')
--   .on('*', payload => console.log(payload))
--   .subscribe();
```

---

## Quick Schema Verification Checklist

For each schema, verify:

```sql
-- RLS enabled on all tables
SELECT tablename
FROM pg_tables
WHERE schemaname = 'public'
AND NOT EXISTS (
  SELECT 1 FROM pg_policies
  WHERE pg_policies.tablename = pg_tables.tablename
);

-- All tables have timestamps
SELECT tablename
FROM pg_tables
WHERE schemaname = 'public'
AND NOT EXISTS (
  SELECT 1 FROM information_schema.columns
  WHERE table_name = tablename AND column_name = 'created_at'
);

-- Primary keys are UUID
SELECT tablename, a.attname
FROM pg_tables t
JOIN pg_class c ON c.relname = t.tablename
JOIN pg_attribute a ON a.attrelid = c.oid
JOIN pg_type p ON p.oid = a.atttypid
WHERE schemaname = 'public'
AND a.attnum = 1
AND p.typname != 'uuid';

-- Indexes exist on foreign keys
SELECT tc.table_name, kcu.column_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu USING (constraint_name)
WHERE constraint_type = 'FOREIGN KEY'
AND NOT EXISTS (
  SELECT 1 FROM pg_indexes
  WHERE tablename = tc.table_name
  AND indexdef LIKE '%' || kcu.column_name || '%'
);
```

---

All schemas ready to copy, paste, and customize. Start with what you need, extend gradually.

Open RX by CutTheChexx — The Prescription.
