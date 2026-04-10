# Tilt Detection using ADXL335 Analog Accelerometer

A tilt detection system built around the ADXL335 3-axis analog accelerometer and IC741 op-amp comparator circuits. The system converts continuous analog tilt signals into discrete directional outputs — left, right, forward, back — indicated by LED indicators.

This repository contains both the **hardware circuit documentation** (the original academic project) and an **Arduino sketch** that replicates the comparator logic in software for reference and extension.

Built as a group academic project during B.Sc. Electronics at Christ University (2022–2023).

---

## How it works

The ADXL335 outputs an analog voltage on each of its three axes (X, Y, Z). At rest on a flat surface, each axis outputs approximately 1.65V (half of the 3.3V supply). Tilting the sensor shifts these voltages proportionally to how much gravitational acceleration acts on that axis.

```
Flat surface:
  X = 1.65V (~0g)   Y = 1.65V (~0g)   Z = 2.0V (~1g, gravity pulls down)

Tilted left:
  X = 1.2V (~-1g)   Y = 1.65V (~0g)   Z = varies
```

The IC741 op-amp is configured as a **voltage comparator** — it compares the axis voltage against a fixed reference voltage (set by a resistor divider). When the axis voltage crosses the threshold, the comparator output switches from LOW to HIGH, driving an LED indicator.

```
            ADXL335
         ┌───────────┐
         │  XOUT ────┼──────────────┐
         │  YOUT ────┼──────────┐   │
         │  ZOUT ────┼──────┐   │   │    IC741 Comparators
         │  VCC  ────┼──┐   │   │   │   ┌────────────────┐
         │  GND  ────┼──┘   │   │   └───┤(+)             │
         └───────────┘      │   │       │     IC741  OUT──┼── LED LEFT/RIGHT
                            │   └───────┤(-)             │
                            │           └────────────────┘
                     Ref voltage (resistor divider sets tilt threshold)
```

Each axis drives its own comparator. The two comparators (one per axis direction) produce a **2-bit digital output** — 4 possible states — mapped to 4 LED indicators.

---

## Hardware

### Components

| Component | Quantity | Purpose |
|---|---|---|
| ADXL335 accelerometer module | 1 | 3-axis analog tilt sensing |
| IC741 op-amp | 2 | Voltage comparator — analog threshold detection |
| LEDs | 4 | Visual output — Left, Right, Forward, Back |
| Resistors 220Ω | 4 | Current limiting for LEDs |
| Resistors 10kΩ | 4 | Voltage divider for comparator reference |
| Arduino Uno | 1 | Power supply + serial monitor (optional) |
| Breadboard + jumper wires | — | Circuit assembly |

### Wiring

```
ADXL335 VCC  → Arduino 3.3V pin
ADXL335 GND  → Arduino GND
ADXL335 XOUT → Arduino A0  (and to IC741 pin 3, non-inverting input)
ADXL335 YOUT → Arduino A1  (and to IC741 pin 3, non-inverting input)
ADXL335 ZOUT → Arduino A2  (monitoring only)

IC741 pin 2 (inverting input) → Resistor divider reference voltage
IC741 pin 6 (output)          → 220Ω resistor → LED → GND

LED_LEFT    → Arduino D5 (via 220Ω)
LED_RIGHT   → Arduino D6 (via 220Ω)
LED_FORWARD → Arduino D7 (via 220Ω)
LED_BACK    → Arduino D8 (via 220Ω)
```

---

## The Arduino sketch

`tilt_detection.ino` replicates the IC741 comparator logic in software. It reads the ADXL335 analog outputs, converts them to g-force values, and drives the same LED outputs — making it easy to test and calibrate without the full comparator circuit.

### Conversion pipeline

```
Raw ADC (0–1023)
      │
      ▼
Voltage = (raw / 1023) × 3.3V
      │
      ▼
Acceleration (g) = (voltage − 1.65) / 0.300
      │
      ▼
Compare against TILT_THRESHOLD (default: 0.3g ≈ 17°)
      │
      ├── gX < −0.3g  →  LED_LEFT ON
      ├── gX > +0.3g  →  LED_RIGHT ON
      ├── gY > +0.3g  →  LED_FORWARD ON
      └── gY < −0.3g  →  LED_BACK ON
```

### Serial monitor output (example)

```
=== Tilt Detection — ADXL335 ===
─────────────────────────────────
Axis_X(g)   Axis_Y(g)   Axis_Z(g)   Direction
X: 0.02g     Y: 0.01g     Z: 0.98g     → Flat
X: -0.61g    Y: 0.03g     Z: 0.78g     → LEFT
X: 0.03g     Y: 0.54g     Z: 0.82g     → FORWARD
X: 0.58g     Y: 0.02g     Z: 0.79g     → RIGHT
```

### How to run it

1. Wire up the ADXL335 and LEDs as described in the wiring section above
2. Open `tilt_detection.ino` in Arduino IDE
3. Select board: **Arduino Uno** and the correct COM port
4. Upload the sketch
5. Open Serial Monitor (Tools → Serial Monitor) at **9600 baud**
6. Tilt the board and watch the LEDs and serial output respond

### Calibration

Adjust these values at the top of the sketch to tune behaviour:

| Constant | Default | Effect |
|---|---|---|
| `TILT_THRESHOLD` | `0.3` | Lower = more sensitive, Higher = less sensitive |
| `ZERO_G_OFFSET` | `1.65` | Adjust if your sensor reads non-zero at rest |
| `SENSITIVITY` | `0.300` | Change only if using a different accelerometer |
| `SAMPLE_INTERVAL` | `100` | Milliseconds between readings |

---

## The IC741 comparator — how it works

The IC741 is a general-purpose operational amplifier used here in **open-loop comparator mode**. In this configuration:

- The non-inverting input (+) receives the ADXL335 axis voltage
- The inverting input (−) receives a fixed reference voltage set by a resistor divider
- When (+) > (−): output swings HIGH → LED lights
- When (+) < (−): output swings LOW → LED off

Two comparators are used — one for the X axis and one for the Y axis. Each comparator handles one direction of tilt (left/right for X, forward/back for Y), producing a 2-bit digital output across the four LEDs.

This is a foundational analog circuit technique used in sensor interfacing, motor control, battery management systems, and industrial automation.

---

## Applications of tilt sensing

- **Consumer electronics** — screen auto-rotation in phones and tablets
- **Automotive** — vehicle rollover detection, stability control
- **Robotics** — balance and orientation feedback
- **Medical devices** — patient posture monitoring
- **Gaming controllers** — motion-based gesture input

---

## Project context

Group project completed during B.Sc. Physics, Mathematics & Electronics at Christ University. The team designed and assembled the comparator circuit on a breadboard, calibrated the IC741 threshold voltages using a resistor divider, and validated tilt detection across all four directions. Contributions included circuit design, component selection, testing, and threshold calibration.

---

## Skills demonstrated

`Analog Circuit Design` `ADXL335 Accelerometer` `IC741 Op-Amp` `Comparator Circuits` `Arduino` `C++` `Sensor Integration` `Analog-to-Digital Conversion` `Embedded Systems` `Signal Thresholding`
