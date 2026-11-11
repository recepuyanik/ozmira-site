# OZMIRA

[Türkçe](README.md) · [English](README.en.md) · Deutsch · [Español](README.es.md)

Eine lokale App, die deine eigenen KI-Systeme in einer einzigen 3D-Szene zeigt. Jedes System erscheint in eigener Farbe und Form (Galaxie, Hologramm-Faser, Doppelhelix, Schwarzes Loch, Kugel). Beginnt ein Agent zu arbeiten, läuft ein Signal vom Zentrum zu ihm, kehrt nach getaner Arbeit zurück, und die verbrauchten Tokens erscheinen in der Szene.

Kein Konto, keine Cloud. Deine Systemdefinitionen, Ereignisse und dein Zugangsschlüssel bleiben nur auf deinem Computer. Es gibt ein optionales anonymes Teilen der Nutzung, das standardmäßig ausgeschaltet ist (Details unten).

## Starten

Voraussetzungen: Python 3.8 oder neuer (mit 3.13 getestet) und ein aktueller Browser. Sonst ist nichts zu installieren.

| System | Ausführen |
| --- | --- |
| Windows | `ozmira.bat` doppelklicken |
| macOS | `ozmira.command` doppelklicken (beim ersten Mal: Rechtsklick, Öffnen) |
| Linux | `./ozmira.sh` |
| Alle | `python3 ozmira.py` (unter Windows: `py ozmira.py`) |

Der Browser öffnet sich von selbst. Falls nicht, rufe die im Terminal angezeigte Adresse `http://localhost:8787/` auf. Zum Beenden im Terminal Strg+C drücken.

Beim ersten Start erscheinen eine Beispielszene und ein Einrichtungsassistent. Im Assistenten benennst du das Zentrum und fügst deine Systeme hinzu. Es gibt fertige Vorlagen (Claude, ChatGPT, Gemini, Grok, Andere); Name, Stil und Farbe kannst du ändern. Später lässt sich alles mit der Schaltfläche Bearbeiten unten ändern.

## Als App-Fenster öffnen

Du kannst OZMIRA statt in einem Browser-Tab in einem eigenen Fenster ohne Adressleiste öffnen. Dafür muss auf deinem Computer Chrome, Edge oder Brave installiert sein (Edge ist unter Windows 10 und 11 bereits vorhanden).

| System | Ausführen |
| --- | --- |
| Windows | `ozmira-uygulama.bat` doppelklicken (öffnet kein Konsolenfenster) |
| macOS | `ozmira-uygulama.command` doppelklicken |
| Linux | `./ozmira-uygulama.sh` |
| Alle | `python3 ozmira.py --uygulama` |

Schließt du das Fenster, wird OZMIRA beendet. Das Fenster nutzt einen eigenen Profilordner (`.ozmira/pencere`); deine persönlichen Browser-Sitzungen und Anmeldungen bleiben also außen vor.

Um eine Verknüpfung auf dem Desktop (und unter Linux im Anwendungsmenü) anzulegen, führe einmal Folgendes aus:

```
python3 ozmira.py kisayol
```

Die Verknüpfung öffnet das App-Fenster und nutzt das OZMIRA-Symbol (`ozmira.ico`, `web/icon.png`). Es ist eine Verknüpfung, keine einzelne `.exe` und kein Installer, und sie setzt installiertes Python voraus.

## Sprache

Die Oberfläche gibt es auf Türkisch, Englisch, Deutsch und Spanisch. Beim ersten Start wird die Sprache des Browsers verwendet (Englisch, falls die Sprache nicht unterstützt wird). Mit der Sprachschaltfläche am rechten Ende des Docks (TR, EN, DE, ES) kannst du sie jederzeit ändern; deine Wahl wird in diesem Browser gemerkt. Die Ausgaben der Befehlszeile folgen der Systemsprache; mit `OZMIRA_DIL=tr|en|de|es` kannst du sie erzwingen. Diese README gibt es auch als `README.md` (Türkisch), `README.en.md` und `README.es.md`.

Hinweis: Die Befehlswörter sind türkisch und bleiben in jeder Sprache gleich: `olay` = Ereignis, `basla` = Start, `bitti` = fertig, `kisayol` = Verknüpfung, `--uygulama` = App-Fenster.

## Kostenlos und Pro

| | Kostenlos | Pro |
| --- | --- | --- |
| 3D-Szene, Ereignismeldung, Token-Zähler, lokaler Betrieb | Ja | Ja |
| KI-Systeme (Claude, ChatGPT, Gemini usw.) | Höchstens 2 | Kein Limit* |
| Agenten | Insgesamt höchstens 7 | Kein Limit* |
| Visuelle Stile | Galaxie, Kugel | Alle fünf (einschließlich Faser, Doppelhelix, Schwarzes Loch) |

\* In Pro gibt es kein kostenpflichtiges Limit. Es gibt nur eine technische Obergrenze, damit die Szene flüssig bleibt: 24 Systeme und 200 Agenten pro System.

Enthält dein Profil in der kostenlosen Version mehr als 7 Agenten (einschließlich der aus einem Ordner gelesenen), erhältst du beim Speichern eine Warnung. Wächst dein Agentenordner später, erscheinen in der Szene die ersten 7 Agenten; der Rest wird auf dem Bildschirm mit dem Hinweis „+N Agenten außerhalb des kostenlosen Limits“ vermerkt; nichts wird gelöscht.

Den Pro-Lizenzschlüssel fügst du über die Schaltfläche Konto im Dock ein (in der kostenlosen Version steht dort „Kostenlos“). Der Schlüssel wird offline auf diesem Computer geprüft; es erfolgt keine Verbindung zu einem Server. Eine Lizenz kann befristet sein; läuft sie ab, kehrt die App zu den kostenlosen Limits zurück, dein gespeichertes Profil wird nicht gelöscht, aber beim erneuten Speichern gelten die Limits.

## Agenten hinzufügen

Für jedes System gibt es zwei Wege, die du auch zusammen nutzen kannst:

- **Ordner:** Zeige auf einen Agentenordner aus `.md`-Dateien. Der Dateiname wird zum Agentennamen; beginnt die Datei mit `name:`- und `description:`-Zeilen, werden diese verwendet. Es werden nur die ersten 4 KB einer Datei gelesen und nur diese beiden Zeilen übernommen; der Inhalt wird nicht gespeichert.
- **Von Hand:** Tippe Agentennamen zeilenweise oder durch Kommas getrennt ein.

## Arbeit live anzeigen (Verbinden)

OZMIRA kann nicht von selbst wissen, wann ein Agent arbeitet; du musst es ihm mitteilen. Mit der Schaltfläche Verbinden im Dock kopierst du fertige Befehle für deine Systeme.

```
python3 ozmira.py olay <System> <Agent> basla
python3 ozmira.py olay <System> <Agent> bitti 1200 "Aufgabenbezeichnung"
```

`<System>` ist die Kurz-ID des Systems (aus dem Namen abgeleitet; das Fenster Verbinden listet die IDs auf). `1200` ist die Zahl der verbrauchten Tokens und optional. Dasselbe geht auch per HTTP:

```
curl -X POST http://localhost:8787/api/olay \
  -H "X-Token: <dein Schlüssel>" -H "Content-Type: application/json" \
  -d '{"sistem":"<System>","ajan":"<Agent>","olay":"bitti","token":1200}'
```

Das kannst du aus eigenen Skripten, Automatisierungen oder den Vor- und Nachbereitungsschritten deiner Werkzeuge aufrufen. Gesendet werden nur System, Agentenname, Ereignisart, Token-Anzahl und, wenn du möchtest, eine Aufgabenbezeichnung mit bis zu 120 Zeichen. Prompts, Antworten oder Code werden nicht gesendet.

Meldet ein Agent `basla` und innerhalb von 30 Minuten kein `bitti`, wird er in der Szene nicht mehr als aktiv angezeigt.

## Datenschutz und Sicherheit

- **Wo die Daten liegen:** im Ordner `.ozmira` in deinem Home-Verzeichnis (`profil.json`, `olaylar.jsonl`, `ayar.json`, `kullanim.json`, `.token`). Für einen anderen Ort setze die Umgebungsvariable `OZMIRA_HOME`. Solange du diesen Ordner nicht teilst, sieht niemand dein System. Der App-Ordner selbst enthält keine persönlichen Daten und kann weitergegeben werden.
- **Nach außen geschlossen:** Der Server lauscht nur auf `localhost`. Andere Computer können sich nicht verbinden.
- **Schlüssel:** Beim ersten Start wird ein zufälliger Schlüssel erzeugt (`.ozmira/.token`; unter macOS und Linux kann ihn nur dein Benutzer lesen; unter Windows funktionieren Dateirechte anders, wenn du den Computer mit anderen teilst, lege den Ordner daher an einen geschützten Ort). Alle API-Anfragen verlangen diesen Schlüssel; die Header Host und Origin werden geprüft.
- **Teilen ist standardmäßig aus.** Erlaubst du es im Konto-Fenster und liegt dieser Kopie eine Sammeladresse (`geri_bildirim_adresi.txt`) bei, wird höchstens einmal täglich Folgendes anonym gesendet: Version, Plan, eine zufällige Installations-ID und Zähler (empfangene Ereignisse, Anzahl der Profilspeicherungen). Deine Systemnamen, Agentennamen, Aufgabenbezeichnungen, Ordnerpfade, Token-Mengen und deine IP-Adresse werden nicht gesendet. Auch der Text, den du im Feedback-Fenster schreibst, wird nur mit dieser Erlaubnis gesendet; was du schreibst, wird in jedem Fall zusätzlich im Datenordner auf diesem Computer gespeichert.
- **Weitere externe Verbindung:** das Laden der Bibliothek three.js von jsdelivr. Zum Abschalten siehe „Offline-Betrieb“ unten.
- **Browserschutz:** Es wird eine strenge Content-Security-Policy verwendet; Text aus Dateien, etwa Agentennamen oder Beschreibungen, wird als reiner Text und nicht als HTML auf die Seite geschrieben.
- **Zurücksetzen:** App beenden und den Ordner `.ozmira` löschen.

## Offline-Betrieb

```
python3 ozmira.py vendor
```

Dieser Befehl lädt three.js einmal herunter und legt es in `web/vendor/three` ab (mit der MIT-Lizenz). Danach läuft OZMIRA ganz ohne Internetverbindung. Gibt es kein Internet, bricht der Befehl mit einer klaren Fehlermeldung ab und hinterlässt keine halbe Datei.

## Vom Handy ansehen (optional, mit Vorsicht)

```
python3 ozmira.py --lan
```

Der Server ist dann aus demselben Netzwerk erreichbar, und das Terminal zeigt eine Adresse für das Handy an. Ohne den Schlüssel `?k=` am Ende der Adresse öffnet sich die Seite nicht. Trotzdem erreicht jeder im selben Netzwerk den Server; nutze das nicht in öffentlichen WLANs (Café, Hotel, Schule). Starte danach ganz normal neu.

## Dateien

| Datei | Aufgabe |
| --- | --- |
| `ozmira.py` | Server und Befehlszeile (Start, `--uygulama`, olay, vendor, bilgi, kisayol) |
| `ozmira.bat`, `ozmira.command`, `ozmira.sh` | Starter für den Browser |
| `ozmira-uygulama.bat`, `.command`, `.sh` | Starter für das App-Fenster |
| `ozmira.ico`, `web/icon.png`, `web/icon.svg` | Symbole |
| `web/index.html`, `web/stil.css` | Seite und Aussehen |
| `web/app.js` | Panel, Einrichtungsassistent, Fenster Verbinden, Konto, Feedback und Sprache |
| `web/i18n.js` | Texte der vier Sprachen und Übersetzung der Server-Fehlermeldungen |
| `lisans_acik.json` | Schlüssel zur Lizenzprüfung (darf öffentlich sein; der Verkäufer legt ihn bei) |
| `web/sahne.js` | Die 3D-Szene (fünf Stile, 1 bis 24 Systeme) |

`python3 ozmira.py bilgi` gibt den Datenordner und die Anzahl der Ereignisse aus.

## Grenzen

- Die Szene ist ein aufwendiges 3D-Bild. Auf alten Grafikkarten oder im Energiesparmodus sinkt die Qualität von selbst; der Leuchteffekt kann entfallen.
- Technische Obergrenze (auch in Pro): 24 Systeme, 200 Agenten pro System. Bei mehr als 8 Systemen wird die Szene größer und die Kamera rückt weg; diese Anordnung wurde auf einer echten Grafikkarte noch nicht visuell erprobt.
- Version 0.2.0. Nur für die lokale Nutzung gedacht.
- Das App-Fenster und die Desktop-Verknüpfung wurden unter Linux mit automatischen Tests geprüft; der Verknüpfungsbefehl wurde auf einem echten Windows- oder macOS-Computer noch nicht ausprobiert. Funktioniert er nicht, nutze `ozmira-uygulama.bat` / `.command`.
- Die Übersetzungen der vier Sprachen wurden mit KI erstellt; ein Muttersprachler für Englisch, Deutsch oder Spanisch sollte sie prüfen.
- Die Szene wurde auf einem einzelnen Windows-Computer mit echter Grafikkarte erprobt. Erscheint sie auf anderen Grafikkarten beim ersten Start nicht, prüfe, ob WebGL im Browser aktiviert ist.
