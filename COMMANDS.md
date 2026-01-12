# Befehls-Referenz - Rocky Linux Security Hardening

Diese Datei enthält eine vollständige Übersicht aller Befehle zur Verwendung dieser Ansible-Rolle.

## Installation

```bash
# Ansible Collections installieren
ansible-galaxy collection install -r requirements.yml

# Repository klonen (falls noch nicht geschehen)
git clone <repository-url> roles/rocky-hardening
cd roles/rocky-hardening
```

## Basis-Befehle

### Alle Module (Härtung + Audit)
```bash
ansible-playbook playbook.yml
```

### Nur Härtung (alle Module)
```bash
ansible-playbook playbook.yml --tags hardening
```

### Nur Audit (alle Module)
```bash
ansible-playbook playbook.yml --tags audit
```

## SELinux

### Härtung
```bash
ansible-playbook playbook.yml --tags selinux
```

### Audit
```bash
ansible-playbook playbook.yml --tags audit_selinux
```

### Beides
```bash
ansible-playbook playbook.yml --tags "selinux,audit_selinux"
```

## Firewall

### Härtung
```bash
ansible-playbook playbook.yml --tags firewall
```

### Audit
```bash
ansible-playbook playbook.yml --tags audit_firewall
```

### Beides
```bash
ansible-playbook playbook.yml --tags "firewall,audit_firewall"
```

## Umask

### Härtung
```bash
ansible-playbook playbook.yml --tags umask
```

### Audit
```bash
ansible-playbook playbook.yml --tags audit_umask
```

### Beides
```bash
ansible-playbook playbook.yml --tags "umask,audit_umask"
```

## Mounts / Filesystem

### Härtung
```bash
ansible-playbook playbook.yml --tags mounts
```

### Audit
```bash
ansible-playbook playbook.yml --tags audit_mounts
```

### Beides
```bash
ansible-playbook playbook.yml --tags "mounts,audit_mounts"
```

## DNF / Package Management

### Härtung
```bash
ansible-playbook playbook.yml --tags dnf
```

### Audit
```bash
ansible-playbook playbook.yml --tags audit_dnf
```

### Beides
```bash
ansible-playbook playbook.yml --tags "dnf,audit_dnf"
```

## Kombinierte Befehle

### Mehrere Härtungs-Module
```bash
# SELinux + Firewall
ansible-playbook playbook.yml --tags "selinux,firewall"

# SELinux + Firewall + Umask
ansible-playbook playbook.yml --tags "selinux,firewall,umask"

# Alle außer DNF
ansible-playbook playbook.yml --tags "selinux,firewall,umask,mounts"
```

### Mehrere Audit-Module
```bash
# SELinux + Firewall Audit
ansible-playbook playbook.yml --tags "audit_selinux,audit_firewall"

# Alle Audits außer DNF
ansible-playbook playbook.yml --tags "audit_selinux,audit_firewall,audit_umask,audit_mounts"
```

### Härtung + Audit für spezifische Module
```bash
# SELinux härten und dann auditieren
ansible-playbook playbook.yml --tags "selinux,audit_selinux"

# Firewall + Umask härten und auditieren
ansible-playbook playbook.yml --tags "firewall,umask,audit_firewall,audit_umask"
```

## Variablen überschreiben

### Einzelne Variable
```bash
ansible-playbook playbook.yml -e "selinux_state=permissive"
```

### Mehrere Variablen
```bash
ansible-playbook playbook.yml \
  -e "selinux_state=permissive" \
  -e "default_umask=0027" \
  -e "enable_dnf_hardening=false"
```

### Aus Datei laden
```bash
ansible-playbook playbook.yml -e "@custom_vars.yml"
```

### Module deaktivieren
```bash
# SELinux Härtung deaktivieren
ansible-playbook playbook.yml -e "enable_selinux_hardening=false"

# Mehrere Module deaktivieren
ansible-playbook playbook.yml \
  -e "enable_selinux_hardening=false" \
  -e "enable_dnf_hardening=false"
```

## Erweiterte Optionen

### Check Mode (Dry Run)
```bash
# Zeigt was geändert würde, ohne Änderungen durchzuführen
ansible-playbook playbook.yml --check

# Mit Diff-Ausgabe
ansible-playbook playbook.yml --check --diff
```

### Verbose Output
```bash
# Standard Verbose
ansible-playbook playbook.yml -v

# Mehr Details
ansible-playbook playbook.yml -vv

# Maximale Details (mit Connection-Debug)
ansible-playbook playbook.yml -vvv

# Extremes Debug-Level
ansible-playbook playbook.yml -vvvv
```

### Nur bestimmte Hosts
```bash
# Einzelner Host
ansible-playbook playbook.yml -l localhost

# Mehrere Hosts
ansible-playbook playbook.yml -l "server1,server2"

# Host-Gruppe
ansible-playbook playbook.yml -l rocky_servers
```

### Mit anderem Inventory
```bash
ansible-playbook -i production.ini playbook.yml
```

### Start bei bestimmtem Task
```bash
ansible-playbook playbook.yml --start-at-task="Task Name"
```

### Step-by-Step Ausführung
```bash
ansible-playbook playbook.yml --step
```

## Audit-Report Management

### Report-Location finden
```bash
# Standard-Location
ls -lh /tmp/security_audit_report_*.txt

# Neuesten Report anzeigen
cat $(ls -t /tmp/security_audit_report_*.txt | head -1)
```

### Custom Report-Location
```bash
ansible-playbook playbook.yml --tags audit \
  -e "audit_report_path=/var/log/security_audit_$(date +%Y%m%d).txt"
```

### Bei Violations fehlschlagen
```bash
ansible-playbook playbook.yml --tags audit \
  -e "audit_fail_on_violations=true"
```

## Testing und Validierung

### Syntax-Check
```bash
ansible-playbook playbook.yml --syntax-check
```

### Ansible-Lint (wenn installiert)
```bash
ansible-lint playbook.yml
```

### Task-Liste anzeigen
```bash
ansible-playbook playbook.yml --list-tasks
```

### Tags anzeigen
```bash
ansible-playbook playbook.yml --list-tags
```

### Hosts anzeigen
```bash
ansible-playbook playbook.yml --list-hosts
```

## Spezielle Szenarien

### Nur Report ohne Änderungen
```bash
ansible-playbook playbook.yml --tags audit --skip-tags hardening
```

### Härtung ohne Audit
```bash
ansible-playbook playbook.yml --tags hardening --skip-tags audit
```

### Alle außer einem Modul
```bash
# Alles außer SELinux
ansible-playbook playbook.yml --skip-tags "selinux,audit_selinux"
```

### Production-Safe Run
```bash
# Check Mode + Diff + Verbose
ansible-playbook playbook.yml --check --diff -v
```

### Schneller Compliance-Check
```bash
# Nur Audits, keine Änderungen
ansible-playbook playbook.yml --tags audit -vv
```

## Beispiel-Workflows

### Initial Setup (neue Maschine)
```bash
# 1. Erst prüfen was geändert würde
ansible-playbook playbook.yml --check --diff

# 2. Härtung durchführen
ansible-playbook playbook.yml --tags hardening

# 3. Audit durchführen
ansible-playbook playbook.yml --tags audit

# 4. Report prüfen
cat /tmp/security_audit_report_*.txt
```

### Regelmäßiger Compliance-Check
```bash
# Nur Audit mit Report
ansible-playbook playbook.yml --tags audit

# Report automatisch per Mail versenden (Beispiel)
ansible-playbook playbook.yml --tags audit && \
  mail -s "Security Audit Report $(date)" admin@example.com < \
  $(ls -t /tmp/security_audit_report_*.txt | head -1)
```

### Schrittweise Härtung
```bash
# Tag 1: Basis-Härtung
ansible-playbook playbook.yml --tags "umask,dnf"

# Tag 2: SELinux (vorsichtig)
ansible-playbook playbook.yml --tags selinux -e "selinux_state=permissive"

# Tag 3: SELinux auf Enforcing
ansible-playbook playbook.yml --tags selinux -e "selinux_state=enforcing"

# Tag 4: Firewall (mit Vorsicht)
ansible-playbook playbook.yml --tags firewall

# Tag 5: Mounts
ansible-playbook playbook.yml --tags mounts

# Final: Vollständiges Audit
ansible-playbook playbook.yml --tags audit
```

## Troubleshooting Commands

### SELinux Probleme untersuchen
```bash
# Denials anzeigen
ausearch -m avc -ts recent

# SELinux Alerts
sealert -a /var/log/audit/audit.log
```

### Firewall Probleme
```bash
# Aktuelle Konfiguration prüfen
firewall-cmd --list-all

# Logs überwachen
journalctl -u firewalld -f
```

### Ansible-Logs prüfen
```bash
# Wenn log_path in ansible.cfg gesetzt ist
tail -f ansible.log
```

## Quick Reference Table

| Aktion | Befehl |
|--------|--------|
| Alles | `ansible-playbook playbook.yml` |
| Nur Härtung | `ansible-playbook playbook.yml --tags hardening` |
| Nur Audit | `ansible-playbook playbook.yml --tags audit` |
| SELinux | `ansible-playbook playbook.yml --tags selinux` |
| Firewall | `ansible-playbook playbook.yml --tags firewall` |
| Umask | `ansible-playbook playbook.yml --tags umask` |
| Mounts | `ansible-playbook playbook.yml --tags mounts` |
| DNF | `ansible-playbook playbook.yml --tags dnf` |
| Check Mode | `ansible-playbook playbook.yml --check` |
| Verbose | `ansible-playbook playbook.yml -vv` |

## Hilfe

```bash
# Ansible Playbook Hilfe
ansible-playbook --help

# Ansible Dokumentation
ansible-doc <module_name>

# z.B. für firewalld
ansible-doc firewalld
```
