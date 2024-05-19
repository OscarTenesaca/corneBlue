# Corne Keyboard ZMK Configuration

Este repositorio contiene la configuración de ZMK para mi teclado Corne, incluyendo la optimización del ahorro de batería mediante el modo de suspensión tras un periodo de inactividad.

## Configuración de Ahorro de Energía

Para optimizar el ahorro de batería, se ha configurado el teclado para que entre en modo de ahorro de energía después de 4 minutos de inactividad. A continuación, se detallan los pasos para realizar esta configuración.

### Paso 1: Habilitar el Power Management

Añade las siguientes líneas al archivo `corne.conf` para habilitar la gestión de energía:

```conf
# Enable Power Management
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=240000
CONFIG_ZMK_IDLE_SLEEP=y
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_POWER=y

```
Explicación de las Configuraciones

CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=240000: Establece el tiempo de inactividad antes de que el teclado entre en modo de ahorro de energía. Está configurado en milisegundos, por lo que 240000 ms equivalen a 4 minutos.
CONFIG_ZMK_IDLE_SLEEP=y: Habilita el modo de sueño por inactividad.
CONFIG_ZMK_SLEEP=y: Habilita el modo de sueño.
CONFIG_ZMK_POWER=y: Habilita las configuraciones de energía en ZMK.

