# GolmarUNO

**Golmar Intercom UNO — Reverse Engineering**

Project: reverse engineering of the T-540 UNO SE board (replacement for the T-740 UNO).

---

## Overview

This repository documents the analysis and findings related to the electronics and communication protocol of the Golmar intercom (T-540 / UNO).  
It includes hardware information, debug access, firmware extraction, serial bus characteristics, and examples of commands observed during the reverse engineering process.

> ⚠️ Use at your own risk. The author takes no responsibility for any damage that may occur.

---

## Main Hardware

| Component                | Description                                                                                                                    |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Microcontroller           | LPC1114 — Data sheet: [https://www.nxp.com/docs/en/data-sheet/LPC111X.pdf](https://www.nxp.com/docs/en/data-sheet/LPC111X.pdf) |
| Debug Interface           | Yes                                                                                                                           |
| Debug Mode                | SWD                                                                                                                           |
| JTAG/SWD Header (bottom to top) | `3v3`, `SWIO`, `SWCLK`, `NC`, `NC`, `GND`                                                                              |

### Firmware Extraction

Firmware was successfully extracted using OpenOCD. Example command:

```bash
openocd -f 'interface/stlink.cfg' -f 'target/lpc11xx.cfg' -c "adapter speed 1000" . reset_init; halt; flash read_bank 0 t-540.bin
```

To open a debug session:

```bash
telnet localhost 4444
reset_init; halt; flash read_bank 0 t-540.bin
```

---

## UART / Single-Wire Bus Access

* Communication between devices is serial (shared line / multidrop).  
* Physical level: 5 V with inverted polarity.  
* The LPC1114 uses standard UART RX/TX pins (compatible with 3.3 V and 5 V).  
* To adapt the MCU to the single-wire bus, the board uses an LM358 op-amp and a pair of transistors.  
* LPC pins connected to the bus: `37 (RX)` and `36 (TX)`.  
* Two test points under the board allow easy access to RX/TX signals.

---

## Serial Line Parameters (observed from the controller)

| Parameter     | Value    |
| -------------- | -------- |
| Baud rate      | 2600 bps |
| Data bits      | 8        |
| Parity         | even     |
| Stop bits      | 1        |

> Some forum posts describe this connection as a 9-bit multidrop bus. Signal analysis supports that possibility, but no practical difference was observed between using 9-bit or 8-bit with even parity. For simplicity and compatibility, the parameters above are documented.

---

## Message Format

* Each message consists of 4 bytes.  
* The first three bytes are address fields, and the fourth byte is the command.

**Examples:**

```
00 00 13 37  -> Main board request (button press)
00 00 13 10  -> Handset starts audio routing
00 00 00 11  -> General command to clear the bus
00 00 13 90  -> Command: open (only valid during a call)
00 00 00 22  -> (From concierge unit) initiates call with the panel
00 00 00 90  -> (From concierge unit) triggers door unlock
```

### Typical Flow (T-540)

1. The main board starts communication (e.g., when pressing a button) by sending `00 00 XX 37` (where `XX` is the handset address).  
2. The handset with address `XX` responds with ACK: `00 00 XX 01`.  
3. When the handset is picked up and requests audio, it sends `00 00 XX 10`.  
4. The handset then clears the bus with `00 00 00 11`.  
5. The main board replies with ACK: `00 00 XX 01`.  
6. To open the door during a call: `00 00 XX 90` (only works while a call is active).

### Concierge Unit (CE-941)

* The concierge unit can initiate a call directly with the panel, without prior interaction from the main board, allowing it to unlock the door remotely.  
* Default address: `00 00 00`.  
* Observed commands:

  * Start call: `00 00 00 22`  
  * Unlock door: `00 00 00 90`

* It’s recommended to clear the bus afterward (e.g., `00 00 00 11`).

### Command Summary

| Action                  | Command     |
| ------------------------ | ----------- |
| Incoming Call            | 00 00 XX 37 |
| ACK                      | 00 00 XX 01 |
| Clear BUS                | 00 00 00 11 |
| Start Audio              | 00 00 XX 10 |
| Panic Call               | 00 00 XX 44 |
| Unlock Door              | 00 00 XX 90 |
| Start Call with Panel    | 00 00 00 22 |

---

## Notes and Recommendations

* The unlock command `0x90` only works during an active call — sending it outside that context won’t have any effect with the observed main board.

---

## ESPHome Integration

A sample ESPHome YAML schema for home automation is included.  
Place a **diode** between the ESP32 TX pin and the intercom TX line. Without it, incoming calls may not be received properly by the handset.  
Keep in mind the intercom is powered by **17 V** — you’ll need a **buck converter** to step it down to **5 V**.

```bash
       |rx intercom - ----- - rx esp32 
D -----|
       |tx intercom - diode - tx esp32
```
