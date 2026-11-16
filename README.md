# deploy-workflows

Zentraler GitHub-Actions-Workflow, der alle vmDesign-Websites baut und per SFTP auf den Verpex-Server lädt.

Jedes Website-Repo ruft ihn mit einer kurzen `deploy.yml` auf (Vorlage siehe Anleitung).

## Was der Workflow macht

1. Code holen, Paketmanager erkennen (npm, pnpm, yarn), Abhängigkeiten installieren
2. `BUILD_ENV` (falls vorhanden) als `.env` bereitstellen und bauen
3. Prüfen, ob `dist/index.html` existiert – sonst Abbruch ohne Upload
4. Probelauf: Was würde am Server gelöscht? Abbruch bei Ordnern mit Domain-Namen oder zu vielen Löschungen
5. Upload in zwei Durchgängen: erst Assets, dann HTML-Seiten und Löschen veralteter Dateien
6. Live-Check der Website (optional)

Davor läuft der Job **Änderungen prüfen**: Bei `only-if-content-changed: true` vergleicht er den Sanity-Fingerabdruck und Commit mit `https://<site>/deploy-info.json` und überspringt den Build, wenn nichts geändert wurde. Jeder Deploy schreibt diese Datei neu.

## Eingaben

| Name | Pflicht | Standard | Bedeutung |
|---|---|---|---|
| `remote-path` | ja | – | Zielordner am Server, eigener Ordner pro Website |
| `site-url` | nein | – | URL für den Live-Check |
| `node-version` | nein | `22` | Node-Version |
| `build-command` | nein | `<pm> run build` | Eigener Build-Befehl |
| `output-dir` | nein | `dist` | Ordner mit der fertigen Seite |
| `working-directory` | nein | `.` | Ordner mit der `package.json` |
| `keep-remote` | nein | `.well-known cgi-bin .ftpquota` | Pfade am Server, die nie angefasst werden |
| `max-delete` | nein | `200` | Abbruch bei mehr Löschungen (0 = keine Grenze) |
| `sanity-project-id` | nein | – | Für den nächtlichen Inhaltsvergleich |
| `sanity-dataset` | nein | `production` | Sanity-Dataset |
| `only-if-content-changed` | nein | `false` | Nur bauen, wenn sich Sanity oder Code seit dem letzten Deploy geändert hat (nächtlicher Lauf) |

## Secrets (im Website-Repo)

`SFTP_HOST`, `SFTP_USER`, optional `SFTP_PORT` (Standard 22), `SFTP_PASSWORD` **oder** `SSH_PRIVATE_KEY`, optional `SFTP_KNOWN_HOSTS`, `BUILD_ENV` und `SANITY_READ_TOKEN` (nur bei privatem Dataset).

## Änderungen ausrollen

Die Websites verwenden `@v1`. Nach einer Änderung hier:

```bash
git commit -am "…" && git push
git tag -f v1 && git push -f origin v1
```

Größere, inkompatible Änderungen als `v2` taggen und die Websites einzeln umstellen.
