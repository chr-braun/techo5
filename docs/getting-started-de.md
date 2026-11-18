# TECHO5 Dot – Einrichtung auf Deutsch

Echo Dot 2. Gen (Codename `biscuit`) als lokalen Home-Assistant-Sprachsatelliten einrichten –
Schritt für Schritt, von einem unberührten Gerät bis zum fertigen Assistenten.

> **Lies das zuerst.** Der Unlock nutzt Bootloader-Exploits. Das Gerät kann dabei dauerhaft
> beschädigt werden, die Garantie erlischt, und alle Daten werden gelöscht.
> Mach das nur mit einem Gerät, dessen Verlust du verkraften kannst, lass es während des gesamten
> Vorgangs am Strom und trenne es nie, während etwas geschrieben wird.
> Diese Projekte sind Hobby-Software, keine Produkte.

---

## Was du brauchst

| Was | Wo |
|-----|----|
| **amonet-biscuit-v2.0.1.zip** | Anhang im [XDA-Thread](https://xdaforums.com/t/unlock-root-twrp-unbrick-amazon-echo-dot-2nd-gen-2016-biscuit.4761416/) (kostenloses XDA-Konto nötig) |
| **Fire-OS-Datei** `update-kindle-biscuit_puffin-NS6574_user_7623_0013121734532.bin` | Gleicher XDA-Thread |
| `boot-root.zip` | Liegt im amonet-ZIP drin (Schritt 3) |
| **TECHO5 Dot Installer** | `git clone https://github.com/HuskerMinion/techo5-dot` (Schritt 6) |
| **Micro-USB-Datenkabel** | Ladekabel reicht nicht – es muss Daten übertragen |
| **Home Assistant** | Mit ESPHome-Integration (ist eingebaut) |
| **Computer** mit Windows, Linux oder macOS | Einrichtung siehe unten |

> **Wichtig:** Verwende **amonet-biscuit-v2.0.1** aus dem ZIP-Anhang des XDA-Threads – nicht den
> `mt8163-biscuit`-Branch auf GitHub (der ist noch v2.0.0 und enthält kein macOS-`fastboot`).

---

## Computer einrichten (einmalig)

Die Installer sind Python-3-Skripte, die auf Windows, Linux und macOS gleich funktionieren.
Du brauchst Python 3, die Android Platform Tools (`adb` und `fastboot`) und `git`:

**macOS:**
```
xcode-select --install
brew install android-platform-tools
```
Prüfen: `python3 --version` und `adb version` müssen eine Versionsnummer ausgeben.

> **macOS-Hinweis:** Der Dot-Unlock läuft vollständig auf dem Mac – kein Linux-Live-USB nötig.
> `fastbrick.sh` arbeitet im Fastboot-Modus des Dots (nicht im BootROM), und `amonet-biscuit-v2.0.1`
> enthält ein universelles macOS-`fastboot`-Binary (Apple Silicon und Intel). Das ist ein
> Unterschied zum Show und zum Spot, die für ihren Unlock Linux brauchen.

**Linux (Ubuntu/Debian):**
```
sudo apt install python3 git adb fastboot
sudo usermod -aG dialout $USER   # dann neu anmelden
sudo systemctl stop ModemManager # falls installiert: während der Installation stoppen
```

**Windows:**
- [Python 3](https://www.python.org/downloads/) installieren (Haken bei „Add python.exe to PATH")
- [Git for Windows](https://git-scm.com/download/win)
- [Android Platform Tools](https://developer.android.com/tools/releases/platform-tools) (entpacken, Ordner zu PATH hinzufügen)
- Überall `python` statt `python3` tippen

---

## Schritt 1 – Unlock mit amonet-biscuit

Folge dem XDA-Thread
[[UNLOCK][ROOT][TWRP][UNBRICK] Echo Dot 2nd Gen / 2016 (biscuit)](https://xdaforums.com/t/unlock-root-twrp-unbrick-amazon-echo-dot-2nd-gen-2016-biscuit.4761416/)
(Quelle: [R0rt1z2/amonet, Branch mt8163-biscuit](https://github.com/R0rt1z2/amonet/tree/mt8163-biscuit)).

1. Das einzige USB-Ein-/Ausgang des Dots ist der Micro-USB-Anschluss – der gleiche, den das Ladekabel
   benutzt. Steck ihn in den Computer, nicht in die Steckdose.
2. **Action-Taste gedrückt halten** und dabei das Micro-USB-Kabel einstecken.
   Das Ringlicht leuchtet **grün** – das ist Amazons Fastboot-Modus.
3. `fastbrick.bat` (Windows) bzw. `./fastbrick.sh` (Linux/macOS) ausführen,
   das gefundene Gerät prüfen und `YES` eingeben.
4. Der Vorgang endet mit `Exploit most likely successful!` und startet in TWRP
   – das Ringlicht leuchtet **weiß**.

**Zurück in TWRP** (wenn nötig): Micro-USB-Kabel abstecken, **Volume-Up-Taste** gedrückt halten,
Kabel wieder einstecken, weißes Licht abwarten.

✅ **Prüfen:** `adb devices` zeigt den Dot als `recovery`.

---

## Schritt 2 – Fire OS 6574.1 in beide Slots flashen

TECHO5 Dots Bluetooth-Kernel ist für genau dieses Build kompiliert (NS6574, Build 7623), und der
WLAN-Treiber wird aus der System-Partition von Fire OS geladen. Andere Builds passen nicht.

Datei: `update-kindle-biscuit_puffin-NS6574_user_7623_0013121734532.bin`

```bash
adb shell twrp wipe cache
adb shell twrp wipe data
adb push update-kindle-biscuit_puffin-NS6574_user_7623_0013121734532.bin /sdcard/update.zip
adb shell twrp install /sdcard/update.zip
adb reboot recovery
adb shell twrp install /sdcard/update.zip
```

Das zweite `install` spielt Fire OS in den zweiten A/B-Slot. Eine Warnung
`no selinux policy bundled` im Install-Log bedeutet, dass das Datei-Build nicht passt –
Vorgang stoppen und die richtige Datei besorgen.

✅ **Prüfen:** Beide Installs ohne Fehler abgeschlossen.

---

## Schritt 3 – Root-Zugang einrichten

Noch in TWRP:

```bash
adb push boot-root.zip /sdcard/
adb shell twrp install /sdcard/boot-root.zip
```

---

## Schritt 4 – ADB-Key hinterlegen

Der Dot hat keinen Bildschirm, um eine ADB-Autorisierungsabfrage anzuzeigen.
Den Schlüssel jetzt dauerhaft hinterlegen:

```bash
# macOS/Linux:
adb push ~/.android/adbkey.pub /sdcard/adbkey.pub

# Windows:
adb push %USERPROFILE%\.android\adbkey.pub /sdcard/adbkey.pub
```

```bash
adb shell "mkdir -p /data/misc/adb && cp /sdcard/adbkey.pub /data/misc/adb/adb_keys && chown 1000:2000 /data/misc/adb/adb_keys && chmod 640 /data/misc/adb/adb_keys"
adb shell restorecon -v /data/misc/adb/adb_keys
adb reboot
```

> Falls `~/.android/adbkey.pub` noch nicht existiert: einmal `adb devices` ausführen, dann nochmal.

---

## Schritt 5 – WLAN einrichten (optional)

Nach dem Neustart startet Fire OS. Den Alexa-App-Schritt für WLAN abschließen;
den Rest der Alexa-Einrichtung kann man überspringen. Der Installer liest das gespeicherte
Netzwerk automatisch aus. Alternativ: Netzwerk beim Installer mit `--wifi "MeinNetzwerk"` angeben
(er fragt dann nach dem Passwort).

✅ **Prüfen:** `adb devices` zeigt `device`, und `adb shell id` gibt `uid=0` aus.

---

## Schritt 6 – TECHO5 Dot installieren

```bash
git clone https://github.com/HuskerMinion/techo5-dot
cd techo5-dot
python3 tools/install-dot.py --dry-run   # Checks + Download, schreibt nichts
python3 tools/install-dot.py
```

Der `--dry-run` sichert alle Boot-Partitionen in `backups/<serial>/` und lädt das aktuelle Release
herunter – **diesen Ordner aufbewahren, er ist der Weg zurück**.
Der zweite Lauf installiert TECHO5, wartet auf den ersten erfolgreichen Boot und meldet „healthy".

> Der Installer funktioniert auch direkt aus TWRP nach Schritt 2 (ohne Schritte 3–5):
> `python3 tools/install-dot.py --lineage-zip <...> --wifi "MeinNetzwerk"` –
> dann übernimmt er die WLAN-Einrichtung selbst.

✅ **Prüfen:** Der Installer endet mit „Dot healthy" und einem erreichbaren Home-Assistant-Port.

---

## Schritt 7 – Dot zu Home Assistant hinzufügen

Home Assistant erkennt das Gerät automatisch als ESPHome-Gerät (Einstellungen → Geräte &
Dienste → ESPHome). Den Verschlüsselungsschlüssel aus `backups/<serial>/home-assistant.key`
(oder der Installer-Ausgabe) beim Hinzufügen eingeben.

Danach auf der Setup-Seite (`http://<Dot-IP>:8181/setup`, erreichbar solange der **Setup page**-Schalter in HA an ist):

1. **Network → Home Assistant access** – HA-Adresse und einen Long-Lived-Token eintragen,
   damit der Dot selbst auf HA zugreifen kann (nötig für Sprachassistent, Fotos, Wetter).
2. **Sound → Wake word** – gewünschtes Wake-Word auswählen.
3. **Sound → Voice** – Assist-Pipeline zuweisen.

---

## Schritt 8 – Sprachassistent einrichten (Assist-Pipeline)

In Home Assistant unter **Einstellungen → Sprachassistenten** eine Pipeline anlegen:

| Feld | Empfehlung |
|------|-----------|
| Sprachaktivierungserkennung | Wyoming (`openWakeWord` oder `microWakeWord`) oder direkt auf dem Dot |
| Sprache zu Text | Whisper (`faster-whisper small-int8` lokal, oder Cloud) |
| Konversations-Agent | Home Assistant (lokal) oder ein LLM |
| Text zu Sprache | Piper – **`tts_language: de_DE`** (nicht `de`!) |

> ⚠️ **Häufiger Fehler:** `tts_language: de` funktioniert nicht. Piper meldet Sprachen mit
> Unterstrich (`de_DE`). Mit dem Kurzformat bricht jede Antwort mit `tts-not-supported` ab –
> auf dem Bildschirm sieht man den Text, aber es spricht nichts.

---

## Mikrofon-Einstellungen (Dot 2 „biscuit")

Im HA-Geräteeintrag unter **Konfiguration → Mikrofon**:

| Einstellung | Wert | Warum |
|-------------|------|-------|
| Mic-Mixing | **All microphones** | 7-Mikro-Durchschnitt – **nicht** Center mic |
| Gain | **20 dB** | Herstellerstandard; höherer Wert bringt laut Maintainer nichts (SNR-Limit) |
| Microphone leveling | **An** | Ziel: −23 dBFS RMS, Peaks unter −3 dBFS |
| Noise reduction | Optional | Bei abgehackten Wortenden (z. B. „Milch" wird als „Mich" erkannt): **aus** |

> Die Einstellung „Center mic" war in älteren Anleitungen als Standard angegeben – das war falsch.
> Sie gibt den zentralen Lautsprecher des Dots wieder, der nur einen Bruchteil des Schalls aus der
> Raumrichtung einfängt. Der 7-Mikro-Durchschnitt (`All microphones`) funktioniert deutlich besser.
> Korrigiert in [techo5-dot#48](https://github.com/HuskerMinion/techo5-dot/pull/48).

---

## Bekannte Probleme und Lösungen

### Wake-Word-Auswahl verschwindet nach HA-Neustart

Nach einem Neustart von Home Assistant stehen die Wake-Word-Einstellungen manchmal auf
`no_wake_word`. Das Gerät hat sie noch, aber HA ordnet sie neu zu und fällt auf den ersten
Assistenten zurück. Lösung: eine Automation, die nach jedem HA-Start die Auswahl wiederherstellt.

```yaml
alias: Dot Wake-Word wiederherstellen
trigger:
  - platform: homeassistant
    event: start
action:
  - delay: 00:00:30
  - service: select.select_option
    target:
      entity_id: select.matilda_dot_wake_word
    data:
      option: hey_jarvis
```

### Wyoming-Integration meldet „setup_error" nach HA-Neustart

Die Kinder-Pipeline zeigt „No speech-to-text provider" – der Wyoming-Eintrag ist im Fehlerzustand.
Lösung: gleiche Automation wie oben, zusätzlich den Wyoming-Eintrag neu laden:

```yaml
  - service: homeassistant.reload_config_entry
    target:
      entity_id: stt.faster_whisper
```

### Jinja2-Templates ergeben „None" statt leer

Bei Template-Sensoren, die auf Dot-Entitäten zugreifen: immer `| default('')` anhängen,
sonst erscheint der Text „None" in Antworten.

### amonet: `restore_gpt` nicht auskommentieren

Beim Update von amonet 1.x auf 2.x: die Funktion `restore_gpt` im `update-binary`-Skript
**nicht auskommentieren**. Sie schreibt die GPT-Partitionstabelle aus `bin/gpt-rook.bin` –
ohne sie ist die Tabelle inkonsistent und der Installer kann `mmcblk0p13` nicht finden.
Wenn amonet 2.0 trotzdem durchgelaufen ist: prüfen mit
`adb shell cat /proc/partitions` ob `mmcblk0p13` aufgelistet ist. Falls nicht: amonet mit dem
unmodifizierten `update-binary` nochmal ausführen.
*(Gilt für den Echo Spot / rook; beim Dot 2 / biscuit ist der Unlock anders aufgebaut.)*

---

## Dot zurück zu TWRP oder Fire OS

TECHO5 Dot lebt in der Recovery-Partition (wo vorher TWRP war) und ersetzt bei jedem Start
den Recovery-Modus. Deshalb startet **Volume Up beim Einschalten TECHO5**, nicht TWRP.

**Während TECHO5 läuft** (SSH in HA aktiviert):
```bash
to-twrp --yes    # startet TWRP; von dort: adb shell sh /cache/techo5/back-to-linux.sh
```

**Aus amonets Fastboot-Modus** (amonet-Unlock-Thread):
```bash
fastboot flash recovery backups/<serial>/recovery.img
fastboot reboot recovery
```

---

## Wenn etwas schiefgeht

- **Vor der TECHO5-Installation:** Jeder XDA-Thread hat einen Unbrick-Abschnitt. Niemals
  eigenmächtig Bootloader-Images flashen – so entstehen dauerhaft beschädigte Geräte.
- **Nach der Installation:** TECHO5 Dot fällt automatisch auf den vorherigen Slot zurück,
  dann auf eine Rescue-Umgebung mit USB-Serial-Konsole, wenn ein Boot nicht als gesund meldet.
  `backups/<serial>/` enthält alles, was nötig ist.
- **Installer stoppt bei einer Prüfsumme:** Der Download wird einmal wiederholt; schlägt er
  wieder fehl, zeigt die Fehlermeldung die empfangenen Bytes – weniger als erwartet heißt
  Verbindungsabbruch, gleich viele Bytes könnten auf Antivirus oder einen Proxy hindeuten.
- Fragen ins GitHub-Issues-Tracker des Projekts, mit Gerät, Schritt und vollständiger
  Ausgabe des Installers.

---

## Danksagung

Die Unlocks stammen von [R0rt1z2](https://github.com/R0rt1z2) und k4y0z (amonet, kaeru,
Echo-TWRP-Builds). TECHO5s Daemon baut auf [EchoLocal](https://github.com/ygelfand/echolocal)
von Yuri Gelfand auf. Die Dot-Schritte gehen auf
[@proffalken](https://github.com/proffalken)'s
[Anleitung](https://gist.github.com/proffalken/377ae50146affe1886dddaaacb87926b) zurück,
die den genauen Fire-OS-Build gefunden hat, der SELinux-Bootloops vermeidet.
