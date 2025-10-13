# GolmarUNO

**Golmar Intercom UNO — Reverse engineering**

Proyecto: ingeniería inversa sobre la placa T-540 UNO SE (repuesto de T-740 UNO).

---

## Resumen

Este repositorio documenta el análisis y los hallazgos sobre la electrónica y el protocolo de comunicación del intercomunicador Golmar (T-540 / UNO). Incluye información de hardware, conexión al debug, extracción de firmware, características del bus serial y ejemplos de comandos observados durante la ingeniería inversa.

---

## Hardware principal

| Elemento                   | Descripción                                                                                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Microcontrolador           | LPC1114 — Data sheet: [https://www.nxp.com/docs/en/data-sheet/LPC111X.pdf](https://www.nxp.com/docs/en/data-sheet/LPC111X.pdf) |
| Debug                      | Sí                                                                                                                             |
| Modo debug                 | SWD                                                                                                                            |
| Conexión JTAG/SWD (header) | De abajo a arriba: `3v3`, `SWIO`, `SWCLK`, `NC`, `NC`, `GND`                                                                   |

### Extracción de firmware

Se ha extraído el firmware con OpenOCD. Comando usado:

```bash
openocd -f 'interface/stlink.cfg' -f 'target/lpc11xx.cfg' -c "adapter speed 1000" . reset_init; halt; flash read_bank 0 t-540.bin
```

---

## Acceso a UART / Bus uni-hilo

* La comunicación entre dispositivos es serial (hilo compartido / multidrop).
* Nivel físico: 5 V con polaridad invertida.
* El LPC1114 dispone de pines UART RX/TX estándares (compatibles con 3.3 V y 5 V).
* Para adaptar el microcontrolador al bus uni-hilo, la placa usa un amplificador/op-amp LM358 y un par de transistores.
* Pines del LPC conectados al bus: `37 (RX)` y `36 (TX)`.
* Bajo la placa hay dos test-points que permiten acceder fácilmente a RX/TX.

---

## Parámetros de la línea serial (observados desde el controlador)

| Parámetro     | Valor    |
| ------------- | -------- |
| Baud rate     | 2600 bps |
| Bits de datos | 8        |
| Paridad       | even     |
| Stop bits     | 1        |

> Nota: en foros se describe esta conexión como multidrop de 9 bits; el análisis de señal coincide con esa posibilidad, pero no se observaron diferencias prácticas entre usar 9-bit u 8-bit con paridad even. Por simplicidad y compatibilidad se documentan aquí los parámetros anteriores.

---

## Formato de mensajes

* Mensaje típico: 4 bytes.
* Los tres primeros bytes hacen referencia a la dirección (o son campos de dirección) y el cuarto byte es el comando.

**Ejemplo**

```
00 00 13 37  -> Placa principal solicita (pulsador)
00 00 13 10  -> Telefonillo inicia enrutamiento de audio
00 00 00 11  -> Comando general para limpiar el bus
00 00 13 90  -> Comando: abrir (solo válido durante llamada)
00 00 00 22  -> (Desde central del conserje) inicia llamada con la placa
00 00 00 90  -> (Desde central del conserje) acciona la apertura de puerta
```

### Flujo típico (T-540)

1. La placa principal inicia la comunicación (por ejemplo al pulsar el botón) enviando: `00 00 XX 37` (donde `XX` es la dirección del telefonillo).
2. El telefonillo con dirección `XX` responde con ACK: `00 00 XX 01`.
3. Cuando el telefonillo descolgado solicita audio, envía: `00 00 XX 10`.
4. A continuacion el telefonillo limpia el bus con: `00 00 00 11`.
5. La placa responde con ACK: `00 00 XX 01`.
6. Para abrir la puerta durante una llamada: `00 00 XX 90` (solo funcionará si hay una llamada en curso con el telefonillo).

### Particularidad — Central del conserje (CE-941)

* La central del conserje puede iniciar una llamada directamente con la placa, sin interacción previa de la placa principal, lo que permite accionar la puerta desde la central.
* Comandos observados:

  * Iniciar llamada: `00 00 00 22`
  * Abrir puerta: `00 00 00 90`
* Se recomienda una limpieza del bus (por ejemplo `00 00 00 11`) tras realizar estas acciones.

---

## Observaciones y recomendaciones

* El comando de apertura `0x90` sólo surte efecto durante una llamada — intentar reproducirlo fuera de ese contexto no funcionará con la placa principal observada.



