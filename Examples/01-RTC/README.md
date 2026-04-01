# 01-RTC — STM32U585 RTC + USB CDC Example

WeAct Studio STM32U585CIU6 BlackPill example demonstrating RTC timekeeping, ADC battery voltage monitoring, and USB virtual serial port output.

## What it does

- Prints `20YY.MM.DD HH:MM:SS ,XmV` over USB CDC once per second
- Pressing the user button toggles the LED and sends `Key Pressed` over USB
- Battery voltage is sampled continuously via ADC1 (VBAT channel) + GPDMA

## Hardware

| | |
|---|---|
| MCU | STM32U585CIUx @ 160 MHz (Cortex-M33) |
| Board | WeAct Studio STM32U585Cx CoreBoard |
| LED | PC13 |
| Button | PA0 (active low) |
| RTC crystal | 32.768 kHz on PC14/PC15 (LSE) |
| HSE crystal | 25 MHz on PH0/PH1 |
| USB | PA11 (D−) / PA12 (D+) — Full Speed |
| SWD | PA13 (SWDIO) / PA14 (SWCLK) |

## Building & Flashing

**Requirements:** STM32CubeIDE + STLink v3

1. `File → Import → Existing STM32CubeMX Configuration File (.ioc)` → select `01-RTC.ioc`
2. Let CubeIDE regenerate code for GCC
3. `Project → Build` (`Ctrl+B`)
4. Connect STLink v3 to the SWD header, then `Run → Debug` (F11) to flash

> The project was originally created for Keil MDK-ARM (`MDK-ARM/01-RTC.uvprojx`). The `.ioc` file can be used to regenerate for either toolchain.

## Clocks

| Domain | Source | Frequency |
|---|---|---|
| SYSCLK | HSE → PLL (÷5 ×64 ÷2) | 160 MHz |
| RTC | LSE | 32.768 kHz |
| USB | HSI48 | 48 MHz |
| ADC | HCLK/8 | ~20 MHz |

## Firmware Pack

STM32Cube FW_U5 V1.6.0 · CubeMX 6.12.1
