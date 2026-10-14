---
description: Schliesst eine MAT-Session (multi-agent-dev-template) vollstaendig ab — Gate (Definition-of-Done, Landeplatz, Secrets, Zeitstempel), Memory im Verify-Modus, Cross-Workspace-Promotion-Gate nach shared/learnings.md, Commit conditional, Sync nach Modell (main), Worktree-Hygiene, naechste Schritte mit Ausfuehrungsort, genau ein Session-Titel, Abschlussblock. MUSS sofort ausgeloest werden wenn Steven sagt "abschluss", "abschliessen", "feierabend", "fertig", "session zu", "session ende", "ich bin durch", "mach zu" oder /abschluss eingibt — egal wie kurz die Eingabe wirkt. NICHT ausloesen wenn Steven nur "push" sagt — das ist ein simpler git push ohne Routine.
---

Steven schliesst die MAT-Session ab.

> **Kern-Vorlage:** `C:/Users/steve/Documents/Claude/shared/routines/abschluss-kern.md`
> (idea-0025 M4). Schritte ohne Zusatz sind der wörtliche Kern; Schritte mit dem Suffix
> **`— MAT-eigen:`** sind dokumentierte Erweiterungen mit je einer Zeile Begründung.
> Kern-Änderungen werden pull-basiert beim nächsten Touch nachgezogen, nicht automatisch verteilt.

## Leitprinzip — Verify, nicht Re-Execute

Der reale Workflow ist kontinuierlich: Commit/Push/PR/Merge, Memory und Capture passieren
**während** der Session. Diese Routine läuft denselben Boden **nicht nochmal ab**, sondern
**prüft, ob es erledigt ist**, und tut nur den Rest. Jeder Schritt fragt zuerst „schon
erledigt?" — bei Ja: 1-Zeilen-Bestätigung + skip. **Ein Skip wird gemeldet**, nicht
stillschweigend übergangen.

_Herkunft: MUO 2026-06-27, promoted in `shared/learnings.md` (`workflow`), zurückportiert
nach IDS 2026-08-22. Der Übertragbarkeitsbeleg liegt damit vor._

---

## Trigger (wörtlich identisch in jedem Workspace)

**Klartext-Trigger:** `abschluss`, `abschliessen`, `feierabend`, `fertig`, `session zu`,
`session ende`, `ich bin durch`, `mach zu` — sowie `/abschluss`.
Sofort auslösen, egal wie kurz die Eingabe wirkt.

**NICHT-Trigger:** `push` allein → nur `git push`, keine Routine.

---

## Kern-Schritte

Reihenfolge ist Teil des Kerns; Nummerierung bleibt monoton. Workspace-Erweiterungen werden
**eingeschoben** und als solche gekennzeichnet (`— MAT-eigen:` als Suffix im Titel).

### 1. Gate — „ist die Arbeit gelandet und tragfähig?"

Drei Teilprüfungen, je 1 Zeile Ergebnis. **(a) und (b) melden**, blockieren nicht —
**nur (c) stoppt den Push.**

**a) Definition-of-Done** — der workspace-eigene Beweis, dass der Stand tragfähig ist.
Merge-Modell MAT: Worktree + PR.
MAT: keine Test-Suite, keine Lint-CI (nur `claude-assistant.yml`) — Beweis ist der bestandene PR-Review durch `code-reviewer.md`; reine Doku-Änderung → `✓ Gate: keine Code-Änderung`. Rot → melden, Ergebnis landet im ━━━-Block.

**b) Landeplatz-Status** — ist der Stand dort angekommen, wo er hingehört?
Repos mit PR-Modell: `gh pr checks <nr>` (required Check namentlich nennen).
Repos mit Worktree→Default-Merge: Diff-Verifikation nach dem Merge (Schritt 5b).

**c) Secrets-Scan vor dem Push** — überall identisch, unparametrisiert:
```bash
git diff origin/main...HEAD | grep -nEi '(api[_-]?key|secret|token|webhook|password|BEGIN [A-Z ]*PRIVATE KEY)\s*[:=]' || echo "✓ kein offensichtliches Secret im Diff"
```
Treffer prüfen (kann ein legitimer Variablenname sein) — bei echtem Secret: **Push stoppen**, melden.

**d) Zeitstempel** (meldet, blockiert nicht) — heutige Uhrzeiten nach „jetzt" sind geschätzt statt abgelesen (PF#570):
```bash
bash /c/Users/steve/Documents/Claude/shared/scripts/now.sh --check-diff "$(git merge-base origin/main HEAD)"
```
Jede Uhrzeit, die die Routine selbst schreibt (Memory, History, `last_session`), kommt aus `now.sh` ohne Argument.

**Output (1 Zeile):** `✓ Gate: DoD ✓ · Landeplatz ✓ · kein Secret`

> **Bündeln, nicht einzeln absetzen.** Die drei Teilprüfungen sind deterministisch — ein Sammel-Skript im Repo (`scripts/abschluss-gate.sh` o.ä.) druckt sie als ≤5 Zeilen und setzt den Exit-Code (0 grün · 1 Secrets-Treffer, Push stoppen · 2 Struktur kaputt). Siehe „Token-Disziplin" in `shared/routines/abschluss-kern.md`.

**Reine Nicht-Code-/Diskussions-Session:** (a) und (b) skippen mit
`✓ Gate: keine Code-Änderung`; (c) läuft trotzdem, sobald etwas gepusht wird.

**Scope-Regel für nachgeholte Gates:** Ein hier nachgeholtes Review/Gate ist richtig — die
daraus folgenden Fixes nur, wenn Klasse-1-trivial (Label-/Textänderung); alles Grössere geht
als Backlog-Eintrag oder Task-Chip raus. _Sonst wird der Abschluss zur zweiten Arbeitssession
(beobachtet MUO 2026-08-28)._

### 2. Memory-/Session-Notiz

**Verify zuerst:** Wurde während der Session schon mitgeschrieben? Dann nur sweepen —
„gibt es eine Erkenntnis, die noch *nicht* erfasst ist?" — kein voller Re-Pass.

Ablageort ist workspace-eigen (Repo-`context/memory/` vs. Projekt-Memory unter
`~/.claude/projects/<slug>/memory/`). Inhalt in beiden Fällen: Thema · Ablauf ·
Schreib-Output als Tabelle (Datei | Aktion) · Offen/Nächste Session · Learnings-Kandidaten.

**Anhängen per `cat >>`-Heredoc (Bash), nicht per Read+Edit.** Die neue Sektion kommt ans
Datei-Ende; ein Append braucht null Dateiinhalt im Kontext. Nur die Session-Nummer vorab
ermitteln: `grep -c '^## Session' <memory-datei>` → +1.

**Output (1 Zeile):** `✓ Memory: N neu / M ergänzt` oder `✓ Memory: bereits in-session gepflegt`.

### 3. Promotion-Gate → `Claude/shared/learnings.md` (Cross-Workspace)

_Kanonisch — dieser Schritt ist in allen Workspaces wörtlich derselbe. Eingeführt
2026-05-01 aus idea-0025 M1._

**Nur ausführen, wenn in dieser Session echte Learnings entstanden sind** (Fehler, Entscheide,
Patterns — keine Routinearbeit). Sonst: `✓ Promotion-Gate: keine cross-workspace-Lesson heute` + skip.

**Auch bei glattem Verlauf einmal fragen:** was wäre schneller/besser gegangen? _Sonst
entstehen Learnings nur aus Fehlern (Hansei-Prinzip) — eine Frage, kein Pflichtfeld._

Session-Learnings als Tabelle mit Claudes **eigener** Empfehlung — kein Fliesstext:

```
| # | Learning | Empfehlung | Begründung |
|---|----------|------------|------------|
| 1 | Kurzbeschreibung | **Ja** | Ein Satz warum cross-workspace relevant |
| 2 | Kurzbeschreibung | **Nein** | Ein Satz warum nicht generalisierbar |
```

Die Tabelle ist **Protokoll, kein Gate** — Steven folgt ohnehin immer der Empfehlung
(bestätigt 2026-06-03). **Keine Rückfrage.** Claude setzt seine `Ja`-Empfehlungen direkt um;
`Nein`-Zeilen bleiben Protokoll.

**Für jede `Ja`-Zeile — geschrieben wird per Skript, nie per Read+Edit.** `shared/learnings.md`
(~590 KB) und `PITFALLS.md` (~325 KB) werden von Read **vollständig** geladen — zusammen ~230k
Tokens, um eine Zeile anzuhängen (gemessen 2026-08-28,
`idea-system/knowhow/projects/idea-0025/abschluss-token-analyse.md`). Das Einfügen selbst ist
deterministisch und gehört deshalb ins Skript:

```bash
bash /c/Users/steve/Documents/Claude/shared/scripts/append-entry.sh <modus> "…"
```

- **Dedup-Grep (Pflicht, vor dem Schreiben):** 2–3 Schlüsselbegriffe des Symptoms in
  `shared/learnings.md` UND `shared/PITFALLS.md` greppen (billig — grep druckt nur Trefferzeilen,
  kein Voll-Read). Treffer → **keine neue Zeile**:
  bestehenden Eintrag ergänzen oder — bei wiederholter Lehre — einen Guard bauen
  (Test/CI-Check/Default im Skript). Wiederholte Promotion derselben Lehre ist ein Befund
  über die Ablage, nicht über die Lehre (shared-Eintrag 2026-08-23; 4×-cp1252-Fall).
- Generalisierte Variante formulieren (workspace-spezifischen Kontext entfernen — gilt es auch
  für IDS / MUO / BDQ / TRD / FIO / CCC / EA / ADT?). Wörtlich übernehmen, falls schon allgemein.
- Eintrag in `shared/learnings.md` unter dem aktuellen Monats-Header (`## YYYY-MM`), neueste
  oben — der Modus legt den Header bei Monatswechsel selbst an:
  ```bash
  … append-entry.sh learning-shared "YYYY-MM-DD | [kategorie] | Erkenntnis (1-2 Sätze) | (Quelle: MAT, <kontext>)"
  ```
- Bei neuem Pitfall zusätzlich `shared/PITFALLS.md` — Nummernvergabe, Einfügen in den
  Top-Block und Kollisionsschutz übernimmt der Modus:
  ```bash
  … append-entry.sh pitfall "**Titel** | Symptom | Fix | Quelle"
  ```
- **Arbeitsteilung:** reproduzierbare Symptom→Fix-Fallen gehören als Tabellen-Zeile NUR nach
  `PITFALLS.md` — nicht zusätzlich als `[fehler]`-Learning-Zeile (keine Dreifachablage).
  `learnings.md` trägt Diagnosen und Prozess-Lehren ohne Wiederholungs-Fix; existiert die
  Falle schon als Learning, wird die Learning-Zeile mit ` [→PF#NNN]` markiert.
- **Lokaler Eintrag im Workspace bleibt zusätzlich** — kein Move, sondern Promotion. Der
  Audit-Trail dazu ist der Marker ` [→shared]` am Zeilenende des lokalen Eintrags — ohne
  Marker gilt die Promotion als nicht abgeschlossen (macht Re-Promotion greppbar;
  eingeführt 2026-08-28, idea-0025). Der Marker wird **beim Schreiben** mitgegeben, sonst
  braucht es einen Nach-Edit und damit wieder einen Voll-Read:
  ```bash
  … append-entry.sh learning-local "YYYY-MM-DD | [kategorie] | Erkenntnis [→shared]"
  ```
  Zieldatei ist `$LOCAL_LEARNINGS`, sonst der erste Treffer aus `context/learnings.md`,
  `docs/learnings.md`, `LEARNINGS.md`, `learnings.md` im Repo-Root.

Kategorien: `system`, `prozess`, `agent`, `fehler`, `entscheid`, `workflow`.

> ⚠️ **`Claude/shared/` ist seit 2026-08-21 ein eigenes Git-Repo**
> (`GalacticNoise/claude-shared`, privat — idea-0039 M0). Promotionen landen **ausserhalb**
> des Workspace-Repos und werden von dessen Commit **nicht** erfasst. Sie brauchen einen
> **eigenen Commit + Push**, sonst bleiben sie ungesichert. Commit, Rebase, Auflösung einer
> Pitfall-Nummern-Kollision aus einer Parallel-Session und Push macht ein Aufruf:
> ```bash
> bash /c/Users/steve/Documents/Claude/shared/scripts/append-entry.sh shared-push "THEMA"
> ```
>
> Der Aufruf **bricht ab (Exit 2)**, wenn ausser `learnings.md`/`PITFALLS.md` weitere Dateien
> im Arbeitsbaum dirty sind: er committet mit `git add -A` und nähme fremde Arbeit sonst
> ungeplant mit (passiert 2026-09-06 — sieben liegengebliebene Dateien, 2199 Zeilen). Die
> Meldung listet die Dateien. Erst sie committen oder zurücksetzen, dann shared-push
> wiederholen; bewusst alles mitnehmen mit `APPEND_ENTRY_ALLOW_DIRTY=1` davor. Die
> Learnings-/Pitfall-Zeilen stehen zu dem Zeitpunkt schon in den Dateien — es geht nichts
> verloren, nur Commit und Push fehlen noch.
>
> Steht der Haupt-Checkout von `shared/` auf einem **fremden Branch** (Parallel-Session),
> landet die Promotion trotzdem auf `master`: der Aufruf committet sie in einem temporaeren
> Worktree auf `origin/master` und pusht `HEAD:master`, der fremde Checkout bleibt unberuehrt.
> Die zweite Output-Zeile sagt das an. Fruehere Fassungen pushten `HEAD` — am 2026-09-20 lagen
> so 14 Promotions-Commits auf einem Feature-Branch statt auf `master`.

**Output (1 Zeile):** `✓ Promotion: <kategorie> in shared/learnings.md (+ PITFALLS.md falls Pitfall), shared/ gepusht`.

### 4. Commit (conditional)

**Erst prüfen, ob überhaupt etwas zu committen ist** — Auto-Commit-Hooks fangen File-Edits
beim Stop-Trigger; ohne Prüfung entsteht „nothing to commit"-Lärm oder ein leerer zweiter Commit.

```bash
DIRTY=$(git status --porcelain)
if [ -n "$DIRTY" ]; then
  git add -A
  git commit -m "session: YYYY-MM-DD — THEMA"
else
  echo "✓ Working tree clean (Auto-Commit-Hook hat bereits committet) — skip"
fi
```

THEMA sprechend und themenbasiert, nie „Diverses"; max. 2 Themen, Komma-getrennt.

> **Workspaces mit strikter Worktree+PR-Disziplin** (nie auf dem Default-Branch committen)
> ersetzen diesen Schritt bewusst durch einen **read-only** Hygiene-Check — siehe Schritt 6.
> Das ist eine dokumentierte Eigenheit, keine Auslassung.

### 5. Sync

**a) Push/Merge** nach dem Modell des Workspaces (Merge-Modell unter 1a): PR-Modell → der Merge ist
in-Session schon passiert, hier nur verifizieren; Worktree→Default-Merge → jetzt mergen.

**b) Verlust-Verifikation — Pflicht bei jedem Merge-Skript.** Nach dem Merge muss
`git diff main <branch> --stat` **leer** sein. Ist es das nicht, fehlen Dateien im
Default-Branch: je File `git checkout <branch> -- <file>` und nachcommitten.

**c) Push** des Default-Branches.

### 6. Worktree-Hygiene (read-only — meldet, committet nichts)

```bash
git status --porcelain            # uncommittete Reste / Scratch-Müll im Worktree?
git rev-parse --abbrev-ref HEAD   # Haupt-Checkout auf dem Default-Branch?
git worktree list                 # gemergte Worktrees als „aufräumbar" auflisten
```
Gemergte Feature-Worktrees nur **vorschlagen** — `git worktree remove` macht Steven, und
**nie** während laufender Background-Tasks (zieht dem Lauf die Dateien weg → Fake-Failures).

### 7. Nächste Schritte festlegen — mit Ausführungsort

Nicht „was kommt als Nächstes", sondern **wo es hingehört**. Ein Schritt ohne Ausführungsort
wird in der nächsten Session neu verhandelt; das kostet mehr als das Festlegen.

Jeden offenen Faden dieser Session in **genau eine** der drei Klassen einsortieren:

| Symbol | Klasse | Wann |
|---|---|---|
| 🟢 | **Hier weiter** | Gleicher Workspace **und** gleicher Themenstrang **und** Kontext unter ~70 %. Nur dann ist der bereits aufgebaute Kontext den Verzicht auf einen frischen wert. |
| 🔴 | **Neue Session** | Anderer Workspace (**harte Grenze** — fremde Repos werden nie aus dieser Session geändert), anderer Themenstrang, oder Kontext über ~70 %. Braucht einen selbsttragenden Kickoff-Prompt, keinen Verweis auf „die letzte Session". |
| 🎫 | **Task-Chip** | Unabhängig von dieser Session, parallelisierbar, kein Sitzungskontext nötig. |

**🎫-Zeilen werden sofort angelegt, nicht nur empfohlen** — `spawn_task` mit selbsttragendem
Prompt (Pfade absolut, Randbedingungen inline), die zurückgegebene `task_id` kommt in die
Tabelle. Ein „das sollte man mal"-Satz ohne Chip ist verloren.

**Selbstprüfung vor dem Senden:** Jede 🎫-Zeile trägt eine echte `task_id` — fehlt sie, Chip
JETZT anlegen, nicht empfehlen.

**Ausgabe** direkt vor dem ━━━-Block:

```
Nächste Schritte:
| # | Schritt | Wo | Grund |
|---|---------|----|-------|
| 1 | [konkreter Schritt] | 🟢 hier | [ein Satz] |
| 2 | [konkreter Schritt] | 🔴 MAT-Session | [ein Satz] |
| 3 | [konkreter Schritt] | 🎫 task_xxxxxxx | [ein Satz] |
```

Das Feld `Nächste Session:` im ━━━-Block trägt danach **genau die Zeile 1** — der Block bleibt
einzeilig, die Tabelle liefert die Tiefe. Steht in Zeile 1 ein 🟢, endet die Antwort mit der
Frage, ob jetzt weitergearbeitet werden soll.

### 8. Session-Rename (Verify-Modus, **genau EIN** Titel, Claude setzt ihn selbst)

**Verify zuerst:** Titel der eigenen Session lesen (`mcp__ccd_session_mgmt__get_session` mit
`session_id: "self"`). Ist er schon themenscharf und passt noch → `✓ Titel passt` + skip.

Sonst **genau EINEN** Titel festlegen, Pflicht-Format `KÜRZEL - Thema` (Kürzel aus
`Claude/CLAUDE.md`, Thema identisch zum Commit-Thema), und ihn **selbst setzen**:
`mcp__ccd_session_mgmt__set_session_title` mit `session_id: "self"` (deferred — vorher per
`ToolSearch` `select:mcp__ccd_session_mgmt__set_session_title` laden). Ausgabe:
`✓ Titel gesetzt: KÜRZEL - Thema`. Hat Steven den alten Titel selbst vergeben, fragt die App
vorher nach — die Freigabe ist dann seine, keine Rückfrage im Chat. Der Aufruf läuft **vor**
dem Abschlussblock, damit der Block der letzte Text bleibt.

_Warum kein Knopf mehr (2026-09-30, IDS):_ Der frühere Ein-Klick-Knopf
(`sendPrompt('/rename …')` via `mcp__visualize__show_widget`, 2026-08-14 in MUO verifiziert)
löst in der Desktop-App nichts mehr aus. Der Titel blieb nach dem Klick unverändert. Das
App-Werkzeug setzt den Titel direkt, prüfbar und ohne Klick.

_Drei Vorschläge widersprechen SOUL.md („eine Empfehlung machen, nicht Optionen aufzählen")._

**Fallback nur ohne das Werkzeug** (Terminal-CLI, Cloud-Session: die `ccd_session_mgmt`-Tools
fehlen): genau EIN Copy-Block `/rename KÜRZEL - Thema` direkt nach dem Abschlussblock.

---

## Der ━━━-Abschlussblock (kanonisch, Pflicht, immer allerletzter Text)

**Erlaubt sind genau zwei Ausgabeformen:** die 1-Zeilen-Bestätigung je Schritt **während**
des Ablaufs und der Block selbst. Eine Bullet-/✓-„Zusammenfassung" der Schritte davor ist
**verboten** — dieselben Fakten stünden sonst 2–3×. Ebenso: leere Klassen in der
Nächste-Schritte-Tabelle weglassen, keine „Nichts offen — –"-Füllzeilen.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ SESSION ABGESCHLOSSEN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Ziel erreicht:   [ja / teilweise / nein] — [ein Satz: was war das Ziel, wurde es gelöst]
Gate:            DoD [✓/✗] · Landeplatz [✓/⏳/✗] · Secrets [✓/⚠️]
Thema:           [Thema dieser Session]
Commit:          [hash]
main:       [hash] (lokal)
origin/main:[hash]  ✓ sync  /  ⚠️ [N ahead/behind]
Loose Ends:      [offene Fäden dieser Session — oder „keine"]
Nächste Session: [konkreter nächster Schritt]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Der Block steht als **Monospace-Codeblock** (Fenced Code Block, drei Backticks), nie als
Fliesstext — sonst zerfallen die ━-Linien und die Spalten fluchten nicht.

Danach kommt nichts mehr. Den Titel hat Schritt 8 schon vorher gesetzt; nur im Fallback ohne
Werkzeug steht direkt nach dem Block der `/rename`-Copy-Block.
Die Tabelle „Nächste Schritte" aus Schritt 7 steht **vor** dem Block, nicht darin.

**Re-Run-Kurzform:** Läuft `/abschluss` ein zweites Mal in derselben Session ohne neue
Änderungen, ersetzt ein 3-Zeilen-Block den vollen — `unverändert seit <hash>` · Sync-Stand ·
Verweis auf den ersten Block. „Niemals weglassen" bleibt damit gewahrt, ohne den fast
identischen Block ein zweites Mal zu drucken.

**Warum diese drei Felder über den Ur-Standard hinaus:**
- **Ziel erreicht** — trennt „committet und gepusht" von „Problem gelöst"; fängt das Muster
  *Arbeit gelandet, Frage offen*.
- **Gate** — ein rotes Gate bleibt sonst bis zur nächsten Session unsichtbar.
- **Loose Ends** — offene Fäden explizit auf Disk; zwischen zwei Sessions erinnert sich niemand.

Sync-Stand ermitteln:
```bash
git fetch origin --quiet
LOCAL=$(git rev-parse --short main)
REMOTE=$(git rev-parse --short origin/main)
AHEAD=$(git rev-list origin/main..main --count)
BEHIND=$(git rev-list main..origin/main --count)
```
Gleich → `✓ sync`; sonst → `⚠️ [N] ahead / [N] behind origin`.

Kein Padding, kein „Gute Frage".
