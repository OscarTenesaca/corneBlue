# Corne Keyboard ZMK Configuration

Este repositorio contiene la configuración de ZMK para mi teclado Corne, incluyendo la optimización del ahorro de batería mediante el modo de suspensión tras un periodo de inactividad.

## Configuración de Ahorro de Energía

Para optimizar el ahorro de batería, se ha configurado el teclado para que entre en modo de ahorro de energía después de un periodo de inactividad. A continuación, se detallan los pasos para realizar esta configuración.

### Paso 1: Habilitar el Power Management

Añade las siguientes líneas al archivo `corne.conf` para habilitar la gestión de energía:

```conf
# Enable Power Management
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000
CONFIG_ZMK_SLEEP=y

```
Explicación de las Configuraciones

CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000: Establece el tiempo de inactividad antes de que el teclado entre en modo de ahorro de energía (deep sleep). Está en milisegundos, por lo que 900000 ms equivalen a 15 minutos (subido desde los 4 minutos originales a pedido explícito, coincide con el default de fábrica de ZMK).
CONFIG_ZMK_SLEEP=y: Habilita el modo de sueño.

Nota: `CONFIG_ZMK_IDLE_SLEEP` y `CONFIG_ZMK_POWER` que aparecían antes en este archivo no son opciones reales de ZMK (no existen en el Kconfig del firmware) — se quitaron junto con `config/custom_kconfig.conf`, que tampoco se llegaba a incluir en el build.

https://github.com/nperez0111/zmk-config/blob/main/config/nibble.conf
https://github.com/deintech/corne-zmk-config/blob/master/config/corne.conf
https://deploy-preview-722--zmk.netlify.app/docs/config/power/

---

## Compatibilidad de build (west.yml / GitHub Actions)

`config/west.yml` y `.github/workflows/build.yml` estaban sin pinnear (`revision: main` / `@main`), lo que hacía que cada build descargara el ZMK "de hoy". Cuando ZMK migró a Hardware Revisions (parte de la actualización a Zephyr 4.1), el board `nice_nano_v2` dejó de existir con ese nombre y el build empezó a fallar con `Invalid BOARD`.

**Solución aplicada:** ambos quedaron pinneados a `v0.3.0` (última release de ZMK anterior a esa migración):
- `config/west.yml` → `revision: v0.3.0`
- `.github/workflows/build.yml` → `uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3.0`

Esto mantiene `nice_nano_v2`, `build.yaml` y el resto de la configuración funcionando sin cambios.

---

## Batería / BLE — optimizaciones adicionales

Además del sleep timeout, en `corne.conf`:

- `CONFIG_BT_PERIPHERAL_PREF_MIN_INT=24` / `CONFIG_BT_PERIPHERAL_PREF_MAX_INT=40`: intervalo de conexión BLE devuelto al valor de fábrica de Zephyr (30-50ms) en vez del 6/12 (7.5-15ms) que usa ZMK por defecto — menos "despertares" de radio, batería un poco mejor, sin latencia perceptible al escribir.
- Encoder (`CONFIG_EC11`) eliminado — no hay hardware de encoder en este board, era config inerte.

### Indicador de batería (LED azul del nice!nano)

Módulo comunitario [`zmk-poor-mans-led-indicator`](https://github.com/BlueDrink9/zmk-poor-mans-led-indicator) (agregado en `west.yml`), usando el LED azul integrado del propio nice!nano (pin `P0.15`), **separado por completo** de la cadena RGB de 27 LEDs — funciona aunque el RGB esté apagado.

```conf
CONFIG_INDICATOR_LED_WIDGET=y
CONFIG_INDICATOR_LED_SHOW_BATTERY_ON_BOOT=y
CONFIG_INDICATOR_LED_BATTERY_LEVEL_HIGH=50
CONFIG_INDICATOR_LED_BATTERY_LEVEL_CRITICAL=10
```

Devicetree del LED en `config/nice_nano_v2.overlay` (nodo `leds` + alias `indicator-led`).

**Batería en macOS:** con `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING/PROXY=y` (ya estaban activos), ZMK reporta la batería de ambas mitades por BLE, pero macOS solo muestra la del lado central (izquierdo) en su UI nativa — es una limitación de macOS, no de esta config. Para ver ambas por separado: app de terceros [zmk-battery-center](https://github.com/kot149/zmk-battery-center).

---

## RGB (per-key + underglow)

Hardware: PCB Corne v3.0.1 (Typeractive), **27 LEDs SK6812/WS2812 por mitad** (6 underglow + 21 per-key, uno por switch), en una sola cadena serie por el pin `P0.06` (SPI3 MOSI). Ver `config/nice_nano_v2.overlay` para el detalle y las fuentes usadas para inferir el pin (Typeractive no documenta RGB para este PCB oficialmente).

**Estado actual: parcialmente funcional, sin resolver del todo.** Encienden los 6 underglow + los primeros ~4 per-key (10 de 27); el resto no responde. Se descartaron como causa: desajuste de chain-length, frecuencia SPI, brillo/corriente de batería, y defecto de hardware (confirmado con una prueba en QMK que sí encendió los 27). Sigue en investigación — valores actuales (`spi-max-frequency=2000000`, brillo al 15%, efecto Solid) son de diagnóstico, no definitivos.

Kconfig relevante en `corne.conf`: `CONFIG_ZMK_RGB_UNDERGLOW=y`, `CONFIG_WS2812_STRIP=y`, arranca apagado (`RGB_UNDERGLOW_ON_START=n`), se apaga solo en idle (`AUTO_OFF_IDLE=y`).

**Efectos reales disponibles (confirmado en el código fuente de ZMK v0.3.0):** solo existen 4 — Solid, Breathe, Spectrum (Rainbow), Swirl. No hay Reactive/Ripple/Fire/Comet/Scanner/Random ni nada per-key-reactivo en ZMK estándar; requeriría escribir un driver C nuevo (módulo fuera del árbol).

---

## Keymap — controles agregados

Todo en `adjust_layer` (`config/corne.keymap`), sin modificar el resto de las capas:

**Lado derecho — RGB:**
- `F`: tap = encender, doble-tap = apagar (`td_rgb_onoff`)
- `C` / `R` / `L` / siguiente col = salto directo a Solid / Breathe / Rainbow / Swirl (`RGB_EFS_CMD`)
- Fila "subir" (`D H T N`) y fila "bajar" (`B M W V`): Brillo / Saturación / Velocidad / Hue, +/− respectivamente

**Lado izquierdo — Bluetooth:**
- `BT0` / `BT1` / `BT2`: seleccionar perfil
- `BTCLR`: borrar el emparejamiento del perfil actual (recuperación cuando no reconecta)
- `BTDISC`: forzar desconexión/reconexión del perfil actual

### `default_layer` — atajos de plegado de código (VSCode / Antigravity)

`H`, `T`, `M`, `W` mantienen su letra en tap simple; en doble-tap envían un macro con la secuencia de 2 combos:

| Tecla | Doble-tap | Atajo enviado |
|---|---|---|
| `H` | Expandir todo | `⌘K` `⌘J` |
| `M` | Plegar todo | `⌘K` `⌘0` |
| `T` | Expandir nivel actual | `⌘K` `⌘]` |
| `W` | Plegar nivel actual | `⌘K` `⌘[` |

Funciona en VSCode y en Antigravity (basado en el editor de VSCode, hereda los mismos atajos base).

---

## Limpieza realizada

- `config/custom_kconfig.conf` eliminado (nunca se incluía en el build, nombre no coincidía con shield ni board).
- Behavior `gqt`/`global-quick-tap` eliminado (no se usaba en ninguna capa, y estaba deprecado).
- Behavior `td_ctrl_right` eliminado (solo se usaba una vez, se liberó esa posición).
- Propiedad `label` quitada de todos los `tap-dance`/`hold-tap` (deprecada en Zephyr, no afecta funcionalidad).
