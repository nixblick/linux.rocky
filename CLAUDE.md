# Rocky Linux Security Hardening Role

Ansible-Rolle zur Härtung und Auditierung von Rocky Linux Systemen.

## Projektstruktur

```
.
├── defaults/main.yml      # Alle Konfigurationsvariablen
├── tasks/
│   ├── main.yml           # Haupteinstiegspunkt (nur includes)
│   ├── selinux.yml        # SELinux Härtung
│   ├── firewall.yml       # Firewall Härtung
│   ├── umask.yml          # Umask Härtung
│   ├── mounts.yml         # Mount-Point Härtung
│   ├── dnf.yml            # DNF/Package Härtung
│   ├── audit_*.yml        # Audit-Tasks für jeden Bereich
├── templates/             # Jinja2-Templates
├── meta/main.yml          # Rollen-Metadaten
└── playbook.yml           # Test-Playbook
```

## Konventionen

- Task-Namen beginnen mit `[Modulname]` in eckigen Klammern
- Jeder Sicherheitsbereich hat eine eigene Task-Datei
- Jede Task-Datei hat eine zugehörige `audit_*.yml`
- Alle Features über Tags und Boolean-Variablen steuerbar
- FQCN (Fully Qualified Collection Names) verwenden
- Alle Tasks müssen idempotent sein

## Tags

### Härtung
- `hardening` - Alle Härtungs-Tasks
- `selinux`, `firewall`, `umask`, `mounts`, `dnf` - Einzelne Module

### Audit
- `audit` - Alle Audit-Tasks
- `audit_selinux`, `audit_firewall`, `audit_umask`, `audit_mounts`, `audit_dnf`

## Variablen-Schema

Jedes Modul hat:
- `enable_{modul}_hardening: true/false`
- `enable_{modul}_audit: true/false`

## Befehle

```bash
# Syntax prüfen
ansible-playbook playbook.yml --syntax-check

# Linting
ansible-lint .

# Dry-Run
ansible-playbook playbook.yml --check --diff

# Tags auflisten
ansible-playbook playbook.yml --list-tags

# Nur Härtung
ansible-playbook playbook.yml --tags hardening

# Nur Audit
ansible-playbook playbook.yml --tags audit

# Einzelnes Modul
ansible-playbook playbook.yml --tags selinux
```

## Neue Module erstellen

1. Task-Datei: `tasks/{modul}.yml`
2. Audit-Datei: `tasks/audit_{modul}.yml`
3. Variablen in `defaults/main.yml`:
   - `enable_{modul}_hardening: true`
   - `enable_{modul}_audit: true`
4. Include in `tasks/main.yml`
5. README.md aktualisieren

## Abhängigkeiten

- `community.general` Collection
- Rocky Linux 8 oder 9