Corne Keyboard ZMK — Claude Code Instructions
Contexto del proyecto
Este repositorio contiene la configuración de ZMK para mi teclado Corne.
La configuración actual funciona correctamente y debe considerarse estable.

El objetivo principal de Claude Code en este repositorio es ayudarme a:

Configurar y controlar los LEDs/RGB del teclado.
Encender y apagar los LEDs.
Cambiar efectos de iluminación.
Cambiar colores, brillo y velocidad de los efectos cuando sea compatible.
Crear keybindings para controlar la iluminación desde el teclado.
Optimizar la configuración de LEDs sin romper la configuración existente.
Ayudar a depurar problemas relacionados con LEDs/RGB y ZMK.
Regla crítica: no modificar Power Management
La configuración actual de ahorro de energía NO debe cambiarse.
Actualmente el teclado entra en modo de suspensión después de 4 minutos de inactividad.

La configuración existente es:

# Enable Power Management

CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=240000
CONFIG_ZMK_IDLE_SLEEP=y
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_POWER=y

Estas opciones deben mantenerse exactamente como están, salvo que yo solicite explícitamente modificarlas.
No hacer
Claude NO debe:
Eliminar CONFIG_ZMK_IDLE_SLEEP_TIMEOUT.
Cambiar 240000.
Desactivar CONFIG_ZMK_IDLE_SLEEP.
Desactivar CONFIG_ZMK_SLEEP.
Desactivar CONFIG_ZMK_POWER.
Cambiar la lógica actual de suspensión.
Sustituir el sistema actual de Power Management por otra implementación.
Modificar la configuración de batería o suspensión como parte de una tarea relacionada con LEDs.
"Optimizar" el ahorro de energía automáticamente.
Si una configuración de LEDs entra en conflicto con el Power Management existente, primero explicar el conflicto y proponer una solución que preserve el comportamiento actual.
Principio de cambios mínimos
Antes de modificar archivos:
Inspeccionar la configuración existente.
Identificar cómo está configurado actualmente el hardware.
Reutilizar la configuración existente siempre que sea posible.
Hacer el cambio mínimo necesario.
No reorganizar archivos ni hacer refactors innecesarios.
No modificar funcionalidades que no estén relacionadas con la tarea solicitada.
No asumir que una configuración estándar de ZMK es mejor que la configuración existente.
LEDs / RGB
La prioridad actual del proyecto es implementar un sistema de control de iluminación para el Corne.
Quiero poder, cuando el hardware y la versión de ZMK lo permitan:

Encender los LEDs.
Apagar los LEDs.
Cambiar entre diferentes efectos.
Cambiar colores.
Ajustar brillo.
Ajustar velocidad de los efectos.
Controlar los LEDs mediante keybindings.
Mantener los LEDs apagados cuando sea necesario para ahorrar batería.
Antes de implementar una funcionalidad de LEDs:
Revisar qué hardware de LEDs tiene realmente el teclado.
Revisar los archivos .conf, .overlay, .dtsi, keymap y shields existentes.
Determinar si se trata de RGB underglow, RGB individual, backlight u otro sistema.
Comprobar qué soporte proporciona la versión de ZMK utilizada por este repositorio.
No asumir que una API, Kconfig, binding o feature existe simplemente porque aparece en documentación antigua, forks o ejemplos externos.
Compatibilidad con ZMK
La compatibilidad con la versión de ZMK utilizada en este repositorio es prioritaria.
Si existe una diferencia entre:

documentación actual,
documentación antigua,
ejemplos de otros forks,
configuración de otro teclado,
se debe priorizar la configuración compatible con este repositorio.
No copiar configuraciones completas de otros repositorios sin verificar qué partes son necesarias.

Referencias
Estas referencias pueden utilizarse como ejemplos para investigar configuraciones existentes:
https://github.com/nperez0111/zmk-config/blob/main/config/nibble.conf
https://github.com/deintech/corne-zmk-config/blob/master/config/corne.conf
https://deploy-preview-722--zmk.netlify.app/docs/config/power/
Las referencias externas son únicamente material de consulta.
No asumir que una configuración de esos repositorios debe copiarse directamente.

Keymap
Cuando se agreguen controles para LEDs:
Preferir keybindings claros y fáciles de recordar.
Evitar sobrescribir keybindings existentes.
Revisar primero el keymap actual.
Mantener intactas las capas y bindings existentes.
Si no existe una combinación adecuada, proponer una nueva combinación antes de modificar el keymap.
Ejemplos de funcionalidades que podrían implementarse:
LED ON
LED OFF
Next Effect
Previous Effect
Increase Brightness
Decrease Brightness
Increase Speed
Decrease Speed
Next Color
Previous Color

Los nombres exactos y bindings deben determinarse según las capacidades reales de ZMK y del hardware.
Batería
El teclado es inalámbrico, por lo que el consumo energético es importante.
Al trabajar con LEDs:

Evitar mantener iluminación innecesariamente activa.
Considerar el consumo de RGB al diseñar efectos.
Preferir apagado automático cuando sea compatible.
No eliminar ni alterar el sleep actual de 4 minutos.
No introducir funcionalidades que mantengan el teclado despierto innecesariamente.
Si una funcionalidad de LEDs puede afectar significativamente la batería, indicarlo antes de implementarla.
La configuración de Power Management existente tiene prioridad sobre cualquier optimización adicional de LEDs.
Antes de realizar cambios
Cuando solicite una nueva funcionalidad, primero analizar:
Hardware
↓
Shield / Device Tree
↓
Kconfig / .conf
↓
ZMK feature disponible
↓
Keymap
↓
Build

Si falta información crítica, inspeccionar primero los archivos del repositorio antes de proponer cambios.
No inventar nombres de:

Kconfig options.
Device Tree nodes.
Devicetree labels.
ZMK behaviors.
Keybindings.
Shields.
Drivers.
APIs.
Si no se puede confirmar que una opción existe para esta versión de ZMK, decirlo claramente.
Compilación y validación
Después de realizar cambios relacionados con LEDs:
Revisar los archivos modificados.
Verificar que no se haya modificado accidentalmente Power Management.
Compilar el firmware si las herramientas del repositorio están disponibles.
Revisar errores y warnings relevantes.
Si la compilación falla, investigar la causa antes de hacer cambios adicionales.
No considerar una configuración terminada simplemente porque los archivos parecen correctos.
Git
Este repositorio es un fork y la carpeta .claude/ es exclusivamente para configuración local de Claude Code.
No modificar ni eliminar las reglas de .gitignore relacionadas con .claude/.

No intentar agregar .claude/ al repositorio.

Filosofía de trabajo
Este proyecto ya tiene una configuración funcional.
La regla general es:

Extend the existing configuration; don't replace it.
Especialmente:
LED functionality must not break the existing 4-minute power-saving behavior.
Si para implementar una funcionalidad de LEDs es necesario modificar una configuración existente, explicar primero:
Qué archivo debe cambiarse.
Por qué es necesario.
Qué impacto tiene.
Cómo se preservará el comportamiento actual de ahorro de energía.
No realizar cambios destructivos sin confirmación explícita.
