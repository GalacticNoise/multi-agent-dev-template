# [Projekt-Name]

_Template-Version 1.0 — generiert aus `multi-agent-dev-template` (idea-0017) am 2026-04-22._

Diese Datei ist das Projekt-Memory für Claude Code. Beim Klonen/Forken dieses Templates: Platzhalter ersetzen, Kontext mit eigenem Projekt-Inhalt füllen.

---

## Projekt-Kontext

**Name:** [Projekt-Name]
**Zweck:** [1–2 Sätze — was wird hier gebaut und warum]
**Stack:** [Node/TypeScript | Python | Flutter | Unity | ...]
**Owner:** Steve Weber

---

## Agents

Die Agents unter `.claude/agents/` sind die Multi-Agent-Rollen für dieses Projekt:

| Datei | Rolle | Wann einsetzen |
|---|---|---|
| `agent-organizer.md` | Planner — zerlegt Aufgaben, definiert Scope | Zu Beginn jedes neuen Features/Tickets |
| `fullstack-developer.md` | Developer — implementiert Code in Feature-Branches | Umsetzungsphase |
| `test-automator.md` | Tester — schreibt Tests, **entkoppelt vom Developer** | Nach jedem Developer-Lauf |
| `code-reviewer.md` | Reviewer — prüft PRs auf Logik/Struktur | Vor Merge in `main` |
| `devops-engineer.md` | DevOps — CI/CD, Deploy | Setup + Release-Zeitpunkt |

**Regel:** Test-Agent und Developer-Agent dürfen nicht dieselbe Session sein — sonst Test-Gaming (AI testet seinen eigenen Code gegen sich selbst).

---

## Branching & Workflow

Siehe `docs/branching.md` für die vollständige Strategie.

**Kurzform:**
- `main` = stabil, geschützt (Branch Protection aktiv)
- `feature/*` = neue Features
- `fix/*` = Bugfixes
- **Parallele Arbeit:** Worktrees (`claude --worktree`), nie zwei Agents auf derselbe Datei
- **Merge:** Pull Request → Review-Agent → CI grün → manueller Merge (oder Auto-Merge bei Level 2)

---

## Security & Kosten

Siehe `docs/security.md` für Permission-Baseline, Kosten-Limits, Incident-Response.

**Pflicht-Checks vor jedem Release:**
- Spending-Cap im Anthropic-Konto aktiv
- `.claude/settings.json` Deny-Liste eingehalten
- Keine hardcoded Credentials
- PR-Review durch `code-reviewer.md` bestanden

---

## GitHub-Integration

- `claude-code-action@v1` installiert via `/install-github-app`
- Secret: `ANTHROPIC_API_KEY`
- Workflow: `.github/workflows/claude-assistant.yml`
- **Trigger-Policy:** keine `issue_comment`-Trigger ohne `author_association`-Filter (CVE-2025-54794/54795)

---

## Remote Control

Optional: `claude remote-control` für Mobile-/Web-Zugriff auf laufende Sessions.
- Auth: claude.ai OAuth (nicht API-Key, nicht long-lived Token)
- Outbound HTTPS only, keine Firewall-Öffnung
- Session endet bei >10min Netzwerkausfall

---

## Shortcuts (projektspezifisch — erweitern nach Bedarf)

| Shortcut | Aktion |
|---|---|
| `/status` | Projekt-Kurzstatus |
| `/review [pr-nr]` | Review-Agent auf PR loslassen |
| `/test` | Test-Agent ausführen |

---

## Offene Punkte / Nächste Schritte

_Hier projektspezifisch füllen._

- [ ] [Erste Aufgabe]
