# Platform Rules

Translate the user's governance request into repository settings that can actually be configured.

## GitHub

Map the request to:

- branch protection rule for `main`
- disallow force push
- disallow deletion
- require pull request before merging
- require status checks to pass
- require approvals
- restrict who can push or merge if the plan supports it
- use CODEOWNERS or team-based review assignment when applicable

Suggested interpretation of the user's requirement:

- direct push to `main`: disabled
- merge path: PR from `stg` into `main`
- allowed approvers: PO team or circle lead team
- auto deploy: workflow on `push` to `main`

## GitLab

Map the request to:

- protected branch `main`
- allowed to push: no one or maintainers only for emergency
- allowed to merge: specific users, group, or maintainers matching PO and circle lead
- merge request approvals required
- pipeline must succeed before merge
- deploy job triggered on `main`

## When Role Names Are Too Vague

If the user says only "PO" or "trưởng circle", convert that into one of:

- GitHub team slugs
- GitHub usernames
- GitLab group or subgroup
- GitLab usernames
- CODEOWNERS entries

If identity mapping is unavailable, keep the policy wording but flag the missing mapping as the only unblocker.
