# Dividenden-Baukasten – Android-App

Dieses Projekt verpackt den Dividenden-Baukasten mit Capacitor in eine echte Android-App.
Die gesamte App steckt in `www/index.html`, der Ordner `android/` ist das fertige Android-Studio-Projekt.

## Weg A: APK automatisch mit GitHub bauen (empfohlen)

GitHub baut die App in der Cloud. Du brauchst kein Android Studio, nur einmalig Java für den Signaturschlüssel.

### 1. Signaturschlüssel erzeugen (einmalig)

Jede APK muss signiert sein. **Mit demselben Schlüssel** lassen sich spätere Versionen einfach über die alte installieren, deine Daten in der App bleiben erhalten. Geht der Schlüssel verloren, musst du die App deinstallieren und neu installieren (Tabelle und Einstellungen sind dann weg).

Java installieren, falls noch nicht vorhanden: https://adoptium.net (Temurin 21). Dann im Terminal:

```
keytool -genkeypair -v -keystore release.jks -alias dividenden -keyalg RSA -keysize 2048 -validity 10000
```

Du wirst nach einem Passwort und ein paar Angaben (Name usw.) gefragt. Name genügt, der Rest darf leer bleiben.
**`release.jks` und das Passwort sicher aufbewahren** (z. B. im Passwortmanager) und niemals ins Repository hochladen.

Datei in Text umwandeln, damit sie in ein GitHub-Secret passt:

- macOS: `base64 -i release.jks | pbcopy` (liegt dann in der Zwischenablage)
- Linux: `base64 -w0 release.jks > keystore-base64.txt`
- Windows (PowerShell): `[Convert]::ToBase64String([IO.File]::ReadAllBytes("release.jks")) | Set-Clipboard`

### 2. Repository anlegen und hochladen

Neues Repository auf GitHub anlegen (privat ist in Ordnung) und den Inhalt dieses Ordners hochladen, inklusive des versteckten Ordners `.github`. Mit Git:

```
git init
git add .
git commit -m "Dividenden-Baukasten"
git branch -M main
git remote add origin https://github.com/DEIN-NAME/dividenden-baukasten.git
git push -u origin main
```

Hinweis: Beim Hochladen über die GitHub-Webseite per Drag-and-drop wird der Ordner `.github` leicht vergessen – dann fehlt der Workflow.

### 3. Secrets eintragen

Im Repository: *Settings → Secrets and variables → Actions → New repository secret*. Vier Einträge anlegen:

| Name | Inhalt |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | der Base64-Text aus Schritt 1 |
| `ANDROID_KEYSTORE_PASSWORD` | dein Keystore-Passwort |
| `ANDROID_KEY_ALIAS` | `dividenden` |
| `ANDROID_KEY_PASSWORD` | dasselbe Passwort (keytool verwendet standardmäßig dasselbe) |

### 4. APK bauen

- **Testbuild:** *Actions → APK bauen → Run workflow*. Nach etwa 5 bis 8 Minuten liegt die APK unten am Lauf unter *Artifacts* (als ZIP, darin die APK).
- **Version veröffentlichen:** einen Tag pushen, z. B.

  ```
  git tag v1.0.0
  git push origin v1.0.0
  ```

  Die APK erscheint dann unter *Releases* im Repository und lässt sich direkt am Handy herunterladen.

Die Versionsnummer für Android wird automatisch hochgezählt, damit jedes Update über die vorherige Version installiert werden kann.

### 5. Auf dem Handy installieren

APK auf dem Handy herunterladen (bei einem privaten Repository vorher in der GitHub-App oder im Browser anmelden), antippen und „Installation aus dieser Quelle zulassen“ bestätigen. Google Play Protect warnt eventuell, weil die App nicht aus dem Play Store stammt – „Trotzdem installieren“ wählen.

## Weg B: Lokal mit Android Studio

### Einmalig installieren

1. **Node.js 22 oder neuer**: https://nodejs.org
2. **Android Studio** (aktuelle Version): https://developer.android.com/studio
   Beim ersten Start den Einrichtungsassistenten durchlaufen lassen (lädt Android SDK und Emulator).

### App bauen und auf dem Handy starten

Im Projektordner ein Terminal öffnen und ausführen:

```
npm install
npx cap sync android
npx cap open android
```

Android Studio öffnet das Projekt und lädt beim ersten Mal einige Minuten lang Gradle-Abhängigkeiten.

**Auf dem eigenen Handy testen:** Am Handy unter *Einstellungen → Über das Telefon* sieben Mal auf *Build-Nummer* tippen, dann unter *Entwickleroptionen* das *USB-Debugging* einschalten. Handy per USB verbinden, oben in Android Studio auswählen und auf den grünen Play-Knopf drücken.

**APK-Datei erzeugen** (zum Weitergeben oder Installieren ohne Kabel):
*Build → Generate App Bundles or APKs → Generate APKs*. Die Datei liegt danach unter
`android/app/build/outputs/apk/debug/app-debug.apk`. Auf dem Handy öffnen und die Installation aus unbekannten Quellen erlauben.

## In den Google Play Store

1. In Android Studio *Build → Generate Signed App Bundle or APK → Android App Bundle* wählen und einen neuen Signaturschlüssel anlegen. **Schlüsseldatei und Passwort gut aufbewahren** – ohne sie kannst du keine Updates veröffentlichen.
2. Entwicklerkonto in der Google Play Console anlegen (einmalig 25 US-Dollar): https://play.google.com/console
3. Neue App anlegen, die `.aab`-Datei hochladen, Store-Eintrag, Datenschutzerklärung und Inhaltseinstufung ausfüllen.
   Hinweis: Neue private Entwicklerkonten müssen die App vor der Veröffentlichung meist eine Zeit lang mit einer Gruppe von Testern im geschlossenen Test laufen lassen.

Vor einer Veröffentlichung solltest du die App-ID in `capacitor.config.json` (`de.dividenden.baukasten`) durch eine eigene ersetzen, z. B. `de.deinname.dividenden` – das geht danach nicht mehr. Nach dem Ändern `npx cap sync android` ausführen und in `android/app/build.gradle` `applicationId` und `namespace` anpassen.

## Live-Daten einschalten

Ohne Schlüssel nutzt die App ihre eingebaute Datenbank (rund 40 beliebte Dividendenwerte, Momentaufnahmen).
Für aktuelle Kurse und Dividenden:

1. Kostenlos registrieren bei https://site.financialmodelingprep.com (Basic-Tarif: 250 Abrufe pro Tag).
2. API-Schlüssel kopieren.
3. In der App unten *Live-Daten, Wechselkurse und Steuern* öffnen und den Schlüssel einfügen.
4. *Live-Daten aktualisieren* tippen. Neue Kürzel werden ab dann automatisch live geladen.

Pro Aktie braucht die App zwei Abrufe (Profil und Dividendenhistorie). Der kostenlose Tarif liefert vor allem US-Werte; für Kürzel, die er nicht abdeckt, bleiben die Datenbankwerte stehen und die App zeigt einen Hinweis. Deutsche Werte gibst du mit `.DE` ein (z. B. `ALV.DE`).
Der Schlüssel wird nur auf deinem Gerät gespeichert.

## Aktien suchen

Das Suchfeld über der Tabelle findet Aktien nach Name oder Kürzel:

- Beim Tippen durchsucht die App sofort die eingebaute Datenbank (kostet keine Abrufe).
- Mit Enter oder „Online suchen“ sucht sie bei Financial Modeling Prep in allen bekannten Aktien und ETFs weltweit (2 Abrufe). Suchergebnisse werden 30 Tage gespeichert, dieselbe Suche kostet danach nichts mehr.
- Ein Tipp auf ein Ergebnis fügt die Aktie hinzu und lädt Kurs und Dividenden. Danach springt der Cursor in die Stückzahl.
- „Als Kürzel übernehmen“ legt eine leere Zeile an, wenn du die Werte lieber selbst einträgst.

## Etwas an der App ändern

`www/index.html` bearbeiten, committen und pushen. Mit GitHub dann einen neuen Tag setzen (z. B. `v1.1.0`), lokal stattdessen `npx cap sync android` ausführen und in Android Studio auf Play drücken.
