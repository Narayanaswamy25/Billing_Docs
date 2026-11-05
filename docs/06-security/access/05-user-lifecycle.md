# User Lifecycle

## States
`Invited → Active → (Locked) → Disabled → (Removed)`. A membership has a status per company; an identity has its own (Active, Locked, Disabled).

| State | Meaning | Can sign in? |
| --- | --- | --- |
| Invited | Account created, link sent or temporary password issued; not accepted yet | No |
| Active | Normal | Yes |
| Locked | Too many failed attempts | No until the timer ends or an admin unlocks |
| Disabled | Left the company or paused by an admin; history stays | No |
| Removed | Personal details erased on request where the law allows; records keep a neutral name | No |

## 1. Create
1. Admin, or a Manager with `people.users.add`, opens People → Add user. The form lists only roles the creator may assign.
2. Fields: name, email **or** mobile (see 01), role, branches (within the creator's), reports to (defaults to the creator for managers), start date.
3. The form says: "Will see: Sales, Stock. Starts on Sales."
4. The server checks: creator's level is above the new role's level, role is in the creator's assignable list, branches within the creator's, seat available (reserved now), and the creator holds every verb and scope in the role.
5. Email login: an invitation link (256-bit random token, stored hashed, valid 72 hours, single use) is emailed. Mobile-only: a temporary password is shown once to the creator.
6. Audit event `user.invited`.

## 2. Accept
- New identity: person opens the link, sets a password (and enrols two-step if required), confirms their name; membership becomes Active.
- Existing identity (already works for another company): person signs in, accepts, and gets a company picker next time. No second password.
- Expired or used link: "This link has expired. Ask your admin to send a new one." Resending invalidates the old link.

## 3. Reporting line and Team
Every user has `reportsTo` (a user in the same company). **Team = everyone below you in that tree, direct and indirect, within your branches.** Managers see Team scope in lists, Activity and approvals. Changing someone's `reportsTo` needs `people.users.change` for both people's Teams; the Activity tab shows history.

## 4. Change role, branches or personal changes
| Change | Who | Effect |
| --- | --- | --- |
| Role | Admin only (a manager cannot change a role) | Applies at the next request; open forms finish first |
| Branches | Admin; Manager inside their own branches for people in their Team | Lists stop showing other branches; scope recomputed |
| Personal change (add or remove one verb or field mode) | Admin | Shown with a badge; listed in Personal changes; reviewable |
| Limits | Admin | Per role or per person |
A person cannot change their own role, branches, scope or limits (no self-promotion).

## 5. Someone leaves
1. Admin (or a manager for their Team) **disables** the person. Their sessions, refresh tokens and registered devices are revoked at once, the seat is freed and the identity's other memberships are untouched.
2. The screen lists what the person leaves behind: held bills, open drafts, pending approvals, assigned documents, unsent Counter bills.
3. Choose **Reassign** to a named user (held bills, drafts, approvals, reports-to for their direct reports). Unsynced bills on a Counter stay and sync under the person's name; if they were sold after the disable time they are flagged Authorization conflict.
4. All past records keep the person's name and the audit trail keeps every action; nothing is deleted.
5. Rejoining: admin sets Active again (or sends a new invitation); the old role is restored only if the admin chooses.

## 6. Erasure and removal
A removal request (the person's or the company's) is handled under the privacy rules: personal details are erased or replaced with "Former user"; statutory records, audit entries and posted documents keep a pseudonymous reference where the law requires retention. The decision is made by the company's privacy lead with the lawyer's rules (see 47-privacy-compliance).

## 7. Company-level protections
| Rule | Behaviour |
| --- | --- |
| Last admin | Cannot be disabled, demoted or removed. To hand over, invite a second admin, wait for acceptance, then step down |
| Admin recovery | See 06; super admin cannot just set an admin's password |
| Seat limit | Invite blocked with "Plan limit reached. Ask your admin." |
| Duplicate person | Same email or mobile already in the company: "Already added. Open this person." |
| Cross-branch moves | A person with Own scope keeps only their own records after a move; Branch scope follows the new branches |
| Bulk onboarding | Import of users is `people.users.add` plus `import`, with the same checks per row |

## 8. Changes to people are tracked and revertible
Role, branch, personal-change and status changes are change events with revisions; the admin can restore an earlier version (step-up, preview of who is affected). See [history-and-backup/02](../../05-logic/history-and-backup/02-revert-and-restore.md).

## 9. Events written to the audit log
`user.invited`, `user.invitation_resent`, `user.accepted`, `user.signed_in`, `user.sign_in_failed`, `user.locked`, `user.unlocked`, `user.password_changed`, `user.mfa_enrolled`, `user.mfa_reset`, `user.role_changed`, `user.branches_changed`, `user.personal_change_added/removed`, `user.limits_changed`, `user.disabled`, `user.reactivated`, `user.reassigned`, `user.erased`, `session.revoked`, `device.registered/approved/revoked`, `reveal.used`, `step_up.used`, `offline_grant.issued`, `authorization_conflict.flagged`. Each has who, when, device, branch, before and after.
