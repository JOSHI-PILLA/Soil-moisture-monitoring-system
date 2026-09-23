# Soil Moisture Monitoring System

**IoT-based water level indication using NodeMCU (ESP8266)**
Project by: **Pilla Joshi**

A low-cost embedded system that senses soil moisture, decides whether the soil is dry or adequately wet, and reports that decision both visually (LEDs) and digitally (serial log) — forming a base that can later be extended into a fully automated, Wi-Fi-connected irrigation controller.

---

## Overview

Irrigation is often scheduled by habit rather than by actual soil condition, wasting water. This project uses a resistive soil moisture probe connected to a NodeMCU ESP8266's analog input to continuously measure soil condition and act on it in real time — no display, app, or network connection required.

## Features

- Continuous analog soil moisture sampling once per second
- Threshold-based wet/dry decision using the ADC's 10-bit range
- Dual-LED local status indication (green = OK, red = low)
- Live serial telemetry (raw value + status) at 115200 baud
- Built on the ESP8266, so Wi-Fi upgrades need firmware changes only — no new hardware
- No external libraries — core Arduino functions only

## Hardware Required

| Component | Notes |
|---|---|
| NodeMCU ESP8266 (ESP-12E) | Main controller |
| Resistive soil moisture sensor (LM393 breakout) | Analog + digital output |
| Green LED | "WATER OK" indicator |
| Red LED | "WATER LOW" indicator |
| 220–330 Ω resistors ×2 | LED current limiting |
| Jumper wires | Male-to-male / male-to-female |
| Micro-USB cable (data-capable) | Power + programming |

## Wiring / Pin Configuration

| Signal | NodeMCU Pin | GPIO | Direction |
|---|---|---|---|
| Soil sensor analog output | A0 | ADC0 | Input |
| Green status LED | D5 | GPIO14 | Output |
| Red status LED | D6 | GPIO12 | Output |
| Sensor supply | 3V3 | — | Power |
| Common ground | GND | — | Reference |

> ⚠️ The bare ESP8266 chip only tolerates 0–1.0 V on its ADC pin. The NodeMCU board's on-board divider scales this to ~0–3.3 V — do not wire the sensor's analog output directly to a bare ESP-12E module without an external divider.

## Software Setup

1. Install the **Arduino IDE** and add the ESP8266 board core via Boards Manager.
2. Select board: **NodeMCU 1.0 (ESP-12E Module)**.
3. Set upload speed to **115200 baud**.
4. Connect the board and select the correct COM/serial port.
5. Upload the sketch, then open the Serial Monitor at **115200 baud** (must match the firmware rate, or output will be unreadable).

## How It Works

1. **Sample** — read the analog voltage on A0 (0–1023).
2. **Report** — print the raw value to the serial port.
3. **Decide** — compare the value against the threshold (default **512**, the ADC midpoint).
4. **Actuate** — value > 512 → soil is dry → red LED ON, print `WATER LOW`. Otherwise → green LED ON, print `WATER OK`.
5. Repeat every 1 second.

Soil conductivity rises with water content, so **wet soil → low ADC reading**, **dry soil → high ADC reading**.

## Calibration

The default threshold (512) is a safe starting point, but for best results calibrate per soil type:

1. Insert the probe into thoroughly **dry** soil and record the steady reading (dry reference).
2. Water the soil to the ideal moisture level and record the new steady reading (wet reference).
3. Set the threshold to the **midpoint** of the two references and re-flash.
4. Verify by cycling the probe between conditions and confirming clean switching.

Factors that shift readings: soil type (clay vs. sand), fertiliser content, temperature, probe insertion depth/contact, and supply voltage stability.

## Test Results

| Condition | ADC Range | LED | Serial Output |
|---|---|---|---|
| Dry soil / air | 590 – 593 | Red ON | `WATER LOW` |
| Moist soil | 378 – 403 | Green ON | `WATER OK` |

Both clusters sit well clear of the 512 threshold (~190-count separation), and state transitions occur on the very next sampling cycle after the probe is moved.

## Limitations

- Resistive probes corrode over time under continuous DC excitation (capacitive probes recommended for long-term use)
- No hysteresis — a reading sitting exactly at the threshold can toggle between states
- Only one ADC channel — multiple probes need an external multiplexer/ADC
- Output is a relative, calibration-dependent verdict, not an absolute moisture percentage
- No data persistence if the host PC is disconnected

## Future Enhancements

- Add hysteresis (two thresholds) to eliminate boundary flicker
- Average multiple samples to reduce noise
- Drive a relay/MOSFET to automatically switch an irrigation pump
- Publish readings over Wi-Fi (MQTT/HTTP) to a dashboard such as ThingSpeak or Blynk
- Power the probe only during measurement to reduce electrode corrosion
- Add a DHT22 sensor for temperature/humidity correlation
- Migrate to a capacitive soil moisture probe

