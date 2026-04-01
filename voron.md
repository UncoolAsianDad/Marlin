# Voron 0 — Marlin context (this repo)

## Hardware

| Item | Detail |
|------|--------|
| Machine | Voron 0 (**CoreXY**), print volume **120 × 120 × 120 mm** |
| Mainboard | Makerbase **MKS GEN_L V2.1** (`BOARD_MKS_GEN_L_V21`) |
| Display | **MKS Mini12864 V3.0** (`MKS_MINI_12864_V3`, FYSETC 2.1 pinout) |
| Stepper drivers | **BigTreeTech TMC2208 V3.0** (UART). Marlin: `TMC2208` on X/Y/Z/E0 — same setting as TMC2225 modules. |
| Motor supply | **12 V** kit PSU → `CHOPPER_TIMING` = `CHOPPER_DEFAULT_12V` |

## Firmware / build

- **Marlin** 2.1.x in this tree; **PlatformIO** env: `mega2560` (default in `platformio.ini`).
- **Kinematics:** `#define COREXY` in `Configuration.h` (Voron 0 belt layout). If X/Y move wrong after flashing, use `INVERT_X_DIR` / `INVERT_Y_DIR` or swap to `COREYX` only if your build matches that variant.
- **SD:** `SDSUPPORT` — use the **SD slot on the LCD**, not only the onboard slot.
- **NeoPixels (display RGB bar):** `NEOPIXEL_LED`, `NEO_RGB`, `NEOPIXEL_PIXELS` 3; `LED_CONTROL_MENU`, `LED_COLOR_PRESETS`, `LED_USER_PRESET_STARTUP` enabled per Marlin recommendations for this panel.

## Motion / thermal baseline

Defaults in `Configuration.h` were aligned with an **M503** capture from the previous printer firmware (see log below). That firmware reported *hardcoded defaults* and EEPROM not loaded — treat as a starting point; **re-verify** steps, direction, homing, and probe after flashing.

- Steps: M92 X80 Y80 Z400 **E380**
- Max accel M201: 2000, 2000, 100, **1000** (E)
- M204 P/R/T: **2000**
- Junction deviation M205 J: **0.01**
- Preheat 1 (M145 S0): **H195 B60**; preheat 2: **H240 B110**
- PID M301: **P22.20 I1.08 D114**

## Wiring / pins to double-check

- **TMC UART:** PDN_UART per axis to RAMPS **AUX-2** pairs as in Marlin `pins_RAMPS.h` (`HAS_TMC_UART` defaults for MKS GEN_L). On **BTT TMC2208 V3.0**, set the onboard jumpers for **UART mode** per [BIGTREETECH-TMC2208-V3.0](https://github.com/bigtreetech/BIGTREETECH-TMC2208-V3.0) (not standalone STEP/DIR-only). Typical wiring: **1 kΩ** MCU TX → PDN_UART as in Marlin’s TMC UART notes; RX to PDN_UART directly for readback. With **one wire pair per driver** (Marlin’s default here), each module can use **slave address 0**. Sense resistor on these boards is usually **0.11 Ω** (`*_RSENSE` **0.11** in `Configuration_adv.h` — change only if your PCB says otherwise).
- **Filament runout:** `FILAMENT_RUNOUT_SENSOR` enabled; default RAMPS-style pin is often **SERVO3** (pin **32** on this board). Change `FIL_RUNOUT_PIN` if your sensor is elsewhere, or disable runout if unused.
- **Filament change:** `ADVANCED_PAUSE_FEATURE` + `NOZZLE_PARK_FEATURE`; runout script **M600** (needs correct sensor pin).

## Reference: serial log (old firmware M503)

Captured from the machine before this config (Marlin bugfix-2.0.x, hardcoded defaults, SD init fail on that build):

```
Processing mega2560 (board: megaatmega2560; platform: atmelavr@~4.0.1; framework: arduino)
----------------------------------------------------------------------------------------------------------------------------------------- Terminal on COM12 | 250000 8-N-1
--- Available filters and text transformations: colorize, debug, default, direct, hexlify, log2file, nocontrol, printable, send_on_enter, time
--- More details at https://bit.ly/pio-monitor-filters
--- Quit: Ctrl+C | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H
19:18:54.284 > start
19:18:54.285 > echo:Marlin bugfix-2.0.x
19:18:54.285 >
19:18:54.286 > echo: Last Updated: 2019-12-02 | Author: (none, default config)
19:18:54.288 > echo:Compiled: Jun 23 2025
19:18:54.288 > echo: Free Memory: 3499  PlannerBufferBytes: 1200
19:18:59.596 > echo:SD init fail
19:18:59.596 > echo:Hardcoded Default Settings Loaded
19:18:59.597 > echo:  G21    ; Units in mm (mm)
19:18:59.599 > echo:  M149 C ; Units in Celsius
19:18:59.600 >
19:18:59.600 > echo:Filament settings: Disabled
19:18:59.601 > echo:  M200 D1.75
19:18:59.603 > echo:  M200 D0
19:18:59.603 > echo:Steps per unit:
19:18:59.604 > echo: M92 X80.00 Y80.00 Z400.00 E380.00
19:18:59.605 > echo:Maximum feedrates (units/s):
19:18:59.607 > echo:  M203 X300.00 Y300.00 Z5.00 E25.00
19:18:59.609 > echo:Maximum Acceleration (units/s2):
19:18:59.611 > echo:  M201 X2000.00 Y2000.00 Z100.00 E1000.00
19:18:59.612 > echo:Acceleration (units/s2): P<print_accel> R<retract_accel> T<travel_accel>
19:18:59.617 > echo:  M204 P2000.00 R2000.00 T2000.00
19:18:59.618 > echo:Advanced: B<min_segment_time_us> S<min_feedrate> T<min_travel_feedrate> J<junc_dev>
19:18:59.622 > echo:  M205 B20000.00 S0.00 T0.00 J0.01
19:18:59.623 > echo:Home offset:
19:18:59.625 > echo:  M206 X0.00 Y0.00 Z0.00
19:18:59.626 > echo:Material heatup parameters:
19:18:59.627 > echo:  M145 S0 H195 B60 F0
19:18:59.627 > echo:  M145 S1 H240 B110 F0
19:18:59.628 > echo:PID settings:
19:18:59.630 > echo:  M301 P22.20 I1.08 D114.00
19:18:59.631 > echo:LCD Contrast:
19:18:59.632 > echo:  M250 C220
19:18:59.632 > echo:Filament load/unload lengths:
19:18:59.634 > echo:  M603 L0.00 U100.00
19:18:59.635 > echo:Filament runout sensor:
19:18:59.638 > echo:  M412 S1
```
