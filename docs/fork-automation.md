# Fork-Automation für ChrMaass/ioBroker.tesla-motors

Diese Dateien sind **private Fork-Infrastruktur** für `ChrMaass/ioBroker.tesla-motors`.
Sie dürfen nicht in den Pull Request zum Community-Adapter übernommen werden.

## Grundregel

- Community-PR: `feature/335-fleet-telemetry` → `iobroker-community-adapters:master`
- Fork-Automation: nur im Fork `ChrMaass/ioBroker.tesla-motors`, idealerweise auf dem Fork-Default-Branch
- Beim Aktualisieren des Telemetrie-Branches immer `upstream/master` übernehmen, **nicht** den Fork-`master` in `feature/335-fleet-telemetry` mergen.

So bleiben die privaten Workflows aus `.github/workflows/fork-*` außerhalb des Upstream-PRs.

## Workflows

### `Fork Upstream Sync`

Datei: `.github/workflows/fork-upstream-sync.yml`

Aufgabe:

1. Prüft regelmäßig, ob `iobroker-community-adapters/ioBroker.tesla-motors:master` neue Commits enthält.
2. Merged diese Commits in den Fork-Branch `feature/335-fleet-telemetry`.
3. Führt `npm run lint -- .` und `npm test` aus.
4. Erstellt bei Erfolg einen PR im Fork gegen `feature/335-fleet-telemetry`.
5. Erstellt bei Merge-Konflikten oder Testfehlern ein Issue im Fork.
6. Versucht optional, Copilot einzubinden.

Copilot-Optionen:

- Für PR-Reviews nutzt der Workflow `gh pr edit --add-reviewer @copilot`.
- Für Copilot Coding Agent Issues nutzt er die GitHub GraphQL API und sucht nach `copilot-swe-agent`.
- Dafür sind optional User-Tokens als Repository-Secrets sinnvoll:
  - `COPILOT_REVIEW_TOKEN` für Copilot-Code-Review-Anfragen
  - `COPILOT_ASSIGNMENT_TOKEN` für das Zuweisen von Issues an Copilot Coding Agent

Ohne diese Secrets werden PR/Issues trotzdem angelegt; die Copilot-Zuweisung wird dann übersprungen oder nur best-effort versucht.

### `Fork ioBroker Deploy`

Datei: `.github/workflows/fork-iobroker-deploy.yml`

Aufgabe:

1. Läuft auf einem Self-hosted Runner in Christians Heimnetz.
2. Checkt standardmäßig `feature/335-fleet-telemetry` aus.
3. Vergleicht den auszurollenden SHA mit `/opt/iobroker/.codex/tesla-motors-deployed-sha`.
4. Führt bei neuen Commits `npm run lint -- .` und `npm test` aus.
5. Installiert den Adapter in `/opt/iobroker` und startet `tesla-motors.0` neu.
6. Legt vorher ein Backup unter `/opt/iobroker/backups/` an.

Empfohlene Runner-Labels:

- `self-hosted`
- `maass-home`
- `iobroker`

Der Runner sollte auf dem ioBroker-Host laufen und als Benutzer `iobroker` gestartet werden, damit `/opt/iobroker` ohne `sudo` beschreibbar ist.


## Bekannte Betriebsdetails

- Die Workflows setzen `GH_REPO` und übergeben bei Label-Operationen zusätzlich `--repo`, damit `gh` im Fork nicht versehentlich das Parent-Repository verwendet.
- Der Deploy-Workflow nutzt auf dem Self-hosted Runner bewusst **kein** `actions/setup-node`-npm-Caching. Der globale npm-Cache auf dem ioBroker-Host kann sehr groß werden; ein Cache-Upload würde den Deploy-Job unnötig blockieren.

## Aktivierung

Scheduled Workflows laufen nur, wenn diese Dateien auf dem Default-Branch des Forks liegen.
Da direkte Pushes auf `master` bewusst vermieden werden, sollte diese Änderung zuerst als PR im Fork landen und danach explizit in den Fork-`master` gemerged werden.

## Manuelle Nutzung

- Upstream-Sync sofort starten: GitHub Actions → `Fork Upstream Sync` → `Run workflow`
- ioBroker-Deployment testen: GitHub Actions → `Fork ioBroker Deploy` → `Run workflow` mit `dry_run=true`
- ioBroker-Deployment erzwingen: `force=true`, `dry_run=false`
