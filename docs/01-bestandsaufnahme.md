# Bestandsaufnahme und Architekturentscheidung

Stand: 13.11.2026. Am Gerät und am Server wurde bisher nichts verändert.

## Hardware

| Gerät | Befund |
|---|---|
| Amazon Echo Dot, Model RS03QR, FCC ID 2AHSE-2045 | **Echo Dot 2. Generation (2016)**, Codename `biscuit`, SoC MediaTek MT8163V, Micro-USB (Strom und Daten) |
| HP t630 (8 GB RAM, Proxmox, Portainer) | geeignet für Home Assistant OS (VM), Whisper (kleine Modelle) und Piper; nicht sinnvoll für Ollama (keine GPU) |

## Rahmenbedingungen (Festlegungen)

- Der Assistent muss **Deutsch** verstehen und sprechen.
- Sprachaufnahmen dürfen an einen Cloud-Anbieter gehen.
- **Keine zusätzliche Hardware**, **keine zusätzlichen Abonnements**.
- Installationsrechner: Windows-PC bevorzugt, Linux vorhanden.

## Geprüfte Firmware-Optionen für den Dot

| Projekt | Status RS03QR | Bemerkung |
|---|---|---|
| **TECHO5 Dot** | im Alltagsbetrieb (3 Geräte beim Autor) | Windows-Installer, Sicherung der Boot-Partitionen, Rückweg dokumentiert, benötigt Home Assistant |
| LibreEcho 0.14.5 | experimentell | ohne Home Assistant nutzbar, Spracherkennung/-ausgabe auf dem Gerät nur Englisch, Linux empfohlen |
| EchoMuse | unterstützt | benötigt eigenen Controller-Rechner |

## Entscheidung

**Variante B: Home Assistant auf dem t630 + TECHO5 Dot.**

```
Echo Dot 2 (TECHO5, Wake-Word lokal) ── ESPHome ──► Home Assistant (HAOS-VM, t630)
                                                      ├─ STT: Whisper lokal (Deutsch)
                                                      ├─ Befehle zuerst lokal verarbeiten
                                                      ├─ Conversation Agent: Mistral-API (kostenloser Experiment-Tarif)
                                                      └─ TTS: Piper lokal (deutsche Stimme)
```

Datenfluss nach außen: nur der **Text** freier Fragen geht an Mistral. Training im
Mistral-Admin-Konto unter „Privacy“ abschalten.

Hinweis: Das Le-Chat-Pro-Abo umfasst keine API-Nutzung; Claude-Max ebenso nicht.

## Unlock- und Installationsweg TECHO5 Dot (Kurzfassung)

1. Unlock mit amonet-biscuit (`fastbrick.bat`, Action-Taste beim Einstecken, kein Öffnen) → TWRP.
2. Fire OS 6574.1 (`update-kindle-biscuit_puffin-NS6574_user_7623_0013121734532.bin`) zweimal installieren.
3. `python tools/install-dot.py --dry-run`, dann `python tools/install-dot.py`.
4. Gerät in Home Assistant über ESPHome hinzufügen.

Rückweg: Ordner `backups/<Seriennummer>/` aufbewahren.

## Offene Punkte

- XDA-Thread lesen: Funktioniert der Unlock mit jeder vorhandenen Fire-OS-Version?
- Offene TECHO5-Issues zum Dot 2 prüfen („zu leise“, Hänger nach dem Flashen).
- Ressourcen des t630 erfassen (Proxmox-Version, freier RAM/Speicher, laufende Dienste).
- Mistral-Integration für Home Assistant auswählen (Community, HACS).

## Quellen

- TECHO5: https://github.com/HuskerMinion/techo5 · https://github.com/HuskerMinion/techo5-dot
- TECHO5 Getting Started: https://github.com/HuskerMinion/techo5/blob/main/docs/getting-started.md
- amonet-biscuit (XDA): https://xdaforums.com/t/unlock-root-twrp-unbrick-amazon-echo-dot-2nd-gen-2016-biscuit.4761416/
- LibreEcho: https://libreecho.org/ · https://github.com/aslater3/LibreEcho
- EchoMuse: https://github.com/wilbowes/EchoMuse
- iFixit Echo Dot 2nd Gen: https://ts.ifixit.com/Device/Amazon_Echo_Dot_2nd_Generation
- FCC 2AHSE-2045: https://fcc.report/FCC-ID/2AHSE-2045/3218293.pdf
- Mistral Experiment-Tarif: https://help.mistral.ai/en/articles/450104-how-can-i-try-the-api-for-free-with-the-experiment-plan
- Mistral Training-Opt-out: https://help.mistral.ai/en/articles/455207-can-i-opt-out-of-my-input-or-output-data-being-used-for-training
- Anthropic Abo vs. API: https://support.anthropic.com/en/articles/9876003
