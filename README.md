# Reflow Oven & Heating Table Controller (STM32F103C8T6)

*Читать на русском: [README_ru.md](README_ru.md)*

Hardware design (KiCad schematics and PCB layout) for a dual-channel controller designed for reflow ovens, hot plates, infrared/convection soldering stations, and PCB drying chambers.

The controller is based on the **STM32F103C8T6** microcontroller and supports two independent heating zones (e.g., top and bottom heaters) along with two K-type thermocouples via **MAX6675** cold-junction compensated converters.

---

## Key Features & Hardware Architecture

* **Microcontroller:** STM32F103C8T6 (LQFP-48) with an external 8 MHz HSE crystal and a 32.768 kHz LSE crystal for RTC.
* **Dual-Channel Temperature Sensing:** Supports two K-type thermocouple modules based on **MAX6675** sharing a common SPI bus (`PA5` SCK, `PA6` MISO) with dedicated chip-select lines (`PA3`, `PA4`).
* **Isolated Sensor Power Rails:** To minimize noise and cross-talk between thermocouple readings, the MCU and each MAX6675 module are powered by separate **AMS1117-3.3** linear voltage regulators (`U1`, `U2`, `U3`).
* **Dual-Channel Heater Control:** Control outputs (`PB8` and `PB9`) are buffered through dual Schmitt-trigger inverters **74HC14D** (`U6`) with pull-down resistors to guarantee that power switches (SSR / MOSFET) remain safely off during power-up and MCU reset.
* **User Interface:**
  * **LCD1602** character display interface over I2C (`PB6` SCL, `PB7` SDA).
  * Rotary encoder with push-button (`PA0`, `PA1`, `PA2`) for menu navigation and thermal profile configuration.
  * External status LED outputs for both heater channels and buzzer indication buffered via **74HC14D** (`U5`).
  * Piezo/magnetic buzzer driven by an `MMBT3904` NPN transistor (`PB12`).
* **Protection & Debugging:**
  * Resettable PTC fuse `MF-MSMF075-2` (0.75 A) and input bulk/bypass filtering on the +5 V power rail.
  * Dedicated **SWD** and **USART** (`PA9` TX, `PA10` RX) headers for flashing and debugging.

---

## Repository Structure

* `ControllerSTM32F103C8T6.kicad_pro` — KiCad project file.
* `ControllerSTM32F103C8T6.kicad_sch` — Root schematic sheet.
* `power.kicad_sch` — Power supply and 3.3 V LDO regulators sheet.
* `mcu.kicad_sch` — STM32F103C8T6, Schmitt-trigger buffers, and connectors sheet.
* `ControllerSTM32F103C8T6.kicad_pcb` — PCB layout (with 4× M3 mounting holes).
* `out/ControllerSTM32F103C8T6_bom.txt` — Bill of Materials (BOM).
* `out/ControllerSTM32F103C8T6_net.txt` — Simplified netlist.

---

## Connector Pinouts

### J1 — Power Input (`+5V In`, JST-XH 2-pin)

| Pin | Signal | Description |
| :---: | :--- | :--- |
| 1 | `+5P` | +5 V power input (before fuse F1) |
| 2 | `GND` | Ground |

### J10 — Heater Control Outputs (`Signal Out`, JST-XH 4-pin)

| Pin | Signal | MCU Pin (via U6) | Description |
| :---: | :--- | :--- | :--- |
| 1 | `GND` | — | Channel 1 ground |
| 2 | `HEATER_OUT_01` | `PB8` (pin 45) | Heater 1 control signal (3.3 V logic) |
| 3 | `GND` | — | Channel 2 ground |
| 4 | `HEATER_OUT_02` | `PB9` (pin 46) | Heater 2 control signal (3.3 V logic) |

### J7 / J8 — Thermocouple Sensors (`MAX6675_01` / `MAX6675_02`, JST-XH 5-pin)

| Pin | Signal | MCU Pin | Description |
| :---: | :--- | :--- | :--- |
| 1 | `+3V3_MAX6675_01 / 02` | — | Dedicated +3.3 V supply |
| 2 | `GND` | — | Ground |
| 3 | `MAX_SCK` | `PA5` (pin 15) | SPI clock (SCK) |
| 4 | `MAX_CS_01 / 02` | `PA3` (pin 13) / `PA4` (pin 14) | Chip Select |
| 5 | `MAX_MISO` | `PA6` (pin 16) | SPI data from MAX6675 (SO / MISO) |

### J4 — Display (`LCD1602`, JST-XH 4-pin)

| Pin | Signal | MCU Pin | Description |
| :---: | :--- | :--- | :--- |
| 1 | `GND` | — | Ground |
| 2 | `+5V` | — | +5 V display power |
| 3 | `LCD_SDA` | `PB7` (pin 43) | I2C1 data line (SDA) |
| 4 | `LCD_SCL` | `PB6` (pin 42) | I2C1 clock line (SCL) |

### J6 — Rotary Encoder (`Rotary Encoder`, JST-XH 5-pin)

| Pin | Signal | MCU Pin | Description |
| :---: | :--- | :--- | :--- |
| 1 | `+5V` | — | Encoder module power |
| 2 | `RENC_A` | `PA0` (pin 10) | Encoder phase A |
| 3 | `RENC_B` | `PA1` (pin 11) | Encoder phase B |
| 4 | `RENC_BUTTON` | `PA2` (pin 12) | Encoder push-button |
| 5 | `GND` | — | Ground |

### J9 — External Status LEDs (`Led Out`, JST-XH 5-pin)

| Pin | Signal | MCU Pin (via U5) | Description |
| :---: | :--- | :--- | :--- |
| 1 | `HEATER_LED_OUT_01` | `PB10` (pin 21) | Heater 1 LED anode (via 330 Ω resistor) |
| 2 | `GND` | — | Heater 1 LED cathode |
| 3 | `HEATER_LED_OUT_02` | `PB11` (pin 22) | Heater 2 LED anode (via 330 Ω resistor) |
| 4 | `GND` | — | Heater 2 LED cathode |
| 5 | `BUZZER_LED_OUT` | `PB13` (pin 26) | Buzzer indicator LED anode (via 330 Ω resistor) |

### J3 — UART Interface (`USART`, JST-XH 4-pin)

| Pin | Signal | MCU Pin | Description |
| :---: | :--- | :--- | :--- |
| 1 | `GND` | — | Ground |
| 2 | `TX` | `PA9` (pin 30) | USART1 transmit |
| 3 | `GND` | — | Ground |
| 4 | `RX` | `PA10` (pin 31) | USART1 receive |

### J5 — Debug Interface (`SWD`, PinHeader 1x5, 2.54 mm pitch)

| Pin | Signal | MCU Pin | Description |
| :---: | :--- | :--- | :--- |
| 1 | `+3V3_STM32` | — | +3.3 V reference voltage |
| 2 | `SWDIO` | `PA13` (pin 34) | SWD data I/O |
| 3 | `SWCLK` | `PA14` (pin 37) | SWD clock |
| 4 | `NRST` | `NRST` (pin 7) | Hardware reset |
| 5 | `GND` | — | Ground |

---

## Assembly & Hardware Notes

1. **I2C (`J4`) & Rotary Encoder (`J6`) Logic Levels:**
   * Standard PCF8574 I2C backpacks for LCD1602 include onboard pull-up resistors tied to `+5V`. Remove those pull-ups and tie `SDA`/`SCL` to `+3.3V` (or use a bidirectional logic level shifter).
   * Pins `PA0`–`PA2` are **not** 5V-tolerant. If your rotary encoder breakout board has onboard pull-up resistors, power the encoder from `3.3 V` instead of `+5V`.
2. **32.768 kHz Crystal (`Y2`):** Load capacitors `C19` and `C31` are `6.8 pF`, suitable for crystals rated at `CL = 6–7 pF`. If using a standard `CL = 12.5 pF` crystal, replace `C19` and `C31` with `15–18 pF` C0G/NP0 capacitors.
3. **Flashing Firmware:** Close jumper `J2 (Boot)` to pull `BOOT0` high when flashing via the built-in UART bootloader (`J3`). Keep `J2` open during normal operation.
