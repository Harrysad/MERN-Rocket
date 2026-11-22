# CRM Rocket

Multi-tenant B2B CRM for service companies, built with Angular, Spring Boot and PostgreSQL.

> **Status: under construction.** The repository foundation is in place; the application is built stage by stage (see the [roadmap](docs/architecture.md#roadmap)).

CRM Rocket is the successor of [MERN CRM](https://github.com/Harrysad/MERN-CRM): the same core (customers, interaction history, company lookup by NIP) rebuilt for a team instead of a single user, with security designed in from the first commit.

## Planned features

- **Organizations and roles** – admin, manager and sales rep, with data isolated per organization
- **Customers** – company autofill by NIP from Poland's VAT payer registry, interaction history, assignment to sales reps
- **Sales pipeline** – deals moving through configurable stages, won or lost with a reason
- **Calendar and visits** – visits marked as completed with a note, iCal feed for external calendars
- **Email** – correspondence with a customer kept on the customer's page, sending from the CRM
- **Projects** – a won deal becomes a project with milestones, budget and logged hours
- **Notifications** – in-app and push notifications on the phone (PWA)
- **Dashboard** – pipeline, conversion, revenue forecast and team activity charts
- **Demo sandbox** – every visitor gets a fresh organization with sample data, deleted after 24 hours

The user interface is in Polish.

## Tech stack

| Area | Technologies |
| --- | --- |
| Backend | Java 25, Spring Boot, Spring Modulith, Spring Security, Spring Data JPA, Flyway, Maven |
| Database | PostgreSQL |
| Frontend | Angular, Angular Material, Apache ECharts, PWA |
| Testing | JUnit 5, Testcontainers, Playwright |
| DevOps | Docker, Docker Compose, GitHub Actions, GitHub Container Registry, Caddy, Hetzner VPS |

## Repository structure

```text
MERN-Rocket/
├── .githooks/            # commit-msg, pre-commit and pre-push hooks
├── .github/rulesets/     # main branch protection, kept as code
├── docs/architecture.md  # architecture and security model
├── scripts/              # hook tests
└── SECURITY.md           # how to report a vulnerability
```

## Getting started

Requirements: Git and [gitleaks](https://github.com/gitleaks/gitleaks) (the pre-commit hook refuses to commit without it).

```bash
git clone https://github.com/Harrysad/MERN-Rocket.git
cd MERN-Rocket
git config core.hooksPath .githooks
sh scripts/test-hooks.sh
```

## Development workflow

- `main` is protected: changes land only through pull requests, merged with squash.
- Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/), e.g. `feat(customers): add NIP lookup`, with a subject of up to 72 characters.
- Git hooks run locally:
  - `commit-msg` – message format, no attribution trailers
  - `pre-commit` – no commits on `main`, no `.env` files, no secrets (gitleaks)
  - `pre-push` – no pushes to `main`, runs the test suites

## Security

The security model is described in [docs/architecture.md](docs/architecture.md#security). To report a vulnerability, see [SECURITY.md](SECURITY.md).

## License

No license is granted. The code is public for review only; all rights reserved.
