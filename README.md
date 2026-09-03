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

### Estado aceptado: 10 de 27 LEDs

Encienden los 6 underglow + los primeros ~4 per-key (10 de 27); el resto no responde. Es una limitación conocida y aceptada, no un bug pendiente de arreglar. Se investigó a fondo antes de aceptarla:

- **Hardware descartado**: mismo PCB/controlador probado con QMK, los 27 LEDs encendieron correctamente, uno por uno.
- **Config descartada**: `chain-length` correcto (27, confirmado por conteo físico), frecuencia SPI probada en 2MHz/4MHz/8MHz sin ningún cambio, brillo/corriente de batería bajado al 15% sin cambio.
- **Sin error de software**: con `CONFIG_ZMK_USB_LOGGING` (snippet oficial `zmk-usb-logging`) y log en modo debug, al activar `RGB_ON` no aparece ningún error de `spi`/`ws2812`/`led_strip` — el driver reporta la transmisión como exitosa.
- **Los 2 bugs conocidos de Zephyr para esto en nRF52840 ya estaban evitados** desde el inicio: se usa `compatible = "nordic,nrf-spim"` (no el driver legacy sin DMA, zephyrproject-rtos/zephyr#29877) y SPI3 (la única instancia que funciona bien en nRF52840, zephyrproject-rtos/zephyr#57147).

**Alternativas evaluadas y no tomadas** (por riesgo o falta de referencia verificada, no por pereza):
- `bits-per-symbol` reducido: sin ejemplo verificado para nRF52840, riesgo real de empeorar lo que ya funciona si el cálculo de temporización sale mal.
- Driver I2S (`worldsemi,ws2812-i2s`): existe en Zephyr, nRF52840 tiene el periférico, pero tampoco hay referencia verificada.
- Driver GPIO bit-banging de Zephyr: no soporta nRF52840 (solo nRF51).
- Escribir un driver GPIO bit-banging propio (como probablemente hace QMK): viable en teoría, pero es desarrollo de firmware real (días de trabajo), no una edición de config.
- Preguntar en el Discord de ZMK: sigue siendo una opción de bajo riesgo si se retoma esto más adelante.

Kconfig relevante en `corne.conf`: `CONFIG_ZMK_RGB_UNDERGLOW=y`, `CONFIG_WS2812_STRIP=y`, arranca apagado (`RGB_UNDERGLOW_ON_START=n`), se apaga solo en idle (`AUTO_OFF_IDLE=y`), brillo tope 60%, efecto por defecto Breathe.

**Efectos reales disponibles (confirmado en el código fuente de ZMK v0.3.0):** solo existen 4 — Solid, Breathe, Spectrum (Rainbow), Swirl. No hay Reactive/Ripple/Fire/Comet/Scanner/Random ni nada per-key-reactivo en ZMK estándar; requeriría escribir un driver C nuevo (módulo fuera del árbol).

---

## Keymap — controles agregados

### Cómo llegar a `adjust_layer`

`adjust_layer` es la capa 3 (índice), donde viven los controles de RGB y Bluetooth. Dos formas de activarla:
- Anidada: mantener `lower_layer` y, dentro de esa capa, su propia tecla de pulgar `mo 3`.
- **Más práctica**: en `default_layer`, el pulgar derecho (antes solo `Alt`) es ahora `td_ralt_adjust` — tap/doble-tap = `Alt` normal, **triple-tap = activa/desactiva `adjust_layer`** (usa `&tog 3`, se queda encendida hasta que la vuelvas a activar con otro triple-tap; funciona igual desde dentro de `adjust_layer` para volver a `default_layer`).

**Lado derecho de `adjust_layer` — RGB:**
- `F`: tap = encender, doble-tap = apagar (`td_rgb_onoff`)
- `C` / `R` / `L` / siguiente col = salto directo a Solid / Breathe / Rainbow / Swirl (`RGB_EFS_CMD`)
- Fila "subir" (`D H T N`) = Brillo+ / Saturación+ / Velocidad+ / Hue+
- Fila "bajar" (`B M W V`) = Brillo− / Saturación− / Velocidad− / Hue−

**Lado izquierdo de `adjust_layer` — Bluetooth:**
- `BT0` / `BT1` / `BT2`: seleccionar perfil
- `BTCLR`: borrar el emparejamiento del perfil actual (recuperación cuando no reconecta)
- `BTDISC`: forzar desconexión/reconexión del perfil actual

### `default_layer` — atajos adicionales

**Plegado de código (VSCode / Antigravity)** — `H`, `T`, `M`, `W` mantienen su letra en tap simple; en doble-tap envían un macro con la secuencia de 2 combos:

| Tecla | Doble-tap | Atajo enviado |
|---|---|---|
| `H` | Expandir todo | `⌘K` `⌘J` |
| `M` | Plegar todo | `⌘K` `⌘0` |
| `T` | Expandir nivel actual | `⌘K` `⌘]` |
| `W` | Plegar nivel actual | `⌘K` `⌘[` |

Funciona en VSCode y en Antigravity (basado en el editor de VSCode, hereda los mismos atajos base).

**Números rápidos** — `P` doble-tap = `4`, `Y` doble-tap = `5`.

**`/` y `?`** — esa tecla (antes `KP_SLASH` fijo) ahora es `td9`: tap = `/`, doble-tap = `?`.

---

## Limpieza realizada

- `config/custom_kconfig.conf` eliminado (nunca se incluía en el build, nombre no coincidía con shield ni board).
- Behavior `gqt`/`global-quick-tap` eliminado (no se usaba en ninguna capa, y estaba deprecado).
- Behavior `td_ctrl_right` eliminado (solo se usaba una vez, se liberó esa posición).
- Propiedad `label` quitada de todos los `tap-dance`/`hold-tap` (deprecada en Zephyr, no afecta funcionalidad).
