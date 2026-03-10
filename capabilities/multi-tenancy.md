# Multi-Tenancy

## Generation Instructions
Multi-Tenancy introduces organization-level data isolation: users belong to one or more organizations, and all data is scoped to the organization. Use the term declared in `config.org-term` throughout all generated code, UI copy, API routes, and database schema — never hardcode "organization."

Generate an Organization entity with fields: id, name, slug (URL-safe identifier), logo_url, created_at, created_by. Generate an org creation and setup flow. If `config.org-creation` is `self-serve`, any authenticated user can create an org. If `admin-provisioned`, only admin users can create orgs.

Generate org-scoped data access middleware: all database queries for scoped entities automatically include a filter for the current org. Generate middleware that identifies the current org from the request (subdomain, path segment, or session — use subdomain by default) and injects it into the request context.

Generate a member invitation flow: send an invitation email to a new member (connects to Mailer if present), generate an acceptance URL, and on acceptance add the user to the org with their assigned member role. Generate member management UI: list members, view their org role, change their role, and remove them.

Generate member roles within the org from `config.member-roles`. These are distinct from application-level roles in the Permissions capability — org roles govern what a user can do within the org (invite members, manage settings), not what they can do in the application.

If `config.multi-org-membership` is true, generate an org switcher UI that allows users to switch between their orgs. Store the active org in session.

If `config.data-isolation` is `row-level`, add an `org_id` foreign key to all scoped entity tables and enforce it in queries. If `schema-level`, generate a separate database schema per org.

capability: multi-tenancy
type: singleton
version: 1.0

depends-on:
  - auth
  - user-management
  - permissions

emits:
  - event: multi-tenancy.org-created
    payload:
      orgId: string
      createdBy: string
  - event: multi-tenancy.member-invited
    payload:
      orgId: string
      email: string
      invitedBy: string
  - event: multi-tenancy.member-joined
    payload:
      orgId: string
      userId: string
  - event: multi-tenancy.member-removed
    payload:
      orgId: string
      userId: string
      removedBy: string
  - event: multi-tenancy.org-deleted
    payload:
      orgId: string
      deletedBy: string

config:
  org-term: org                 # org | team | workspace | company | account

  multi-org-membership: false   # allow users to belong to more than one org

  org-creation: self-serve      # self-serve | admin-provisioned

  member-roles:                 # roles within an org — distinct from app-level Permissions roles
    - name: owner
      description: Full control of the org including deletion
    - name: admin
      description: Manage members and settings
    - name: member
      description: Standard member access

  data-isolation: row-level     # row-level | schema-level

  admin-surface:
    org-list: true              # view and manage all orgs
    org-impersonation: true     # view the application as a specific org (requires Impersonation)
