# settings.local.json - Dokumentation

**Pfad:** `.claude/settings.local.json`  
**Projekt:** Rocky Linux Security Hardening Role

---

## Wichtiger Hinweis: Pfad-Beschränkungen

Claude Code ist **NICHT** auf das Startverzeichnis beschränkt!  
Die Bash-Regeln schränken nur **Befehle** ein, nicht **Pfade**.

```bash
# Claude kann das hier machen - auch wenn gestartet in ~/ansible-rolle/
cat /etc/passwd
cat ~/.ssh/id_rsa
diff ~/projekt-a/file.txt ~/projekt-b/file.txt
```

### Hierarchie der Settings-Dateien

| Datei | Scope | Pfad-Beschränkung |
|-------|-------|-------------------|
| `.claude/settings.local.json` | Projekt | Nein - nur Befehle |
| `~/.claude/settings.json` | User | Nein - nur Befehle |
| `/etc/claude-code/managed-settings.json` | System/Enterprise | Nein - nur Befehle |

### Für Pfad-Schutz: Read() und Edit() Regeln nutzen

```json
"Read(/etc/shadow)",
"Read(**/.env)",
"Edit(/etc/**)"
```

### Working Directory beibehalten (optional)

```bash
export CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=1
```

---

## Allow-Regeln (Erlaubte Befehle)

### Dateisystem - Lesen

```json
"Bash(cat:*)",       // Dateiinhalt anzeigen
"Bash(ls:*)",        // Verzeichnis listen
"Bash(tree:*)",      // Baumstruktur
"Bash(head:*)",      // Erste Zeilen
"Bash(tail:*)",      // Letzte Zeilen
"Bash(find:*)",      // Dateien suchen
"Bash(grep:*)",      // In Dateien suchen
```

### Dateisystem - Schreiben

```json
"Bash(mkdir:*)",     // Verzeichnisse erstellen
"Bash(touch:*)",     // Leere Dateien erstellen
"Bash(cp:*)",        // Dateien kopieren
"Bash(mv:*)",        // Dateien verschieben/umbenennen
```

### Ansible - Validierung & Linting

```json
"Bash(ansible-lint:*)",                    // Linting
"Bash(ansible-playbook:--syntax-check*)",  // Syntax prüfen
"Bash(ansible-playbook:--check*)",         // Dry-Run (Check Mode)
"Bash(ansible-playbook:--list-tasks*)",    // Tasks auflisten
"Bash(ansible-playbook:--list-tags*)",     // Tags auflisten
"Bash(ansible-inventory:*)",               // Inventory anzeigen
"Bash(yamllint:*)",                        // YAML Linting
```

### Ansible Galaxy

```json
"Bash(ansible-galaxy:*)",  // Collection und Role Management
```

### Git - Lesend

```json
"Bash(git status:*)",   // Status anzeigen
"Bash(git diff:*)",     // Änderungen anzeigen
"Bash(git log:*)",      // Historie anzeigen
"Bash(git branch:*)",   // Branches auflisten
"Bash(git show:*)",     // Commits anzeigen
```

### Git - Schreibend

```json
"Bash(git add:*)",      // Dateien stagen
"Bash(git commit:*)",   // Committen
```

### Python/Pip

```json
"Bash(pip:install --user*)",  // Pakete installieren (user-scope)
"Bash(pip:list*)",            // Installierte Pakete anzeigen
```

### Molecule Testing

```json
"Bash(molecule:list*)",    // Status anzeigen
"Bash(molecule:syntax*)",  // Syntax prüfen
"Bash(molecule:lint*)",    // Linting
```

---

## Deny-Regeln (Verbotene Befehle und Pfade)

**Wichtig:** Deny hat höhere Priorität als Allow!

### Pfad-Beschränkungen - Read (Lesen blockieren)

#### SSH & Crypto - Niemals lesen

```json
"Read(**/.ssh/**)",       // Alle SSH-Verzeichnisse
"Read(**/id_rsa)",        // RSA Private Keys
"Read(**/id_ed25519)",    // ED25519 Private Keys
"Read(**/*.pem)",         // PEM-Zertifikate/Keys
"Read(**/*.key)",         // Private Key-Dateien
```

#### Secrets & Credentials

```json
"Read(**/.env)",              // Environment-Dateien
"Read(**/.env.*)",            // .env.local, .env.prod, etc.
"Read(**/*secret*)",          // Dateien mit "secret" im Namen
"Read(**/*password*)",        // Dateien mit "password" im Namen
"Read(**/*.vault)",           // Ansible Vault-Dateien
"Read(**/*.vault.yml)",       // Ansible Vault YAML
"Read(**/vault.yml)",         // Ansible Vault
"Read(**/.netrc)",            // FTP/Curl Credentials
"Read(**/.aws/credentials)",  // AWS Credentials
"Read(**/.kube/config)",      // Kubernetes Credentials
```

#### System-Dateien

```json
"Read(/etc/shadow)",    // Passwort-Hashes
"Read(/etc/gshadow)",   // Gruppen-Passwörter
"Read(/etc/sudoers)",   // Sudo-Konfiguration
"Read(/root/**)",       // Root Home
```

#### History-Dateien (können Passwörter enthalten)

```json
"Read(**/.bash_history)",   // Bash Historie
"Read(**/.zsh_history)",    // ZSH Historie
"Read(**/.mysql_history)",  // MySQL Historie
```

### Pfad-Beschränkungen - Edit (Schreiben blockieren)

```json
"Edit(/etc/**)",       // Kein Schreiben nach /etc
"Edit(/usr/**)",       // Kein Schreiben nach /usr
"Edit(/bin/**)",       // Kein Schreiben nach /bin
"Edit(/sbin/**)",      // Kein Schreiben nach /sbin
"Edit(/boot/**)",      // Kein Schreiben nach /boot
"Edit(/root/**)",      // Kein Schreiben nach /root
"Edit(/home/*/**)",    // Kein Schreiben bei anderen Usern
```

### Ansible/Playbook Schutz

```json
"Bash(ansible-playbook:*-i /etc/ansible/hosts*)",  // System-Inventory
"Bash(ansible-playbook:*--limit prod*)",           // Produktions-Hosts
"Bash(ansible-playbook:*--limit production*)",     // Produktions-Hosts
"Bash(ansible-playbook:*inventory/prod*)",         // Produktions-Inventory
```

### Git Schutz

```json
"Bash(git push:*)",          // Kein automatisches Push
"Bash(git push:--force*)",   // Kein Force-Push
"Bash(git push:-f*)",        // Kein Force-Push (kurz)
"Bash(git reset:--hard*)",   // Hard Reset
"Bash(git clean:-fd*)",      // Untracked löschen
"Bash(git clean:-fx*)",      // Alles löschen
```

### System-Administration

```json
"Bash(sudo:*)",        // Keine Root-Rechte
"Bash(su:*)",          // Kein User-Wechsel
"Bash(systemctl:*)",   // Keine Service-Kontrolle
"Bash(service:*)",     // Keine Service-Kontrolle (alt)
"Bash(reboot:*)",      // Kein Neustart
"Bash(shutdown:*)",    // Kein Herunterfahren
```

### Destruktive Operationen

```json
"Bash(rm:-rf /*)",       // Kein rekursives Löschen von /
"Bash(rm:-rf ~/*)",      // Kein Löschen von Home
"Bash(rm:-rf /home/*)",  // Kein Löschen von /home
"Bash(rm:-rf /etc/*)",   // Kein Löschen von /etc
"Bash(chmod:-R 777*)",   // Keine unsicheren Berechtigungen
"Bash(chown:-R*)",       // Kein rekursives Ownership-Change
```

### Paket-Management

```json
"Bash(dnf:*)",     // Kein DNF
"Bash(yum:*)",     // Kein YUM
"Bash(rpm:-i*)",   // Keine RPM-Installation
"Bash(rpm:-e*)",   // Keine RPM-Deinstallation
```

### Netzwerk/Firewall

```json
"Bash(iptables:*)",      // Keine Firewall-Änderungen
"Bash(firewall-cmd:*)",  // Keine Firewall-Änderungen
"Bash(nft:*)",           // Keine nftables-Änderungen
```

### Sensitive Dateien via Bash (doppelte Absicherung)

```json
"Bash(cat:*id_rsa*)",       // Keine SSH-Keys ausgeben
"Bash(cat:*id_ed25519*)",   // Keine SSH-Keys ausgeben
"Bash(cat:*.env*)",         // Keine .env-Dateien
"Bash(cat:*vault*)",        // Keine Vault-Dateien
"Bash(cat:*secret*)",       // Keine Secret-Dateien
"Bash(cat:*password*)",     // Keine Passwort-Dateien
"Bash(cat:*/etc/shadow*)",  // Keine Shadow-Datei
"Bash(grep:*id_rsa*)",      // Kein Grep in SSH-Keys
"Bash(grep:*.env*)",        // Kein Grep in .env
```

---

## Erweiterungen

### Playbook-Ausführung auf localhost erlauben

In `allow` hinzufügen:

```json
"Bash(ansible-playbook:*-c local*)",
"Bash(ansible-playbook:*--connection=local*)",
"Bash(ansible-playbook:*localhost*)"
```

### Molecule vollständig erlauben

```json
"Bash(molecule:*)"
```

### Git Push erlauben

Aus `deny` entfernen:

```json
"Bash(git push:*)"
```

---

## Referenz

- [Claude Code Settings Dokumentation](https://docs.anthropic.com/en/docs/claude-code/settings)
- Deny hat immer Vorrang vor Allow
- `Read()` und `Edit()` werden "best-effort" auch auf Bash-Tools angewendet