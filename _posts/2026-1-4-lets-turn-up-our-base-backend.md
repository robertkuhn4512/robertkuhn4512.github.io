---
layout: post
title: "Let's build a basic ish backend!"
tags: [intro, architecture, keycloak, vault, postgres, docker]
---

## New Year, New Backend!

This blog is the build log for the *Network Tools* backend—where the goal is not just “make it run,” but to build a foundation that is repeatable, secure by default, and easy to extend as the platform grows.

## The Foundation Stack (What We’re Standardizing On)

### Keycloak — Authentication and Identity
Keycloak will be the authentication layer for both the frontend and our FastAPI backend. The frontend direction is Symfony/PHP (Unless someone can suggest anything fun to try for that stack) tied into Keycloak. The bigger picture is that Keycloak can also integrate with other authentication methods later and act as a “go-between” so the applications don’t need to reinvent auth.

**TODO**
- Setup a permanent admin account and a FastAPI service account
- Harden local authentication (2FA / security keys)
- I'm sure I'm missing something else, but I'll play it by ear as I go!

### HashiCorp Vault — Credential Storage and Policy Enforcement
Vault is the authoritative source for secrets and sensitive configuration. The intent is to reduce the footprint of static secrets (in repos, images, or long-lived config files) and to tighten access through policies.

**TODO**
- Harden the installation further
- Revoke/lock down the root token when not needed, and minimize operational dependence on it

### Vault Agents — Runtime Secret Delivery
Vault Agents are the delivery mechanism: containers and services should receive credentials at runtime rather than embedding them in Compose, container layers, or `.env` files that may or may not sit around in someone's repo..

The long-term principle: **services should not need to “know Vault” to use Vault**—they should just consume rendered outputs safely.

### PostgreSQL — Production Database
Postgres is the production database layer. The objective is to keep it predictable and controlled:
- explicit access rules
- secure connectivity (TLS where appropriate)
- repeatable initialization and bootstrap

### pgAdmin — Operational UI
pgAdmin is the operator-friendly frontend to Postgres for the days when you don’t want to live entirely in CLI.

---

## Lessons Learned So Far

1. **“Works once” is not a milestone—repeatability is.**  
   If the environment can’t be rebuilt cleanly, it’s not done. Scripts, seed steps, and configuration should be treated as product—not tribal knowledge.
<br><br>
2. **Hostnames and certificates are not details—they are dependencies.**  
   If a service advertises one hostname but certificates and clients assume another, you lose hours to problems that look like “the app is down” but are really trust/identity mismatches.
<br><br>
3. **Secret delivery is an architecture decision, not a convenience feature.**  
   Vault is only half the solution. The workflow that gets secrets into containers (agents, rendered files, rotation strategy) determines whether the system is maintainable or fragile.
<br><br>
4. **Small permissions mistakes create loud failures.**  
   Database connection rejections (e.g., access rules) and secret auth failures are painful—but they’re also the system doing what you asked. The lesson is to invest early in clear policy, clear logging, and minimal-permission debugging paths.
<br><br>
5. **“Operator usability” must be designed in.**  
   Tools like pgAdmin are not a compromise; they’re part of how you keep momentum while still enforcing strong standards (no credential spr
