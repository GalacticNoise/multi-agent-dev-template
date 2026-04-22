# multi-agent-dev-template

_Wiederverwendbares Template-Repo für code-getriebene Projekte mit Claude Code + GitHub + CI/CD._

Entstanden als idea-0017 M1 im Idea-Management-System von Steve Weber. Ziel: stack-agnostisches Fundament, das bei jedem neuen Code-Projekt (Web-App, Game, Desktop/Mobile-App, GUI) in wenigen Minuten einsatzbereit ist.

---

## Was drin ist

- **`.claude/`** — Multi-Agent-Setup
  - `agents/` — 5 spezialisierte Subagent-Prompts (Planner, Developer, Tester, Reviewer, DevOps)
  - `commands/` — projektspezifische Slash-Commands (leer, pro Projekt füllen)
  - `hooks/` — PreToolUse/PostToolUse Hooks (leer, pro Projekt füllen)
  - `skills/` — wiederverwendbare Fähigkeiten (leer, pro Projekt füllen)
- **`.github/workflows/`** — CI/CD-Workflows (`claude-assistant.yml` + stack-spezifische Pipelines pro Projekt)
- **`docs/`**
  - `branching.md` — Branch-Strategie, Worktree-Konventionen, Merge-Regeln
  - `security.md` — Permission-Baseline, Kosten-Limits, Incident-Response
- **`CLAUDE.md`** — Projekt-Memory-Template (beim Klonen anpassen)

---

## So nutzt man das Template

1. Repo klonen oder als GitHub-Template „Use this template" verwenden
2. `CLAUDE.md` mit eigenem Projekt-Kontext füllen (Name, Zweck, Stack)
3. Spending-Cap im Anthropic-Konto setzen (siehe `docs/security.md`)
4. `claude-code-action` via `/install-github-app` installieren
5. Stack-spezifischen CI-Workflow unter `.github/workflows/` ergänzen (Node, Python, Flutter, Unity — Beispiele folgen)
6. Loslegen: `claude` starten, Planner-Agent für Scoping aufrufen

---

## Stack-Support

- ✅ Node/TypeScript (Basis — im Template vorbereitet)
- 🚧 Python — als Extension (idea-0017 M3.a)
- 🚧 Flutter/React Native — als Extension (idea-0017 M3.c)
- 🚧 Unity/Godot — als Extension (idea-0017 M3.d/e)

---

## Lizenz

MIT — siehe [LICENSE](LICENSE).

Agent-Prompts unter `.claude/agents/` sind geforkte MIT-Lizenz von [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) mit Anpassungen für Stevens Use-Case (Deutsch, Tool-Whitelist-Härtung).

---

## Sicherheit

Dieses Template ist für **Solo-Entwicklung** konzipiert. Bei Team-Einsatz oder Fork mit externen Contributors:
- `docs/security.md` vollständig lesen
- `issue_comment`-Trigger in Workflows mit `author_association`-Filter härten (CVE-2025-54794/54795)
- Prompt-Injection-Fläche minimieren

---

## Status

- **v1.0 (2026-04-22):** Skeleton + 5 Agents + Branching-Docs + Security-Baseline — M1 aus idea-0017.
- **v1.1 (geplant):** Erste Pilot-Integration mit echtem Projekt.
