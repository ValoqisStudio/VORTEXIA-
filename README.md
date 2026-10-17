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

## Direkt im Browser spielen (GitHub Pages)

Einmalig im Repository: **Settings → Pages → Source: Deploy from a branch → Branch `main` / `/ (root)` → Save**.
Danach läuft das Spiel unter https://valoqisstudio.github.io/VORTEXIA-/ (Ultra: `…/VORTEXIA-/Vortexia_ULTRA.html`).

## Steuerung

- **PC:** WASD laufen, Maus umsehen, Shift rennen, E einsteigen, Q Waffe, Tab Figurenwechsel (Vincent, Darnell, Rocco)
- **Handy:** Joystick links, Knöpfe rechts, „⇄ Figur" oben rechts
