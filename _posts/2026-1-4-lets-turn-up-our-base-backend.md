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


## Below is the current file structure we are left with currently and brief notes of interest

```bash
|-- backend
|   |-- app
|   |   |-- keycloak
|   |   |   |-- bin
|   |   |   |   `-- keycloak_entrypoint_from_vault.sh
|   |   |   |-- certs
|   |   |   |   |-- ca.crt
|   |   |   |   |-- ca.key -> Remove / Store somewhere safe
|   |   |   |   |-- ca.srl -> Remove / Store somewhere safe
|   |   |   |   |-- cert.crt
|   |   |   |   `-- cert.key
|   |   |   `-- vault_agent -> Templates and such used to build the vault agent environments
|   |   |       |-- agent.hcl
|   |   |       |-- keycloak_agent_policy.hcl
|   |   |       `-- templates
|   |   |           |-- keycloak.env.ctmpl
|   |   |           |-- keycloak_tls.crt.ctmpl
|   |   |           `-- keycloak_tls.key.ctmpl
|   |   |-- pgadmin
|   |   |   `-- certs
|   |   |       |-- ca.crt
|   |   |       |-- ca.key -> Remove / Store somewhere safe
|   |   |       |-- ca.srl -> Remove / Store somewhere safe
|   |   |       |-- cert.crt
|   |   |       `-- cert.key
|   |   |-- postgres
|   |   |   |-- certs
|   |   |   |   |-- ca.crt
|   |   |   |   |-- ca.key -> Remove / Store somewhere safe
|   |   |   |   |-- ca.srl -> Remove / Store somewhere safe
|   |   |   |   |-- cert.crt
|   |   |   |   `-- cert.key
|   |   |   |-- config
|   |   |   |   |-- pg_hba.conf
|   |   |   |   `-- postgres.conf
|   |   |   `-- vault_agent -> Templates and such used to build the vault agent environments
|   |   |       |-- agent.hcl
|   |   |       `-- templates
|   |   |           |-- pgadmin_password.ctmpl
|   |   |           |-- postgres_db.ctmpl
|   |   |           |-- postgres_password.ctmpl
|   |   |           |-- postgres_user.ctmpl
|   |   |           `-- servers.json.ctmpl
|   |   |-- routers
|   |   `-- security
|   |       `-- configuration_files
|   |           `-- vault
|   |               |-- bootstrap -> These are the initial files created on bootstrap. Remove these and store securly off server. 
|   |               |   |-- postgres_pgadmin_credentials.json
|   |               |   |-- postgres_pgadmin.env
|   |               |   |-- root_token
|   |               |   |-- root_token.json
|   |               |   |-- seeded_secrets_all.json
|   |               |   |-- seed_kv_spec.postgres_pgadmin.json
|   |               |   `-- unseal_keys.json
|   |               |-- certs
|   |               |   |-- ca.crt
|   |               |   |-- ca.key -> Remove / Store somewhere safe
|   |               |   |-- ca.srl -> Remove / Store somewhere safe
|   |               |   |-- cert.crt
|   |               |   `-- cert.key
|   |               |-- config
|   |               |   |-- acl_policies
|   |               |   |   |-- keycloak_read
|   |               |   |   `-- postgres_pgadmin_read.hcl
|   |               |   |-- certs
|   |               |   |-- keycloak_kv_read.hcl
|   |               |   |-- postgres_pgadmin_kv_read.hcl
|   |               |   `-- vault_configuration_primary_node.hcl
|   |               `-- Dockerfile
|   |-- build_scripts
|   |   |-- generate_local_keycloak_certs.sh
|   |   |-- generate_local_pgadmin_certs.sh
|   |   |-- generate_local_postgres_certs.sh
|   |   |-- generate_local_vault_certs.sh
|   |   |-- generate_postgres_pgadmin_bootstrap_creds_and_seed.sh
|   |   |-- guides -> Quick notes written on how to build your own json files to autoseed into vault
|   |   |   |-- seed_kv_spec.example.json
|   |   |   `-- seed_kv_spec.GUIDE.md
|   |   |-- helper_scripts
|   |   |   |-- vault_unseal_kv_seed_bootstrap_rootless.sh -> Helper script to auto populate vaules to vault - Not really used in the latest but I left them here
|   |   |   `-- vault_unseal_multi_kv_seed_bootstrap_rootless.sh
|   |   |-- keycloak_approle_setup.sh
|   |   |-- postgress_approle_setup.sh
|   |   |-- seed_postgres_with_vault_credentials.sh
|   |   |-- startover_scripts -> Nuke your developent server to easily start over! (Remove from production deployments)
|   |   |   `-- reset_network_tools_docker.sh
|   |   |-- validation_scripts -> A few quick scripts to check some odds and ends via bash and cli.
|   |   |   |-- postgres_inventory.sh
|   |   |   `-- read_postgres_pgadmin_approle.sh
|   |   |-- vault_first_time_init_only_rootless.sh
|   |   |-- vault_unseal_kv_seed_bootstrap_rootless.sh
|   |   `-- vault_unseal_multi_kv_seed_bootstrap_rootless.sh
|   `-- nginx
|-- container_data
|   `-- vault
|       |-- approle -> These are one-shot secret_ids used to populate the vault agents
|       |   |-- keycloak_agent
|       |   |   |-- role_id
|       |   |   `-- secret_id
|       |   `-- postgres_pgadmin_agent
|       |       |-- role_id
|       |       `-- secret_id
|       `-- data -> Host OS Mapped files which make backups slightly easier, and prevents issues if the containers flake
|           |-- logs
|           |   `-- audit.log -> Host OS Mapped audit log file
|           |-- raft
|           |   |-- raft.db
|           |   `-- snapshots
|           `-- vault.db
|-- docker-compose.prod.yml -> This is the main docker compose file
|-- frontend
|-- how_to_videos (I will be moving these to the Tube, But this is mainly on my dev setup)
|   |-- Step 1 Walkthroughs
|   |   |-- 1_HOW_TO_Generate Keycloak Self Signed Certificate Files.mov
|   |   |-- 2_HOW_TO_Generate PgAdmin Self Signed Certificate Files.mov
|   |   |-- 3_HOW_TO_Generate Postgres Self Signed Certificate Files.mov
|   |   `-- 4_HOW_TO_Generate Vault Self Signed Certificate Files.mov
|   `-- Step 2 Walkthroughs
|       |-- Step 1
|       |   |-- 1_HOW_TO_First_Time_Vault_Init.mov
|       |   `-- 2_HOW_TO_First_Time_Vault_Credential_Seeding.mov
|       `-- Step 2
|           |-- 1_HOW_TO_First_Time_Checking_Vault_GUI_Verification.mov
|           `-- 2_HOW_TO_First_Time_Checking_Vault_Credential_Seeding.mov
|-- README.full.md -> Full walkthroughs on specific tasks (May or may not be organized nicely, please reach out if you see a way to improve it)
`-- README.md -> Brief rundown on how to get your containers up and prepped
```