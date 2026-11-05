# Notifications and Email/SMS Templates

**Status: triggers and channels are fixed; message wording needs the owner.** Templates for WhatsApp and SMS must be approved by the provider/regulator; keep each short, with named variables, and store them in `templates_rules` per company and language (English, Hindi, Tamil). Every send is logged on the record.

## In-app notifications (notification centre)
| Notice | Trigger | Who sees it | Can the person turn it off? |
| --- | --- | --- | --- |
| Approval waiting | Rule fires | Approver | In-app only/push/email choice; admin-set alerts not off for the admin |
| Approval decided | Approve/Reject | Requester | Yes |
| Failed e-invoice | Portal error after retries | Invoice owner + admin | Not for the admin |
| E-invoice nearing reporting window | Daily job | Admin, accountant | Not for the admin |
| E-way bill expiring | Daily | Admin, billing officer | Yes |
| Low stock / near expiry | Daily | Stock roles | Yes |
| Sync problem / bill conflict | Counter sync | Manager, admin | Not for the admin |
| Access changed | Role or personal change | The person | No |
| Export ready | Background export | Requester | No |
| Mention in Notes | At-mention | Mentioned user (only if they may see the record) | Yes |
| Plan limit at 80% / 100% | Usage meter | Admin | Not for the admin |
| Grace period / suspension | Billing state | Admin | No |
| Sensitive change (price, delete, role, settings, bulk, reveal, export) | Change event rule | Admin, manager | Not for the admin |
| Change reverted | Revert | Original editor, admin | No |
| Backup failed, backup not verified in 45 days | Backup job | Admin | No |
| Restore point created | Risky action | Admin | Yes |

## Reminders
Full catalogue, rules and controls: [advanced/03-restocking-and-reminders.md](../05-logic/advanced/03-restocking-and-reminders.md). Customer-facing reminders use approved templates and only consented channels.

## Outbound messages (per company, optional)
| Message | Trigger | Channel | Variables |
| --- | --- | --- | --- |
| Invoice / receipt share | Send action | WhatsApp, email, SMS link, PDF | company, number, total, link |
| Payment reminder | Send reminder or automatic on overdue | WhatsApp, SMS | customer, number, balance, due date |
| Statement | Send statement | Email, WhatsApp | customer, period, link |
| User invitation | Add user | Email | company, role, link, expiry |
| Password reset | Request | Email | link, expiry |
| Support access request/started | Support mode | Email to admin | who, reason, expiry |
| Offer / birthday campaigns | Schedule (later) | WhatsApp, SMS | — |

## Rules
- Email domain has SPF, DKIM and DMARC so invitation and reset emails cannot be forged.
- Invitation and reset tokens: random, shown once, stored as SHA-256 only, with an expiry.
- A message never contains data the recipient is not allowed to see; scheduled report emails contain only what the recipient may see.
- Message credits are counted per plan; messaging provider is per company (D-013 equivalent for messaging: not chosen).

## To fill (owner)
Exact wording per message and language; sender names; reminder cadence; quiet hours.
