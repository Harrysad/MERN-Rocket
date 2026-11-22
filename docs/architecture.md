# Architecture

## Purpose

CRM Rocket is a portfolio project: a multi-tenant CRM for a B2B service company (an agency or a software house) with an admin, managers and sales reps. The demo tells one story in a few minutes:

> customer (by NIP) → deal in the pipeline → visits in the calendar → email with the customer → won deal becomes a project → results on the dashboard

It is a showcase, not a product: the public can only use the demo sandbox, and new organizations cannot sign up on the live instance (see [Demo sandbox](#demo-sandbox)).

## Overview

A **modular monolith**: one Spring Boot application split into modules whose boundaries are enforced by Spring Modulith. Modules communicate through events; background work (email delivery, reminders, sandbox cleanup) goes through an outbox table in PostgreSQL with retries.

Microservices are not used from the start: the system runs on a single small VPS, has one developer, and a monolith has a smaller attack surface. One module will be extracted into a separate service in a later stage.

```text
Browser (Angular PWA) ──HTTPS──> Caddy ──> Spring Boot app ──> PostgreSQL
                                                │
                                                ├──> email provider (outbound + inbound webhook)
                                                └──> Web Push
```

| Module | Responsibility |
| --- | --- |
| `identity` | organizations, users, teams, roles, sessions, 2FA, password reset, email verification, audit log |
| `customers` | customers, contact people, NIP lookup, interaction history, ownership |
| `pipeline` | deals, configurable stages, value and probability, won/lost reasons |
| `calendar` | visits, completion notes, reminders, iCal feed |
| `mail` | CRM mailboxes, threads linked to customers, sending, inbound webhook |
| `projects` | projects created from won deals, milestones, budget, hours |
| `notifications` | in-app notifications and Web Push |
| `dashboard` | aggregates and charts |
| `demo` | sandbox creation, seeding and expiry |

## Organizations and roles

An account is an **organization**. Each user belongs to exactly one organization and has one role; a manager leads one team of sales reps.

| Area | Admin | Manager | Sales rep |
| --- | --- | --- | --- |
| Users | creates, edits and deactivates users, assigns roles and teams, sends password reset links, signs users out | sees the team | own profile, password, 2FA |
| Customers, deals, projects | whole organization, reassignment | the team's records | own records |
| Calendar | everything | team calendar, schedules visits for the team | own calendar |
| Email | mailboxes and templates | team correspondence (read-only) | own correspondence |
| Dashboard | whole organization, team comparison | team results | own results |
| Security | audit log, password policy, mandatory 2FA, active sessions | – | – |

Rules enforced on the server:

- Every record has an owner; a visit can have additional participants, who see only that visit.
- An admin never sets or learns another user's password: a reset is a one-time link valid for 30 minutes that also ends the user's sessions.
- Nobody can grant a role higher than their own, and the last admin of an organization cannot be removed or demoted.
- Removing a user means deactivating them: their sessions end immediately and their records are reassigned.
- Authorization is checked on every request: organization, role, and ownership or team membership.

## Demo sandbox

- "Try the demo" creates a fresh organization with sample data and signs the visitor in as its admin. Inside a sandbox the visitor can switch to a sample manager or sales rep to see their views; this switch does not exist in real organizations.
- A sandbox and all its data are deleted after 24 hours.
- Abuse protection: rate limits per IP and globally, bot protection, and no outgoing email from a sandbox – messages are shown in an in-app preview instead.
- Signing up new organizations is controlled by a server-side switch (`REGISTRATION_ENABLED`) that is off on the live instance and not exposed in the UI. A second switch (`OUTBOUND_MAIL_ENABLED`) stops all outgoing email at once, without a redeploy.

## Security

Target: OWASP ASVS level 2, and level 3 for authentication and tenant isolation where feasible. Every stage ends with a review from an attacker's point of view.

**Authentication and sessions**

- Server-side sessions (Spring Session JDBC) in a `__Host-` cookie that is `HttpOnly`, `Secure` and `SameSite=Lax`, with a CSRF token sent in a header. No tokens in browser storage.
- Passwords hashed with Argon2id, at least 12 characters, checked against known breached passwords.
- TOTP two-factor authentication with recovery codes; admins can make it mandatory.
- Rate limiting per IP and per account, identical responses for unknown accounts, session ID rotation on sign-in, inactivity timeout, a list of active sessions.

**Tenant isolation**

- `organization_id` on every business table, filtered in the application **and** enforced by PostgreSQL Row-Level Security as a second line of defence.
- Automated tests try to read and modify another organization's data through every endpoint.
- UUID identifiers.

**Input and output**

- Server-side validation, explicit field whitelists, parameterized queries only.
- Incoming email HTML is sanitized and rendered in a sandboxed iframe; webhooks are signature-verified, size-limited and idempotent.
- A strict Content Security Policy without `unsafe-inline`, HSTS and the other standard security headers.
- Errors in the RFC 9457 Problem Details format, without internal details.

**Supply chain and infrastructure**

- GitHub Actions pinned to commit SHAs, Dependabot, CodeQL, secret scanning (gitleaks locally and in CI, GitHub push protection), container image scanning (Trivy), SBOM, OWASP ZAP baseline scan.
- Minimal multi-stage images running as a non-root user.
- Only the reverse proxy (Caddy, automatic TLS) is reachable from the internet; the application and the database are not.
- Server: SSH keys only, no root login, firewall, automatic security updates, fail2ban, encrypted off-site database backups.
- Secrets live only in environment files on the server and in GitHub Environments, never in the repository.

## Deployment

Docker Compose on a Hetzner VPS behind Caddy. Pull requests run CI; merging to `main` publishes images to GitHub Container Registry and deploys them over SSH, followed by a health check.

## Roadmap

0. Repository foundation – hooks, branch protection, documentation
1. Skeleton – backend and frontend, Docker Compose, CI/CD
2. Identity and security – organizations, users, roles, sessions, 2FA, audit log
3. Demo sandbox
4. Customers
5. Sales pipeline
6. Calendar and visits
7. Projects
8. Notifications and PWA
9. Email on the project's own domain
10. Dashboard
11. Microsoft 365 email and calendar sync
12. One module extracted into a separate service

Production deployment on the VPS starts as soon as the server is available, whatever the current stage.