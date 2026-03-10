# Permissions / RBAC

## Generation Instructions
Permissions implements role-based access control: what authenticated users are allowed to do. Auth establishes identity (who you are); Permissions determines capability (what you can do). Do not conflate them. Do not generate login, session, or credential logic here.

Generate a role definition model from `config.roles`. Each role has a name and a list of permitted actions drawn from the application's action vocabulary. Generate role assignment — each user has one or more roles. Generate a role management admin UI for viewing and editing role assignments.

Generate permission check helpers available throughout the application: `can(user, action)` returns boolean; use these in route middleware, API handlers, and UI conditional rendering. If `config.hierarchical` is true, a role inherits all permissions of its declared parent role.

If `config.resource-scoped` is true, generate resource-level ownership checks in addition to role checks. A resource-scoped check verifies that the user owns or has been explicitly granted access to the specific record, not just the action in general. Generate `canAccess(user, resource)` helpers for this purpose.

Assign `config.default-role` to every new user on registration. Subscribe to `auth.registered` to perform this assignment.

Generate admin surfaces for each enabled entry in `config.admin-surface`.

capability: permissions
type: singleton
version: 1.0

depends-on:
  - auth
  - user-management

emits:
  - event: permissions.role-assigned
    payload:
      userId: string
      role: string
      assignedBy: string        # admin userId or 'system' for default assignment
  - event: permissions.role-removed
    payload:
      userId: string
      role: string
      removedBy: string

config:
  roles:
    - name: admin
      description: Full access to all operations
      permissions:
        - "*"                   # wildcard — all actions
    - name: member
      description: Standard user access
      permissions:
        - read:own
        - write:own
        - delete:own
    - name: viewer
      description: Read-only access
      permissions:
        - read:own

  default-role: member          # role assigned to new users on registration

  hierarchical: false           # if true, declare parent role on each role definition

  resource-scoped: false        # if true, generate per-record ownership checks

  admin-surface:
    role-list: true             # view all defined roles and their permissions
    assign-role: true           # assign or remove roles from a specific user
    view-user-roles: true       # see all roles assigned to a user
