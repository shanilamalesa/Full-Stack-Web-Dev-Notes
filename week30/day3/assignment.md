# Week 30 - Day 3 Assignment

## Title
Final Deploy -- Production Launch

## Overview
Day 3 is the production launch. You deploy to a real server, point a real domain at it, enable TLS from Let's Encrypt, and create the first real tenant.

## Learning Objectives Assessed
- Deploy the full stack to a production host
- Configure DNS for wildcard subdomains
- Obtain a TLS cert from Let's Encrypt
- Execute a launch checklist

## Prerequisites
- Days 1-2 completed

## AI Usage Rules

**Ratio this week:** 20% manual / 80% AI
**Habit:** Launch week. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Config generation.
- **NOT ALLOWED FOR:** DNS changes, production secrets, the go/no-go call.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Provision host

**What to do:**
Pick a host (Hetzner, DigitalOcean, Fly.io, Render). Provision a small instance. Install Docker and docker-compose.

Document the setup in `infra/PROVISION.md`.

**Expected output:**
Host ready.

### Task 2: DNS

**What to do:**
Buy (or use) a domain. Point `@` and `*` to the host's IP with A records. Verify with `dig`.

**Expected output:**
DNS resolving.

### Task 3: TLS via Let's Encrypt

**What to do:**
Use `acme-companion` in compose, or standalone certbot:

```bash
certbot certonly --standalone -d yourdomain.dev -d '*.yourdomain.dev' --preferred-challenges dns
```

Nginx picks up the cert.

**Expected output:**
`https://yourdomain.dev` shows a green padlock. Subdomains work.

### Task 4: First real tenant

**What to do:**
Go through signup from a browser on a real phone. Create a real tenant with a real subdomain. Log in. Hit the dashboard.

Take screenshots of each step.

**Expected output:**
`day3-launch/*.png`.

### Task 5: Launch checklist

**What to do:**
`docs/LAUNCH_CHECKLIST.md`:

```markdown
- [ ] Prod DB backed up
- [ ] TLS valid
- [ ] Healthcheck green
- [ ] Monitoring/logging in place
- [ ] Runbook accessible to someone other than me
- [ ] First real tenant created
- [ ] Payment sandbox tested end-to-end
- [ ] Known issues documented
- [ ] Rollback plan written
```

Tick honestly. Unchecked items mean you do not launch tomorrow.

**Expected output:**
Committed.

## Stretch Goals (Optional - Extra Credit)

- Automate certificate renewal with a cron.
- Add status page with uptimerobot.
- Wire Slack notifications for deploy failures.

## Submission Requirements

- **What to submit:** Repo, screenshots, checklist, provisioning notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Host provisioned | 15 | Documented. |
| DNS + wildcard | 15 | Resolving. |
| TLS valid | 20 | Subdomains included. |
| First real tenant | 25 | Screenshots. |
| Launch checklist | 20 | Honest and complete. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Wildcard cert via HTTP challenge.** You need DNS challenge.
- **Launching without backups.** Every launch needs a rollback.
- **Leaking real secrets into GitHub.** Use the host's secrets, not `.env` in repo.

## Resources

- Day 3 reading: [Final Deploy.md](./Final%20Deploy.md)
- Week 30 AI boundaries: [../ai.md](../ai.md)
- Let's Encrypt wildcard: https://letsencrypt.org/docs/challenge-types/#dns-01-challenge
