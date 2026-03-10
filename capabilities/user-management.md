# User Management

## Generation Instructions
User Management owns user profile data and account lifecycle after identity is established. It does not own credentials, sessions, or login — those belong to Auth. Auth establishes who you are; User Management manages everything else about you.

Generate a User entity with the following base fields: id, email, name, avatar_url, created_at, updated_at, status. The email field is owned by Auth and is read-only here. Status values are: active, pending, deactivated. Other capabilities extend this entity by adding their own fields; User Management defines the base schema.

Generate user-facing profile surfaces: a profile edit form for fields declared in `config.user-editable-fields`, avatar upload if `config.avatar-upload` is true, and an account deletion request flow. If `config.self-serve-deletion` is true, the user initiates deletion directly and the account is deleted after a configurable grace period. If false, the user submits a deletion request that requires admin approval.

Generate admin surfaces for each enabled entry in `config.admin-surface`. Admin profile editing covers all fields regardless of `config.user-editable-fields`.

Do not generate role or permission assignment here — that belongs to the Permissions capability.

capability: user-management
type: singleton
version: 1.0

depends-on:
  - auth

emits:
  - event: user-management.profile-updated
    payload:
      userId: string
      changedFields: string[]
  - event: user-management.deleted
    payload:
      userId: string

config:
  user-editable-fields:         # fields users can edit on their own profile
    - name
    - avatar_url

  avatar-upload: true           # enable avatar image upload (requires File Management or inline handling)

  deletion:
    self-serve: true            # if false, deletion requires admin approval
    grace-period-days: 7        # days before account is permanently deleted after request

  admin-surface:
    user-list: true             # paginated, searchable user list
    view-profile: true          # view full profile and account details
    edit-profile: true          # edit any profile field
    change-status: true         # set user status to active | deactivated
    export: true                # export user list to CSV
