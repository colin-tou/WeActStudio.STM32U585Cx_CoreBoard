# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

STM32U585CIU6 (WeAct Studio BlackPill) bare-metal firmware example demonstrating RTC, ADC (VBAT), and USB CDC-ACM. Every second it sends a formatted timestamp + battery voltage over USB virtual serial port. Button press toggles LED and sends a message.

## Build & Flash

This project was originally configured for **Keil MDK-ARM** (`MDK-ARM/01-RTC.uvprojx`) but the user works with **STM32CubeIDE** and an **STLink v3**.

**To build in STM32CubeIDE:**
1. `File → Import → Existing STM32CubeMX Configuration File (.ioc)` → select `01-RTC.ioc`
2. CubeIDE regenerates init code for GCC toolchain — say yes
3. `Project → Build` (`Ctrl+B`) → output in `Debug/` as `.elf`

**To flash:**
- `Run → Debug` (F11) — flashes via STLink v3 and halts at `main()`
- `Run → Run` — flash and run without debug

**SWD pins:** SWDIO=PA13, SWCLK=PA14

**Verify:** open any serial terminal on the USB CDC port — you'll see `20YY.MM.DD HH:MM:SS ,XmV` each second.

## Architecture

### Code ownership boundary
CubeMX generates and owns all `MX_*_Init()` functions and peripheral `.c/.h` files in `Core/`. Application logic lives inside `/* USER CODE BEGIN */` / `/* USER CODE END */` blocks and is preserved on regeneration.

Files **never touched by CubeMX**:
- `Bsp/board.c` / `Bsp/board.h` — LED (PC13) and button (PA0) helpers
- `USBX/App/` — USB CDC-ACM application layer, `cdc_acm` global handle

### Main loop (cooperative, tick-based)
`Core/Src/main.c` runs three interleaved tick-gated tasks — no RTOS:

| Task | Interval | What it does |
|---|---|---|
| USB system | 1 ms | `_ux_system_tasks_run()` — drives USBX state machine |
| Button/RTC | 100 ms (pressed) / 500 ms (released) | Reads RTC, formats `txbuf`, toggles LED |
| USB TX | 1 ms (+ 1000 ms backoff after send) | State machine: `UX_STATE_RESET` → `UX_STATE_WAIT` → sends `txbuf` via `ux_device_class_cdc_acm_write_run()` |

`txbuf` is shared between the RTC task (writer) and USB TX task (reader) — no locking, single-core is fine.

### ADC
`adc_inp` (global `uint32_t`) is filled continuously by GPDMA1 Channel 1 in circular mode. Voltage conversion in the format string: `(adc_inp * 3300) >> 12` (3.3V ref, 14-bit).

### Clocks
- SYSCLK: 160 MHz (HSE 25 MHz → PLL ÷5 ×64 ÷2)
- RTC: LSE 32.768 kHz (PC14/PC15)
- USB: HSI48 48 MHz

## Key config
- Heap: 1 KB / Stack: 8 KB
- USBX system memory: 6 KB
- TrustZone: disabled
- Firmware pack: STM32Cube FW_U5 V1.6.0
