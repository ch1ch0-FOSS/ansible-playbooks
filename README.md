# ansible-playbooks

Infrastructure as Code for complete production stack deployment. Bare metal to production in 30 minutes with reproducible, tested playbooks.

## What This Demonstrates

**For DevOps Hiring Managers:**

IaC expertise and reproducibility mindset. Role-based playbook organization with proper error handling. Idempotent operations with security-first approach. No hardcoded credentials (vault integration). All playbooks validated in production environment.

## Key Capabilities

**Complete Stack Bootstrap** - System baseline → Services → Monitoring in one command  
**Reproducible Deployments** - Identical infrastructure across environments  
**Security Hardened** - systemd sandboxing, localhost-only defaults, vault integration  
**Tested Procedures** - All playbooks validated against production srv-m1m  
**30-Minute Bootstrap** - Bare metal to production in quantified, repeatable time

## Structure

playbooks/
├── system/
│ ├── base-setup.yml # OS baseline (users, packages, hardening)
│ ├── storage.yml # Btrfs setup, mount points, permissions
│ └── security.yml # Firewall, SSH hardening, fail2ban
├── services/
│ ├── forgejo.yml # Git hosting deployment
│ ├── vaultwarden.yml # Password vault + PostgreSQL
│ ├── syncthing.yml # File sync service
│ └── ollama.yml # Local LLM inference
└── monitoring/
├── health-checks.yml # Service status validation
└── backup-automation.yml # Daily backup scheduling

roles/
├── common/ # Shared base configuration
├── forgejo/ # Forgejo-specific tasks
├── vaultwarden/ # Vaultwarden + PostgreSQL
└── backup/ # Backup automation tasks

inventory/
├── production/ # Production hosts
├── group_vars/ # Group-level variables
└── host_vars/ # Host-specific variables

tests/
├── syntax-check.sh # YAML syntax validation
└── lint.sh # ansible-lint checks

## Quick Start

**Deploy complete stack:**

ansible-playbook -i inventory/production playbooks/site.yml

**Deploy single service (Forgejo example):**

ansible-playbook -i inventory/production playbooks/services/forgejo.yml

**Validate before deployment:**

ansible-playbook -i inventory/production playbooks/services/forgejo.yml --check --diff

## Example: Forgejo Deployment

playbooks/services/forgejo.yml
hosts: git_servers
become: yes
roles:

common

forgejo

tasks:

name: Deploy Forgejo service
include_role:
name: forgejo
vars:
forgejo_version: "1.21.1"
forgejo_data_path: "/mnt/data/srv/forgejo"
forgejo_listen: "127.0.0.1:3000"


**What this playbook does:**

1. Creates service user `git` with restricted permissions
2. Installs Forgejo binary to `/mnt/data/srv/forgejo/forgejo-bin`
3. Configures systemd service with sandboxing (ReadOnlyPaths, PrivateTmp)
4. Sets up automated daily backups via systemd timer
5. Validates service health (HTTP 200 response on localhost:3000)

**Execution time:** ~5 minutes  
**Idempotent:** Safe to run multiple times  
**Result:** Production-ready Forgejo instance with automated backups

## Design Principles

**Idempotent Operations**  
All tasks safe to run multiple times. State changes only occur when needed. Proper use of `creates`, `state: present`, and conditional checks.

**Declarative Configuration**  
Describe desired state, not imperative steps. Let Ansible determine execution path. Results in maintainable, readable playbooks.

**Modular Roles**  
Reusable components across playbooks. Clear separation of concerns. Easy to test and validate independently.

**Security First**  
- No hardcoded credentials (Ansible Vault integration)
- Localhost-only service defaults
- systemd sandboxing (ReadOnlyPaths, PrivateTmp, NoNewPrivileges)
- Pre-deployment validation via `--check` mode

**Documented Decisions**  
Inline comments explain why choices were made. Variable names describe purpose, not implementation. Links to relevant case studies for complex decisions.

## Service Deployment Details

### Forgejo (Git Hosting)
- **Binary:** `/mnt/data/srv/forgejo/forgejo-bin`
- **Data:** `/mnt/data/srv/forgejo/data`
- **Config:** `/mnt/data/srv/forgejo/app.ini`
- **Service:** `forgejo.service` (systemd)
- **Backup:** Daily via `forgejo-backup.timer`
- **Listen:** `127.0.0.1:3000` (localhost only)

### Vaultwarden (Password Vault)
- **Binary:** `/mnt/data/srv/vaultwarden/vaultwarden`
- **Data:** PostgreSQL database `vaultwarden`
- **Config:** `/mnt/data/srv/vaultwarden/.env`
- **Service:** `vaultwarden.service` (systemd)
- **Backup:** Daily PostgreSQL dump via `vaultwarden-backup.timer`
- **Listen:** `127.0.0.1:8222` (localhost only)

### Syncthing (File Sync)
- **Binary:** System package (`dnf install syncthing`)
- **Config:** `/mnt/data/srv/syncthing/config`
- **Data:** `/mnt/data/srv/syncthing/Sync`
- **Service:** `syncthing@ch1ch0.service` (systemd user service)
- **Listen:** `127.0.0.1:8384` (GUI), `22000` (sync protocol)

### Ollama (Local LLM)
- **Binary:** `/usr/local/bin/ollama`
- **Models:** `/mnt/data/srv/ollama/models`
- **Service:** `ollama.service` (systemd)
- **Listen:** `127.0.0.1:11434` (localhost only)

## Tested Scenarios

**Fresh Installation (Bare Metal):**  
Fedora Asahi base install → Full stack in 30 minutes

**Service Updates:**  
Forgejo 1.21.0 → 1.21.1 with zero downtime (blue-green restart pattern)

**Configuration Changes:**  
Update Vaultwarden `.env` and reload service automatically

**Backup Validation:**  
Deploy backup automation, verify cron execution, test restore procedure

## Related Infrastructure

**Production Deployment:** [fedora-asahi-srv-m1m](https://github.com/ch1ch0-FOSS/fedora-asahi-srv-m1m) - Infrastructure these playbooks build and maintain

**Backup Integration:** [disaster-recovery](https://github.com/ch1ch0-FOSS/disaster-recovery) - Backup scripts deployed and scheduled by these playbooks

**Design Rationale:** [infra-case-studies](https://github.com/ch1ch0-FOSS/infra-case-studies) - Case Study #5 explains IaC decisions and trade-offs

## Requirements

**Control Node:**
- Ansible 2.15+ (tested on 2.16)
- Python 3.9+
- SSH key-based authentication

**Target Hosts:**
- Fedora Linux 40+ (ARM64 tested, x86_64 compatible)
- Python 3.9+ (usually pre-installed)
- Sudo access for privilege escalation

**Optional:**
- Ansible Vault for credential encryption
- ansible-lint for pre-deployment validation

## Validation

**Syntax check:**

bash tests/syntax-check.sh

**Lint playbooks:**

bash tests/lint.sh

**Dry-run deployment:**

ansible-playbook -i inventory/production playbooks/site.yml --check --diff


## Contributing

See [CONTRIBUTING.md](../fedora-asahi-srv-m1m/CONTRIBUTING.md) in main infrastructure repository for standards, workflow, and commit conventions.

**Key Standards:**
- Conventional Commits format (enforced via commitlint)
- Ansible best practices (idempotency, proper variable scoping)
- Security validation (no hardcoded credentials, ansible-vault usage)
- Testing before merge (syntax check + lint + dry-run)

## License

MIT

---

