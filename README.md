# Repair-corrupt-Home-Assistant-Database
Instruction for repairing a corrupt Home Assistant database which impedes Home Assistant from starting

🛠️ Benötigte Hilfsmittel
Ein Windows-PC/Laptop mit installierter PowerShell.

Das offizielle SQLite-Kommandozeilenwerkzeug (installierbar in der PowerShell via WinGet: winget install SQLite.SQLite).

Ein schneller USB-Stick (formatiert in NTFS oder exFAT – wichtig: kein FAT32 wegen der 4-GB-Dateigrößengrenze!).

Das Home Assistant Add-on Samba share (für den späteren Dateiaustausch).

📋 Schritt-für-Schritt-Anleitung
Teil 1: Vorbereitung auf dem USB-Stick
Kopiere die defekte Datenbankdatei (home-assistant_v2.db) von Home Assistant auf den USB-Stick.

Benenne die Datei auf dem Stick um in: home-assistant_v2.db.broken

Öffne die PowerShell auf dem PC und wechsle auf das Laufwerk des USB-Sticks (Beispiel bei Laufwerk H:):

PowerShell
Set-Location "H:\"
Teil 2: Datenrettung in eine Textdatei (Export)
Um Windows-Codierungsfehler (wie das ungewollte UTF-16-Format von PowerShell) zu vermeiden, lassen wir SQLite die SQL-Textdatei direkt selbst im sauberen UTF-8-Format schreiben:

Führe folgenden Befehl aus, um alle lesbaren Daten in eine .sql-Textdatei zu exportieren:

PowerShell
sqlite3 home-assistant_v2.db.broken ".output repair.sql" ".recover" ".exit"
Warte, bis die Eingabezeile PS H:\> wieder erscheint. (Dauert je nach Datenbankgröße einige Minuten).

Teil 3: Datenbank neu aufbauen (Import mit "Brechstange")
Damit der Import bei Fehlern oder unlesbaren Datenblöcken nicht mittendrin abbricht, wird der .bail off-Modus erzwungen. Dadurch ignoriert SQLite Fehler stur und rettet das absolute Maximum der Daten:

Starte den Import mit diesem kombinierten Befehl:

PowerShell
sqlite3 home-assistant_v2.db.repaired ".bail off" ".read repair.sql"
Wichtig beim Warten: Die Datei wächst anfangs schnell (Tabellen-Upload) und stagniert am Ende bei der Index-Berechnung. Solange im Windows-Taskmanager der Prozess sqlite3.exe aktiv CPU-Leistung oder Datenträger-Aktivität zeigt, läuft der Import!

Warte, bis der Prozess fertig ist und die PowerShell wieder die normale Eingabezeile PS H:\> freigibt.

Teil 4: Austausch in Home Assistant
Benenne die fertig reparierte Datei auf dem USB-Stick um in: home-assistant_v2.db (Hinweis: Es ist völlig normal, wenn sie durch das Entfernen von Datenfragmenten/Müll deutlich kleiner ist als das Original!).

Installiere und starte in Home Assistant das Add-on Samba share (Passwortvergabe in der Add-on-Konfiguration nicht vergessen!).

Gehe in Home Assistant auf Entwicklerwerkzeuge → Dienste/Aktionen, suche den Dienst recorder.stop und führe ihn aus. (Das stoppt alle Schreibzugriffe auf die Datenbank, lässt das System und Samba aber aktiv).

Öffne den Windows-Explorer auf dem PC und greife auf den HA-Ordner zu: \\homeassistant\config (oder über die IP-Adresse des HA-Systems).

Benenne die dortige alte, defekte Datenbank zur Sicherheit in home-assistant_v2.db.old um.

Kopiere die frisch reparierte home-assistant_v2.db vom USB-Stick per Drag-and-Drop in den Netzwerkordner.

Gehe in Home Assistant auf Einstellungen → System, klicke oben rechts auf das Ein/Aus-Symbol und wähle Home Assistant neu starten.

Sobald das System hochgefahren ist, lädt der Recorder die frisch optimierte Datenbank und alle Langzeitstatistiken stehen wieder zur Verfügung.
