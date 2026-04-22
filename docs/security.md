# Security & Kosten

_Baseline für Projekte aus diesem Template. Vor Produktivnutzung vollständig lesen und pro Projekt anpassen._

---

## 1. Kosten-Kontrolle

### Spending-Cap im Anthropic-Konto
**Pflicht vor Aktivierung von `claude-code-action`.**

- Gehe zu https://console.anthropic.com/settings/limits
- Setze ein **monatliches Limit** — Vorschlag für M1-/Pilot-Phase: **$50/Monat**
- Alert-Threshold bei 50% und 80%
- Bei Überschreitung: API-Key wird blockiert → keine überraschenden Rechnungen

### Modell-Wahl pro Agent-Rolle

Im Subagent-Frontmatter (`model:`) bewusst wählen:

| Agent | Modell | Grund |
|---|---|---|
| `code-reviewer` | `haiku` | Massenaufgabe, Token-Effizienz wichtiger als maximale Tiefe |
| `agent-organizer` (Planner) | `sonnet` | Scope-Zerlegung erfordert Kontext-Verständnis |
| `fullstack-developer` | `sonnet` | Balance zwischen Qualität und Kosten |
| `test-automator` | `sonnet` | Test-Design ist knifflig, braucht Sorgfalt |
| `devops-engineer` | `haiku` | Meist Routinearbeit, CI/CD-YAML-Edits |

**Faustregel:** `opus` nur für Architektur-Entscheidungen und Debugging mit grossem Kontextfenster.

### PR-Grössen-Limit

Grosse PRs = teure Reviews. Hard-Limit in CI:
- **PR > 500 geänderte Zeilen** → automatischer Hinweis „Bitte aufteilen"
- **PR > 1000 Zeilen** → Review-Agent wird nicht automatisch getriggert, nur manuell

### Cron-Workflows

Jeder Cron-Workflow verbraucht Tokens auch wenn niemand ihn braucht.

- **Keine** `schedule`-Trigger ohne expliziten Nutzen
- Falls doch: dokumentieren warum, monatliche Kosten schätzen, Spending-Cap anpassen
- Bevorzugt: `workflow_dispatch` (manuell auslösbar) statt Cron

---

## 2. Permission-Baseline (`.claude/settings.json`)

Template-Wert für ein neues Projekt. Pro Projekt anpassen, aber **nur restriktiver, nie laxer**.

```json
{
  "permissions": {
    "deny": [
      "Bash(curl *)",
      "Bash(wget *)",
      "Bash(rm -rf *)",
      "Bash(sudo *)",
      "Bash(chmod 777 *)",
      "WebFetch(*)",
      "Bash(git push --force*)",
      "Bash(git reset --hard *)"
    ],
    "allow": [
      "Read(*)",
      "Grep(*)",
      "Glob(*)",
      "Bash(git status)",
      "Bash(git diff*)",
      "Bash(git log*)",
      "Bash(git add *)",
      "Bash(git commit *)",
      "Bash(git checkout *)",
      "Bash(git branch*)",
      "Bash(npm run *)",
      "Bash(pytest*)",
      "Bash(python *)"
    ]
  }
}
```

**Regeln:**
- `deny` > `allow` (Deny hat Vorrang bei Konflikten)
- `Bash(*)` ist **niemals** erlaubt
- Netzwerk-Tools (`curl`, `wget`, `WebFetch`) default deny — nur explizit freigeben wenn nötig
- Destruktive Git-Operationen (`--force`, `--hard`) default deny

---

## 3. Prompt-Injection-Schutz

### CVE-2025-54794 / CVE-2025-54795 (GitHub-Comments)
Angreifer platzieren schädliche Anweisungen in PR-/Issue-Kommentaren. `claude-code-action` liest diese als Prompt und führt Anweisungen aus.

**Gegenmassnahmen in Workflows:**

```yaml
# ❌ UNSICHER — jeder Kommentator kann Agent steuern
on:
  issue_comment:
    types: [created]

# ✅ SICHER — nur OWNER/COLLABORATOR
on:
  issue_comment:
    types: [created]
jobs:
  claude-response:
    if: |
      github.event.comment.author_association == 'OWNER' ||
      github.event.comment.author_association == 'COLLABORATOR' ||
      github.event.comment.author_association == 'MEMBER'
    runs-on: ubuntu-latest
    ...
```

### Trusted-PR-Only

`pull_request_target` gibt Schreibrechte an Fork-Code — **nie** verwenden ausser mit extrem restriktivem Scope. `pull_request` ist sicherer.

### Secrets niemals in Prompts

- Keine API-Keys in Agent-Prompts schreiben
- Secrets ausschliesslich via GitHub Secrets + Workflow-Env-Vars
- In Claude-Code-Sessions: keine Keys in CLAUDE.md oder in Chat kopieren

---

## 4. Remote Control Sicherheit

`claude remote-control` ermöglicht Mobile/Web-Zugriff auf laufende Sessions.

**Regeln:**
- Auth: claude.ai **OAuth only** — keine API-Keys, keine long-lived Tokens
- Session endet nach **>10 Minuten Netzwerkausfall**
- Outbound HTTPS only — keine inbound Ports, keine Firewall-Löcher
- QR-Code nur auf eigenen Geräten scannen — nicht teilen
- Enterprise-Konto: Remote Control default deaktiviert, Admin-Aktivierung nötig

**Regel:** Remote Control **niemals** auf Rechnern aktivieren, die mit dem Firmen-VPN verbunden sind — Datenfluss vorher mit IT klären.

---

## 5. Dependency-Hygiene

### GitHub Actions pinnen

Actions nicht mit Tag, sondern mit Commit-SHA pinnen (Supply-Chain-Schutz):

```yaml
# ❌ Floating Tag — Action kann heimlich geändert werden
- uses: actions/checkout@v4

# ✅ SHA-gepinnt — Version ist unveränderlich
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11  # v4.1.1
```

Für First-Party (Anthropic-Actions) kann Tag-Pinning OK sein. Third-Party: immer SHA.

### npm / pip Audit
- `npm audit` oder `pip-audit` in CI-Pipeline
- Dependabot aktiviert (GitHub-Settings)
- Updates wöchentlich, kritische Updates sofort

### Lockfile committen
- `package-lock.json`, `pnpm-lock.yaml`, `poetry.lock`, `Pipfile.lock` IMMER committen
- Nicht im `.gitignore`

---

## 6. Incident Response

### API-Key-Leak
1. Sofort Key in Anthropic Console rotieren (https://console.anthropic.com/settings/keys)
2. Neuen Key in GitHub Secrets eintragen
3. `git log --all -p -- <verdächtige-datei>` — wenn Key in Git-History: BFG Repo-Cleaner oder `git filter-branch`
4. Zur Sicherheit: Spending seit Leak-Zeitpunkt prüfen

### Verdächtiger Claude-Output
Symptom: Agent schlägt ungewöhnliche Aktionen vor (Files löschen, `curl` zu unbekannten Domains, Änderungen ausserhalb des Tasks).

1. Session sofort stoppen (Esc Esc)
2. Letzte User-Inputs prüfen — kam Prompt-Injection von extern?
3. Git-Status prüfen — nichts gestaged, was unerwartet ist?
4. CLAUDE.md und Subagent-Prompts auf Manipulation prüfen
5. Wenn Schaden: aus letztem sauberen Commit restoren

### Unerwarteter Commit im Repo
1. Wer hat den Push gemacht? `git log --format="%an %ae" -1 <sha>`
2. Wenn via GitHub Action: Action-Run-Log prüfen
3. Wenn Fremd-Account: Repo-Collaborators prüfen, verdächtige Tokens revoken (GitHub Settings → Developer Settings → Personal access tokens)
4. Revert: `git revert <sha>` — niemals `git push --force` auf `main`

---

## 7. Regelmässige Checks (monatlich)

- [ ] Spending Review — Kosten der letzten 30 Tage prüfen, Anomalien untersuchen
- [ ] Dependabot-Alerts abarbeiten
- [ ] `claude-code-action` auf aktuelle Major-Version prüfen (Release Notes)
- [ ] GitHub Audit Log auf ungewöhnliche Aktivität (Settings → Audit log)
- [ ] `.claude/settings.json` → Permissions-Drift zur Baseline prüfen

---

## 8. Checkliste: Neues Projekt aus Template

- [ ] Spending-Cap gesetzt (Vorschlag: $50/Monat)
- [ ] `ANTHROPIC_API_KEY` als GitHub Secret
- [ ] `claude-code-action` via `/install-github-app` installiert
- [ ] Branch Protection auf `main` aktiv
- [ ] `.claude/settings.json` Permission-Baseline übernommen
- [ ] Workflow-Trigger auf `author_association`-Filter geprüft
- [ ] Dependabot aktiviert
- [ ] Alle GitHub Actions SHA-gepinnt (Third-Party)
- [ ] README aktualisiert mit Projekt-spezifischer Security-Policy

---

## Referenzen

- [CVE-2025-54794 — Prompt-Injection via GitHub Comments](https://www.securityweek.com/claude-code-gemini-cli-github-copilot-agents-vulnerable-to-prompt-injection-via-comments/)
- [Anthropic Code Review — Sicherheitsempfehlungen](https://claude.com/blog/code-review)
- [GitHub — Keeping your Actions secure](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [Claude Code Remote Control — Docs](https://code.claude.com/docs/en/remote-control)
