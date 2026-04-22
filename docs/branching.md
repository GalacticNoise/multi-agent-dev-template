# Branching & Workflow

_Verbindliche Branch-Strategie + Multi-Agent-Regeln für Projekte aus diesem Template._

---

## Branches

| Branch | Zweck | Regeln |
|---|---|---|
| `main` | Stabile Version, deploy-ready | **Branch Protection aktiv.** Keine direkten Pushes. PRs mit Review + grüner CI zwingend. |
| `feature/*` | Neue Features | Pro Feature ein Branch. Name: `feature/<kurzname>` (z.B. `feature/auth-oauth`). Merge via PR. |
| `fix/*` | Bugfixes | Pro Fix ein Branch. Name: `fix/<kurzname>`. Merge via PR. |
| `docs/*` | Nur Dokumentations-Änderungen | Optional. Erlaubt schnelle Merges ohne vollen CI-Lauf (wenn so konfiguriert). |
| `experiment/*` | Proof-of-Concepts, WIP, Spikes | Kein Review-Zwang. Werden oft nicht gemerged, dienen Lernzwecken. |

**Nicht verwendet** (bewusst einfach gehalten):
- `develop` — kommt aus GitFlow, für Solo-Entwicklung Overkill
- Long-lived Release-Branches — nur bei Bedarf (z.B. Hotfix-Support auf v1.x nach v2-Release)

---

## Multi-Agent-Regeln

### Parallel-Arbeit über Worktrees

Statt mehrere Claude-Sessions im selben Verzeichnis: **immer Worktrees**.

```bash
# Neuer Worktree für ein Feature
git worktree add ../projekt-feature-x feature/x

# Claude Code mit Worktree-Flag starten
claude --worktree

# Oder Server-Modus mit on-demand Worktrees
claude remote-control --spawn worktree
```

**Worktree-Konventionen:**
- Name des Worktree-Ordners = Branch-Name (mit `/` → `-`)
- Jeder Worktree hat seinen eigenen `CLAUDE.md`-Context (wird aus `main` geerbt)
- Worktrees werden nach Merge aufgeräumt: `git worktree remove <pfad>`

### Regel: Nie zwei Agents auf dieselbe Datei

Worktrees lösen **Filesystem-Isolation**, aber **nicht semantische Merge-Konflikte**. Wenn zwei Agents parallel an derselben Datei arbeiten, produzieren sie konkurrierende Änderungen, die später mühsam aufzulösen sind.

**Faustregel:**
- Feature A berührt `src/auth/*` → Agent 1
- Feature B berührt `src/billing/*` → Agent 2
- ✅ Parallel arbeiten
- Feature A berührt `src/shared/logger.ts` → Feature B auch `src/shared/logger.ts`
- ❌ Nicht parallel — sequenziell arbeiten oder bewusst eines priorisieren

**Optional:** `clash-sh/clash` als PreToolUse-Hook einbauen — detektiert Konflikte, BEVOR die Änderungen geschrieben werden. Wird als Extension in späteren Milestones eingeführt.

### Test-Agent entkoppelt vom Developer-Agent

**Kritisch:** Der Agent, der Code schreibt, darf NICHT derselbe sein, der Tests schreibt. Sonst schreibt die AI sich ihre eigenen Tests so zurecht, dass sie bestehen — inklusive mitgedachter Logikfehler.

- **Developer-Agent** (`fullstack-developer.md`) arbeitet im Feature-Branch
- **Test-Agent** (`test-automator.md`) wird in **separater Session** oder **nach PR-Erstellung** aufgerufen
- Tests durchlaufen zusätzlich Human-Review bei neuen Test-Cases

### Rollen-Reihenfolge pro Feature

```
1. Planner (agent-organizer.md)     → definiert Scope, betroffene Files, Teststrategie
2. Developer (fullstack-developer)  → implementiert im feature/*-Branch
3. Test-Agent (test-automator.md)   → schreibt Tests (separate Session!)
4. Review-Agent (code-reviewer.md)  → reviewed PR automatisch via @claude-Mention
5. DevOps (devops-engineer.md)      → nur bei Setup-/Deploy-Änderungen
```

---

## Pull Request Workflow

### PR-Erstellung
- Branch auf GitHub pushen: `git push -u origin feature/<name>`
- PR gegen `main` eröffnen
- Title: klar, imperativ, < 70 Zeichen (z.B. „Add OAuth2 login flow")
- Description: Problem, Lösung, Test-Plan

### Automatisiertes Review
- `claude-code-action` reagiert auf `@claude review` in PR-Kommentar
- Output: strukturiertes Review mit Logik/Struktur/Security-Befunden
- **Policy:** Nur bei **trusted PRs** (keine Forks von externen Contributors), siehe `docs/security.md` CVE-2025-54794/54795

### Merge-Regeln (Level 1 — Basic)
- ✅ CI grün
- ✅ Mindestens 1 Review (Claude-Review zählt, bei kritischen Features zusätzlich Human-Review)
- ✅ Kein offener Conversation-Thread
- Manueller Merge durch Steve

### Auto-Merge (Level 2 — später)
- Wird separat dokumentiert, nach erstem Pilotprojekt-Durchlauf
- Aktiviert nur wenn: Tests fail-fast, Coverage-Schwellwert, Review-Bestätigung

---

## Commit-Konventionen

**Format:** `<type>: <kurzbeschreibung>`

| Type | Verwendung |
|---|---|
| `feat` | Neues Feature |
| `fix` | Bugfix |
| `docs` | Nur Doku |
| `chore` | Build, Config, Deps, keine User-sichtbare Änderung |
| `refactor` | Code-Umbau ohne Verhaltensänderung |
| `test` | Tests hinzugefügt/geändert |
| `style` | Formatting, Whitespace |
| `perf` | Performance-Optimierung |

**Regeln:**
- Erste Zeile < 72 Zeichen
- Imperativ: „add", „fix", „update" — nicht „added", „fixed"
- Body optional, trennt „was" (Subject) von „warum" (Body)
- Keine AI-Attribution in Commit-Messages, ausser Steve explizit gewünscht

---

## Branch Protection — Konkrete Settings

Einmalige Einrichtung pro Repo (GitHub UI → Settings → Branches → Add rule für `main`):

- ✅ Require a pull request before merging
  - Required approvals: **1**
  - Dismiss stale reviews on new commits
- ✅ Require status checks to pass before merging
  - Require branches to be up to date
  - Required checks: `CI / build` (projektspezifisch ergänzen)
- ✅ Require conversation resolution before merging
- ✅ Do not allow bypassing (Include administrators)
- ❌ Keine Force-Pushes
- ❌ Kein Branch-Delete durch Non-Admins

---

## Aufräumen

**Gemergter Branch:**
```bash
git branch -d feature/<name>         # lokal löschen (nur wenn merged)
git push origin --delete feature/<name>  # remote löschen
git worktree remove ../projekt-<name>    # Worktree aufräumen
```

GitHub kann „automatically delete head branches" aktivieren (Settings → General) — spart den `--delete`-Schritt.
