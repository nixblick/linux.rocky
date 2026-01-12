# Rocky Linux Security Hardening Role

Eine umfassende Ansible-Rolle zur Härtung und Auditierung von Rocky Linux Systemen.

## Übersicht

Diese Rolle implementiert Security-Hardening für Rocky Linux Systeme und bietet eine vollständige Audit-Funktionalität zur Überprüfung der Compliance. Jeder Sicherheitsbereich ist modular aufgebaut und kann einzeln aktiviert/deaktiviert werden.

## Features

- **SELinux Härtung**: Konfiguration von SELinux-Status, File Contexts, Booleans und Ports
- **Firewall Härtung**: Verwaltung von firewalld Zonen, Services, Ports und Rich Rules
- **Umask Härtung**: Systemweite Umask-Konfiguration für erhöhte Sicherheit
- **Mount-Point Härtung**: Sichere Mount-Optionen und Filesystem-Überwachung
- **DNF/Package Management**: Automatische Updates und Paket-Compliance
- **Vollständige Audit-Funktionalität**: Überprüfung aller Sicherheitseinstellungen
- **Tag-basierte Steuerung**: Granulare Kontrolle über Ausführung einzelner Module
- **Detaillierter Audit-Report**: Automatische Generierung von Compliance-Berichten

## Verzeichnisstruktur

```
.
├── .claude/
│   └── claude.yaml          # Claude Code Konfiguration
├── defaults/
│   └── main.yml             # Alle Konfigurationsvariablen
├── tasks/
│   ├── main.yml             # Haupteinstiegspunkt
│   ├── selinux.yml          # SELinux Härtung
│   ├── firewall.yml         # Firewall Härtung
│   ├── umask.yml            # Umask Härtung
│   ├── mounts.yml           # Mount-Point Härtung
│   ├── dnf.yml              # DNF/Package Härtung
│   ├── audit_selinux.yml    # SELinux Audit
│   ├── audit_firewall.yml   # Firewall Audit
│   ├── audit_umask.yml      # Umask Audit
│   ├── audit_mounts.yml     # Mount-Point Audit
│   └── audit_dnf.yml        # DNF/Package Audit
└── README.md                # Diese Datei
```

## Voraussetzungen

- Ansible >= 2.9
- Rocky Linux 8 oder 9
- Root/Sudo-Berechtigungen auf dem Zielsystem
- Collection: `community.general` (für SELinux-Module)

Installation der Requirements:
```bash
ansible-galaxy collection install community.general
```

## Installation

1. Klone dieses Repository oder kopiere die Rolle in dein Ansible-Projekt:
```bash
git clone <repository-url> roles/rocky-hardening
```

2. Erstelle ein Playbook (siehe Beispiel unten)

## Konfiguration

Alle Konfigurationsvariablen sind in `defaults/main.yml` definiert. Die wichtigsten Einstellungen:

### Globale Schalter

```yaml
# Härtungs-Module aktivieren/deaktivieren
enable_selinux_hardening: true
enable_firewall_hardening: true
enable_umask_hardening: true
enable_mounts_hardening: true
enable_dnf_hardening: true

# Audit-Module aktivieren/deaktivieren
enable_selinux_audit: true
enable_firewall_audit: true
enable_umask_audit: true
enable_mounts_audit: true
enable_dnf_audit: true
```

### SELinux Konfiguration

```yaml
selinux_state: enforcing
selinux_policy: targeted

selinux_file_contexts:
  - { target: '/var/www/html(/.*)?', setype: 'httpd_sys_content_t' }

selinux_booleans:
  - { name: 'httpd_can_network_connect', state: 'off', persistent: true }
```

### Firewall Konfiguration

```yaml
firewall_default_zone: public

firewall_allowed_services:
  - ssh
  - https

firewall_allowed_ports:
  - 22/tcp
  - 443/tcp
```

### Weitere Konfigurationen

Siehe `defaults/main.yml` für alle verfügbaren Optionen.

## Verwendung

### Beispiel Playbook

Erstelle eine Datei `playbook.yml`:

```yaml
---
- name: Rocky Linux Security Hardening
  hosts: localhost
  become: true
  roles:
    - rocky-hardening
```

### Basis-Befehle

#### Vollständige Härtung durchführen
```bash
ansible-playbook playbook.yml --tags hardening
```

#### Vollständiges Audit durchführen
```bash
ansible-playbook playbook.yml --tags audit
```

#### Härtung und Audit kombiniert
```bash
ansible-playbook playbook.yml
```

### Modulspezifische Befehle

#### SELinux

**Härtung:**
```bash
ansible-playbook playbook.yml --tags selinux
```

**Audit:**
```bash
ansible-playbook playbook.yml --tags audit_selinux
```

**Beides:**
```bash
ansible-playbook playbook.yml --tags "selinux,audit_selinux"
```

#### Firewall

**Härtung:**
```bash
ansible-playbook playbook.yml --tags firewall
```

**Audit:**
```bash
ansible-playbook playbook.yml --tags audit_firewall
```

**Beides:**
```bash
ansible-playbook playbook.yml --tags "firewall,audit_firewall"
```

#### Umask

**Härtung:**
```bash
ansible-playbook playbook.yml --tags umask
```

**Audit:**
```bash
ansible-playbook playbook.yml --tags audit_umask
```

**Beides:**
```bash
ansible-playbook playbook.yml --tags "umask,audit_umask"
```

#### Mounts

**Härtung:**
```bash
ansible-playbook playbook.yml --tags mounts
```

**Audit:**
```bash
ansible-playbook playbook.yml --tags audit_mounts
```

**Beides:**
```bash
ansible-playbook playbook.yml --tags "mounts,audit_mounts"
```

#### DNF/Package Management

**Härtung:**
```bash
ansible-playbook playbook.yml --tags dnf
```

**Audit:**
```bash
ansible-playbook playbook.yml --tags audit_dnf
```

**Beides:**
```bash
ansible-playbook playbook.yml --tags "dnf,audit_dnf"
```

### Kombinierte Module

**Mehrere Module gleichzeitig:**
```bash
ansible-playbook playbook.yml --tags "selinux,firewall,umask"
```

**Nur Audits für bestimmte Module:**
```bash
ansible-playbook playbook.yml --tags "audit_selinux,audit_firewall"
```

### Erweiterte Optionen

#### Mit eigenen Variablen
```bash
ansible-playbook playbook.yml --tags hardening \
  -e "enable_selinux_hardening=false" \
  -e "default_umask=0027"
```

#### Nur Audit ohne Härtung
```bash
ansible-playbook playbook.yml --tags audit --skip-tags hardening
```

#### Dry-Run (Check Mode)
```bash
ansible-playbook playbook.yml --check --diff
```

#### Verbose Output
```bash
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vv   # Mehr Details
ansible-playbook playbook.yml -vvv  # Maximale Details
```

## Audit-Reports

Nach einem Audit-Lauf wird automatisch ein Report generiert:

**Standard-Location:**
```
/tmp/security_audit_report_<timestamp>.txt
```

**Konfigurierbar über:**
```yaml
audit_report_path: "/tmp/security_audit_report_{{ ansible_date_time.epoch }}.txt"
audit_fail_on_violations: false  # Auf true setzen um bei Violations zu failen
```

**Report-Inhalt:**
- Timestamp und System-Informationen
- Detaillierte Prüfungsergebnisse pro Modul
- Zusammenfassung aller Violations
- Compliance-Status

## Best Practices

1. **Testen vor Produktion**: Teste alle Änderungen zunächst in einer Test-Umgebung
2. **Backup erstellen**: Erstelle Backups kritischer Konfigurationsdateien
3. **Schrittweise Härtung**: Aktiviere Module einzeln und teste jeweils die Funktionalität
4. **Regelmäßige Audits**: Führe regelmäßig Audits durch um Drift zu erkennen
5. **Anpassung an Anforderungen**: Passe die Variablen in `defaults/main.yml` an deine Bedürfnisse an

## Fehlerbehebung

### SELinux verhindert Zugriff
```bash
# Prüfe SELinux Denials
ausearch -m avc -ts recent
sealert -a /var/log/audit/audit.log
```

### Firewall blockiert Verbindungen
```bash
# Liste aktive Firewall-Regeln
firewall-cmd --list-all
# Prüfe Logs
journalctl -u firewalld -f
```

### Audit schlägt fehl
```bash
# Führe einzelne Module mit mehr Verbosity aus
ansible-playbook playbook.yml --tags audit_selinux -vv
```

## Sicherheitshinweise

- Diese Rolle führt weitreichende Systemänderungen durch
- SELinux-Änderungen können einen Neustart erfordern
- Firewall-Änderungen können SSH-Verbindungen unterbrechen (SSH-Port beachten!)
- Teste immer in einer sicheren Umgebung
- Stelle sicher, dass du Zugang zum System hast, auch wenn SSH blockiert wird

## Tags Übersicht

| Tag | Beschreibung |
|-----|-------------|
| `hardening` | Führt alle Härtungs-Tasks aus |
| `audit` | Führt alle Audit-Tasks aus |
| `selinux` | SELinux Härtung |
| `firewall` | Firewall Härtung |
| `umask` | Umask Härtung |
| `mounts` | Mount-Point Härtung |
| `dnf` | DNF/Package Härtung |
| `audit_selinux` | SELinux Audit |
| `audit_firewall` | Firewall Audit |
| `audit_umask` | Umask Audit |
| `audit_mounts` | Mount-Point Audit |
| `audit_dnf` | DNF/Package Audit |
| `report` | Report-Generierung |

## Quick Reference - Befehle

```bash
# Komplette Härtung + Audit
ansible-playbook playbook.yml

# Nur Härtung
ansible-playbook playbook.yml --tags hardening

# Nur Audit
ansible-playbook playbook.yml --tags audit

# Einzelnes Modul härten
ansible-playbook playbook.yml --tags selinux
ansible-playbook playbook.yml --tags firewall
ansible-playbook playbook.yml --tags umask
ansible-playbook playbook.yml --tags mounts
ansible-playbook playbook.yml --tags dnf

# Einzelnes Modul auditieren
ansible-playbook playbook.yml --tags audit_selinux
ansible-playbook playbook.yml --tags audit_firewall
ansible-playbook playbook.yml --tags audit_umask
ansible-playbook playbook.yml --tags audit_mounts
ansible-playbook playbook.yml --tags audit_dnf

# Mehrere Module kombinieren
ansible-playbook playbook.yml --tags "selinux,firewall"
ansible-playbook playbook.yml --tags "audit_selinux,audit_firewall"

# Mit eigenen Variablen
ansible-playbook playbook.yml -e "selinux_state=permissive"

# Check Mode (Dry Run)
ansible-playbook playbook.yml --check

# Mit Verbose Output
ansible-playbook playbook.yml -vv
```

## Lizenz

MIT

## Autor

Erstellt mit Claude Code

## Support

Bei Fragen oder Problemen erstelle bitte ein Issue im Repository.
