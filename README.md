# AssettoCorsaDCServer

Fertiges Setup fuer einen **Assetto Corsa Dedicated Server unter Windows**: Installation per Skript, Firewall-Regeln, Start-Datei und Beispielkonfiguration.

> Hinweis: Das Setup ist noch nicht auf einem echten Windows-Server getestet.

## Inhalt

| Datei | Zweck |
|---|---|
| `install.ps1` | Laedt SteamCMD und den Dedicated Server (Steam-App 302550, anonym) und kopiert `cfg\` in den Server |
| `start.bat` | Startet `server\acServer.exe` |
| `firewall.ps1` | Oeffnet die Ports in der Windows-Firewall (als Administrator) |
| `install-assettoserver.ps1` | Optional: [AssettoServer](https://assettoserver.org) fuer Content Manager / CSP |
| `cfg\server_cfg.ini` | Serverkonfiguration (Name, Passwoerter, Strecke, Autos, Sessions, Wetter) |
| `cfg\entry_list.ini` | Fahrer-Slots (10 Stueck, Abarth 500 und Porsche 911 GT3 Cup im Wechsel) |

## Voraussetzungen

- Windows 10/11 oder Windows Server, PowerShell 5.1 oder neuer
- Internetzugang fuer den Download (ca. 1 GB)
- **Kein** Steam-Konto und **kein** installiertes Assetto Corsa noetig. Eigene oder DLC-Strecken und -Autos musst du aber aus einer AC-Installation kopieren (siehe "Strecken und Autos")
- Spieler brauchen Assetto Corsa

## Installation

1. Repo klonen oder als ZIP herunterladen und entpacken, z. B. nach `C:\AssettoCorsaDCServer`.
2. **Konfiguration anpassen** (vor der Installation, sonst danach `install.ps1` erneut ausfuehren): in `cfg\server_cfg.ini`
   - `NAME` - Servername
   - `PASSWORD` - Passwort zum Beitreten (leer = oeffentlich)
   - `ADMIN_PASSWORD` - Admin-Passwort
   - `TRACK`, `CONFIG_TRACK`, `CARS`, `MAX_CLIENTS`
3. PowerShell im Ordner oeffnen und ausfuehren:
   ```powershell
   PowerShell -ExecutionPolicy Bypass -File .\install.ps1
   ```
   Das legt `steamcmd\` und `server\` an und kopiert die Konfiguration nach `server\cfg\`.
4. Firewall-Ports oeffnen (PowerShell **als Administrator**):
   ```powershell
   PowerShell -ExecutionPolicy Bypass -File .\firewall.ps1
   ```

## Starten

Doppelklick auf `start.bat`. Im Fenster erscheint das Serverlog. Zum Beenden das Fenster schliessen oder `Strg+C`.

Wenn der Server sofort stoppt, steht der Grund im Log, meist fehlt eine Strecke oder ein Auto in `server\content\...` oder eine Konfigurationszeile ist falsch.

## Nutzen

### Verbinden
- **Im selben Netzwerk:** Im Content Manager unter *Drive > Online > LAN* erscheint der Server, oder direkt per `IP-des-PCs:9600`.
- **Ueber das Internet:** Im Router **UDP 9600**, **TCP 9600** und **TCP 8081** auf den Server-PC weiterleiten. Dann per `oeffentliche-IP:9600` verbinden. Mit `REGISTER_TO_LOBBY=1` erscheint der Server in der oeffentlichen Liste.
- Eine Weboberflaeche mit Serverinfo liegt unter `http://IP:8081/INFO`.

### Admin im Spiel
Im Chat `/admin <ADMIN_PASSWORD>`, danach z. B.:
- `/next_session` - naechste Session starten
- `/restart_session` - Session neu starten
- `/kick <Name>` oder `/ban <Name>`
- `/help` - alle Befehle

### Strecken und Autos
- `TRACK` ist der Ordnername unter `server\content\tracks`. Hat die Strecke mehrere Layouts, steht der Layoutname in `CONFIG_TRACK`.
- `CARS` ist eine Liste von Ordnernamen unter `server\content\cars`, getrennt durch `;`.
- In `cfg\entry_list.ini` gibt es einen Block `[CAR_n]` pro Slot. Es muessen mindestens `MAX_CLIENTS` Bloecke sein, und `MODEL` muss in `CARS` vorkommen. Optional lassen sich `SKIN`, `DRIVERNAME` oder `GUID` fest setzen.
- Eigene Inhalte kopierst du aus `...\steamapps\common\assettocorsa\content\tracks` bzw. `\cars` (oder aus Mods) nach `server\content\...`.
- Nach Aenderungen an `cfg\` entweder `install.ps1` erneut ausfuehren oder die Dateien direkt in `server\cfg\` bearbeiten, dann den Server neu starten.

### Sessions
In `server_cfg.ini` die Abschnitte `[PRACTICE]`, `[QUALIFY]`, `[RACE]` anpassen (Dauer in Minuten, `LAPS` fuer die Rundenzahl). Das Wetter steht in `[WEATHER_0]`.

### Optional: AssettoServer (Content Manager, CSP)
```powershell
PowerShell -ExecutionPolicy Bypass -File .\install-assettoserver.ps1
```
Das Skript laedt das neueste AssettoServer-Release nach `assettoserver\` und nutzt dieselbe Konfiguration und denselben Content. Starten mit `assettoserver\AssettoServer.exe`. Weitere Optionen (Wetter, Plugins) in der [AssettoServer-Doku](https://assettoserver.org).

## Updates
`install.ps1` erneut ausfuehren, es aktualisiert den Server per SteamCMD (`validate`) und kopiert die Konfiguration erneut.
**Achtung:** Dabei werden Aenderungen direkt in `server\cfg\` ueberschrieben, pflege deshalb die Konfiguration in `cfg\`.

## Fehlersuche

| Problem | Ursache / Loesung |
|---|---|
| Server startet nicht, Fenster schliesst sich | `start.bat` zeigt am Ende `pause`. Meldung lesen. Meist fehlt Strecke/Auto in `server\content` |
| Nicht im LAN sichtbar | Firewall-Regeln (`firewall.ps1`) pruefen, Netzwerkprofil "Privat" verwenden |
| Aus dem Internet nicht erreichbar | Portfreigabe im Router, bei CGNAT/DS-Lite keine direkte Erreichbarkeit moeglich (VPN oder Tunnel noetig) |
| Spieler "Auto nicht gefunden" | Spieler besitzt Strecke/Auto nicht, bei Mods ueber den Content Manager nachladen |
| Skript meldet "Ausfuehrung deaktiviert" | Mit `-ExecutionPolicy Bypass` starten wie oben gezeigt |
