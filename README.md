# ESP32-C6 Touch LCD 1.47 mit PlatformIO

Dieses Projekt portiert das Waveshare-Arduino-IDE-Beispiel
04_lvgl_arduino_v8 in ein eigenständiges PlatformIO-Projekt.

## Wichtige PlatformIO-Konfiguration

```ini
[env:esp32-c6-touch-lcd-1-47]
platform = espressif32
board = esp32-c6-devkitc-1
framework = arduino
monitor_speed = 115200
upload_speed = 921600
```

### Bedeutung der `build_flags`

In der `platformio.ini` stehen zusätzlich folgende Compiler-Optionen:

```ini
build_flags =
    -DLV_CONF_INCLUDE_SIMPLE
    -I${PROJECT_DIR}/include
```

Diese beiden Optionen gehören zusammen und sorgen dafür, dass LVGL die
projektspezifische Konfiguration findet.

`-DLV_CONF_INCLUDE_SIMPLE` definiert beim Kompilieren das Makro
`LV_CONF_INCLUDE_SIMPLE`. LVGL 8 verwendet bei gesetztem Makro einen einfachen
Include für seine Konfigurationsdatei:

```cpp
#include "lv_conf.h"
```

Ohne dieses Makro erwartet LVGL die Konfigurationsdatei je nach LVGL-Struktur
an einer anderen, relativen Stelle beziehungsweise unter einem LVGL-bezogenen
Pfad. Das wäre hier unpraktisch, weil die Konfiguration bewusst außerhalb der
LVGL-Bibliothek liegt.

`-I${PROJECT_DIR}/include` teilt dem Compiler mit, dass er zusätzlich im
Projektordner `include/` nach Header-Dateien suchen soll. `${PROJECT_DIR}` ist
eine PlatformIO-Variable und steht für den absoluten Pfad dieses Projekts.

Dadurch wird bei `#include "lv_conf.h"` die Datei `include/lv_conf.h` gefunden.
Diese Datei enthält die Einstellungen für dieses Display-Projekt, zum Beispiel
Farbtiefe, aktivierte Widgets, Schriftarten und aktivierte LVGL-Demos.

Der Ablauf ist also:

```text
LVGL wird eingebunden
        |
        v
LV_CONF_INCLUDE_SIMPLE ist definiert
        |
        v
LVGL sucht nach "lv_conf.h"
        |
        v
PlatformIO durchsucht zusätzlich ${PROJECT_DIR}/include
        |
        v
include/lv_conf.h wird verwendet
```

Wichtig: `-DLV_CONF_INCLUDE_SIMPLE` kopiert keine Datei und aktiviert auch
keine LVGL-Funktionen. Es ändert nur, wie LVGL seine Konfiguration einbindet.
Der Pfad mit `-I` macht diese Datei für den Compiler auffindbar. Fehlt eines
der beiden Flags, kann der Build je nach LVGL-Version mit einer fehlenden
`lv_conf.h` abbrechen oder LVGL mit falschen Standardwerten kompilieren.

Die Bibliotheken liegen jetzt direkt unter `lib/`. Dadurch ist das Projekt
unabhängig von der ursprünglichen Ordnerstruktur. LVGL 8 ist wichtig, weil das
Beispiel die LVGL-8-API verwendet.

## Befehle

Im Projektordner:

```bash
cd examples/09_pio_lvgl_test
pio run
pio run -t upload
pio device monitor -b 115200
```

## Links
https://lvgl.io/docs/open/integration/frameworks/platformio
https://docs.waveshare.com/ESP32-C6-Touch-LCD-1.47/Resources-And-Documents
