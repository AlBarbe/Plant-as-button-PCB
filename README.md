<!-- HEADER -->
# Plant as Button — PCB 🌱🔌

Wouldn't be cool to use as a plant a touch sensor? Now it has a proper board to live on.
This repo hosts the hardware side of the project — the companion firmware (the code that actually reads the plant's touch) lives in [Plant-as-button](https://github.com/AlBarbe/Plant-as-button).

<!-- TABLE OF CONTENTS -->
<!--
<details>
  <summary>Table of contenent</summary>
    <ul>
      <li></li><a href="#introduction">Introduction</a></li>
      <li></li><a href="#how-it-works">How it works</a></li>
      <li></li><a href="#repository-structure">Repository structure</a></li>
      <li></li><a href="#status--known-issues">Status & known issues</a></li>
    </ul>
</details>
-->
<!-- INTRODUCTION -->
## Introduction

This is my first PCB design, done in KiCad 9. The idea was to give the plant-as-button concept a real, standalone board instead of a breadboard: an ESP32-S3 module, its own power supply, and a driver stage so the "button" can actually control something (an RGBWW LED strip) when the plant is touched.

The board has been manufactured and it works. Every opinion or suggestion is more than welcome!!

<!-- HOW IT WORKS -->
## How it works
- ### Concept
  The board is built around an ESP32-S3-MINI-1 module, using the same capacitive touch trick described in the firmware repo: a touch-capable GPIO is wired to a pad, meant to be connected (by a simple cable) to the soil near the plant's roots. When a person touches the plant, the capacitance seen on that GPIO changes, and the microcontroller reads it as a "press".

- ### Power
  The board can be powered either from USB-C or from an external 12V supply (screw terminal), with a power mux (TPS2116) automatically selecting between the two. From there, two buck converters step the voltage down to 5V and then 3.3V for the MCU and the rest of the logic. The 12V input is protected with a TVS diode, and the USB data lines have their own ESD protection.

- ### LED driver
  Five low-side MOSFETs let the ESP32 switch a common-anode RGBWW LED strip (Red, Green, Blue, Warm White, Cool White), connected via a 6-pin screw terminal. The idea is the same as the firmware's trigger class: on touch, on release, or after a given time, drive one of the channels.

- ### Debug breakout
  Basically every GPIO of the module is broken out to its own test pad on the silkscreen (labeled with the pin function, not just a generic reference), on top of the usual RESET/BOOT buttons. This turned out to be very handy, see below.

## Repository structure
- `PAB_V1.kicad_pro / .kicad_sch / .kicad_pcb` — the KiCad 9 project (schematic + 4-layer PCB, single sheet).
- `Libraries/` — symbols, footprints and 3D models used by the project (kept local so the repo is self-contained).
- `Datasheets/` — datasheets of the main parts used.
- `BOM.ods` — bill of materials.

## Status & known issues
The board has been fabricated and is working. There is one known bug: the USB-UART bridge (CP2102) is not wired correctly to the ESP32-S3, so programming/monitoring through the on-board USB-C via the CP2102 doesn't work.
It's not a dead end though — the ESP32-S3 has its own native USB, and since every GPIO (including the native USB D+/D-) is broken out to a test pad, the board can still be flashed and debugged by using those pins directly, bypassing the CP2102 entirely.

## Implementation
Firmware-wise, this board is meant to be paired with the code from [Plant-as-button](https://github.com/AlBarbe/Plant-as-button): the touch pad on this PCB is where the plant's cable connects, and the LED terminal is a ready-made output for whatever trigger logic you build on top of it.
