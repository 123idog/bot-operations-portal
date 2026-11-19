# 5. Permissions in the Database

The portal is a static web app. Anyone can open its JavaScript and call the database API directly. So **every rule lives in the database**, and the UI only hides buttons the user couldn't use anyway.

## The model

```text
profiles      one per signed-in user (created by a trigger on sign-up), active flag
memberships   user_id, department_id, role (team_lead | viewer)
is_admin      flag on the profile
```

- Sign-in is Google SSO. **Signing in grants nothing.** A new user sees an empty portal until an admin adds a membership.
- Row-level security on every table uses two helpers: `can_see_dept(dept)` and `can_edit_bot(bot)`. Both also require the profile to be **active**, so removing a user takes effect on their next request, without waiting for sessions to expire.
- Writes go only through `security definer` functions that check those helpers and validate input. Tables have no insert or update policies for normal users.

## Files follow the same rule

Screenshots attached to problem reports and catalog photos live in private storage buckets. The object path starts with the department id (`<department>/<uuid>.png`), and storage policies check `can_see_dept` on that first folder. Files are shown through signed URLs that expire after an hour.

## The default grants that almost bit

The platform grants `EXECUTE` on every new function in the public schema **directly** to the anonymous and authenticated roles. The usual hardening, `revoke execute ... from public`, removes the grant to `PUBLIC` but leaves those direct grants in place. Internal helper functions were callable by anyone with the public API key.

The fix was explicit:

```sql
revoke execute on function internal_helper(...) from public, anon, authenticated;
alter default privileges in schema public revoke execute on functions from anon, authenticated;
```

The same default grants included `TRUNCATE`, `TRIGGER` and `REFERENCES` on tables to the client roles. Row-level security does not apply to `TRUNCATE`. Those were revoked too.

**Rule: after every schema change, run the platform's security advisor and read every warning.** Keep a short list of warnings that are intentional, with the reason next to each.

## Removing a user

Users with history can't be hard-deleted (their id is in the audit log and in requests). "Remove" sets the profile inactive and deletes all memberships, so access stops immediately and history stays intact. The function refuses to remove the last admin or yourself.

## Why not a document database

It was suggested. For this system it would have meant writing and securing an API server, an auth integration and a permission layer, then keeping all three patched, as one person. Row-level security, auth and storage built into the database removed that entire layer. The trade-off was learning the platform's defaults well, which is what this chapter is about.
