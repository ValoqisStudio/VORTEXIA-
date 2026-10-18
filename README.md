# VORTEXIA – Valoqis Studio

Open-World-Spiel im Stil von Los Santos, komplett in **einer HTML-Datei** – läuft im Browser, auch offline auf dem Handy.

| Datei | Was ist das? |
|---|---|
| `index.html` | Normale Version – wählt die Grafikstufe automatisch passend zum Gerät |
| `Vortexia_ULTRA.html` | Ultra-Version – startet immer mit maximaler Grafik (Schatten, Lichtstrahlen, Spiegelungen) |
| `tools/build_ultra.py` | Erzeugt die Ultra-Version aus `index.html` (`python3 tools/build_ultra.py`) |

## Aufs Handy holen

1. Download-Link im Handy-Browser öffnen:
   - Normal: https://github.com/ValoqisStudio/VORTEXIA-/raw/main/index.html
   - Ultra: https://github.com/ValoqisStudio/VORTEXIA-/raw/main/Vortexia_ULTRA.html
2. Zeigt der Browser nur Text an: Menü (⋮) → **Herunterladen** bzw. **Teilen → In Dateien sichern**.
3. Die gespeicherte Datei in der Dateien-App antippen und mit **Chrome** (Android) bzw. **Safari** (iPhone) öffnen.

Die Dateien sind je ca. 20 MB groß, weil 3D-Engine, Modelle und Stimmen eingebettet sind – dafür braucht das Spiel kein Internet.

## Direkt im Browser spielen

- Normal: https://valoqisstudio.github.io/VORTEXIA-/
- Ultra: https://valoqisstudio.github.io/VORTEXIA-/Vortexia_ULTRA.html

Die Webseite kommt aus dem Zweig `gh-pages` – Änderungen dort hin pushen (`git push origin main:gh-pages`).

## In die eigene Webseite einbauen

Es reicht diese eine Zeile in deiner Webseite:

```html
<iframe src="https://valoqisstudio.github.io/VORTEXIA-/" style="width:100%;height:100vh;border:0" allow="fullscreen; autoplay; gamepad" allowfullscreen></iframe>
```

Für die Ultra-Version `…/VORTEXIA-/Vortexia_ULTRA.html` als `src` eintragen.
Alternativ `index.html` direkt bei deinem Webhoster hochladen und verlinken.

## Flüssig spielen

Ruckelt das Spiel, senkt es automatisch erst die Auflösung und dann die Grafikstufe
(Ultra → Hoch → Mittel → Niedrig) und merkt sich die passende Stufe für den nächsten Start.
Im Menü unter Einstellungen kannst du jederzeit selbst eine Stufe wählen.

## Steuerung

- **PC:** WASD laufen, Maus umsehen, Shift rennen, E einsteigen, Q Waffe, Tab Figurenwechsel (Vincent, Darnell, Rocco)
- **Handy:** Joystick links, Knöpfe rechts, „⇄ Figur" oben rechts
