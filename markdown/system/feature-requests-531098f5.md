# Feature requests

Created: 2026-05-31T19:08:10Z

## Note

This document collects product and architecture ideas that would make Mianotes better, but are not part of the current release scope.

## Workspace membership

**Big Gaps**

1. **Switch check must be atomic**
   Don’t do “check permission, then switch” as two trusted steps. The backend switch endpoint itself should resolve the workspace and either switch or return `403`. The frontend can pre-check for UX, but the backend switch must be authoritative.

2. **Every workspace-scoped endpoint needs protection**
   Not only workspace switching. Protect:
   `notes`, `folders`, `tags`, `jobs`, `sources`, `publish`, file downloads, markdown/source serving, search, context/MCP endpoints, and any direct `/workspace/&lt;slug&gt;/...` route.

3. **Agent/API access needs a decision**
   Mianotes has API keys and MCP. If there is one global API key, then membership can be bypassed unless agent sessions have a user/admin/system identity. Decide:
   - API key acts as admin/system and can access all workspaces, or
   - API requests must include a user/session and respect membership.
   
   This is probably the biggest hidden security edge case.

4. **“Main” should not be name-based**
   Don’t rely on workspace name `"Main"`. Use a stable flag or ID, e.g. `is_default = true`, because users may rename it.

5. **Workspace membership table**
   The plan implies it but doesn’t name it. You likely want this in `system.db`:
   `workspace_memberships(user_id, workspace_id, created_at, created_by)`
   
   Unique constraint on `(user_id, workspace_id)`. Main can either be implicit access or seeded membership for every user. I’d prefer implicit access to Main, because it avoids sync bugs.

**Edge Cases**

- If a user’s active workspace is removed from their access, their session should fall back to Main.
- If a user has a direct URL to a protected note, backend returns `403`, not `404`, unless you intentionally want to hide existence.
- If a workspace is archived/unavailable while someone is using it, decide whether access is denied, read-only, or redirected to Main.
- If the last admin removes their own access to a workspace, is that allowed? Probably yes for content access, but not relevant if admins can manage all workspaces globally.
- If an admin removes access while a job is running, decide whether the job continues. I’d let jobs continue because they belong to workspace state, not user visibility.
- If a workspace is renamed, membership should remain attached to workspace ID, not slug/name.
- If two admins add/remove at the same time, API should be idempotent: `PUT membership` and `DELETE membership` are better than action-style POSTs.
- Autocomplete must not leak unavailable/archived workspaces to normal users.

**UI Notes**

The normal-user “Workspace from their own 3-dot menu” may be confusing if non-admins don’t normally see Users. This might belong in their profile menu or workspace switcher instead.

For the protected workspace modal, `{admin_name}` is tricky if there are multiple admins or no named admin. Safer copy:

> This is a protected workspace. Please contact an admin to request access.

**Testing Additions**

Add tests for direct API calls, not only switch:

- blocked `GET /workspace/&lt;slug&gt;/notes`
- blocked note detail URL
- blocked search/context/MCP access
- blocked source file/markdown access
- active workspace fallback when access is removed
- archived workspace cannot be assigned
- Main access still works without explicit membership

The main change I’d make is to define a single backend “workspace resolver” dependency that resolves workspace + checks access before any workspace-scoped code runs. That keeps the rule central and makes the open-source code look intentional rather than permission checks scattered everywhere.
