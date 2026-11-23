# Dual-Channel LED Driver Module (3S Li-ion Powered)

An efficient, dual-channel constant-current LED driver system designed to run directly off a **3S Li-ion battery pack (9.0V – 12.6V)** without the need for intermediate buck converters. This design uses two independent **Texas Instruments TPS61165** boost regulator ICs to drive a bank of 15 white LEDs and a bank of 20 red LEDs with optimal efficiency and protection against thermal runaway.

---

## 🛠️ Circuit Architecture Overview

| Parameter | Channel 1: White LEDs | Channel 2: Red LEDs |
| :--- | :--- | :--- |
| **Driver IC** | **TPS61165** (Boost) | **TPS61165** (Boost) |
| **LED Count & Matrix** | 15 LEDs (3 Parallel × 5 Series) | 20 LEDs (2 Parallel × 10 Series) |
| **Per-LED Spec (Assumed)** | ~3.2V @ 20mA | ~2.0V @ 20mA |
| **Target Output Voltage (\(V_{OUT}\))** | **16.0 V** | **20.0 V** |
| **Target Output Current (\(I_{OUT}\))** | **60 mA** | **40 mA** |
| **Sense Resistor (\(R_{SET}\))** | **3.3 Ω** | **5.1 Ω** |

### Why This Topology?
* **No Linear Ballast Resistors:** Series strings inherently balance current natively within the branch, eliminating the need for 35 individual resistors and preventing thermal runaway domino failures.
* **Pure Boost Operation:** By structuring the red LEDs into strings of 10 (\(V_{OUT} = 20\text{V}\)), the target voltage stays safely above the maximum fully charged battery voltage (\(12.6\text{V}\)), allowing a common boost architecture for both channels.

---

## 📋 Bill of Materials (BOM) Per Channel

To build this system on your PCB, you will need to populate the following supporting components for **each** TPS61165 chip:

1. **IC₁ (LED Driver):** Texas Instruments TPS61165
2. **L₁ (Inductor):** `10 µH` to `22 µH` shielded power inductor. Saturation current (\(I_{SAT}\)) **must be \(\ge\) 1.5A**.
3. **D₁ (Schottky Diode):** Fast switching Schottky diode (e.g., **MBR0540** or **B140**). Must be rated for \(\ge\) 30V or 40V reverse breakdown voltage.
4. **C_{IN} (Input Capacitor):** `4.7 µF` to `10 µF` X7R Ceramic Capacitor (\(\ge\) 16V rated). Place immediately adjacent to the chip's power pins.
5. **C_{OUT} (Output Capacitor):** `1 µF` to `4.7 µF` X7R Ceramic Capacitor. **Crucial:** Must be rated for **\(\ge\) 35V or 50V** to handle the high-voltage boost output safely.
6. **C_{COMP} (Compensation):** `220 nF` Ceramic Capacitor connected from the `COMP` pin to Ground.
7. **R_{SET} (Current Sense Resistor):** 0603 or 0805 thick film surface mount resistor. 
   * *White Channel:* **3.3 Ω** (sets total output to 60mA)
   * *Red Channel:* **5.1 Ω** (sets total output to 40mA)

---

## 📐 PCB Layout & Routing Guidelines

The TPS61165 operates at a high switching frequency of **1.2 MHz**. Adhering strictly to these routing rules is necessary to prevent electromagnetic interference (EMI) and unexpected voltage ripple.

### Recommended Trace Widths (assuming 1 oz/ft² copper)
* **High-Current Power Input (Battery to Chip):** `0.6 mm to 1.0 mm` (24 to 40 mils). Applies to \(V_{IN}\) power rails running through the input decoupling capacitors, inductor, and IC power pin.
* **High-Frequency Switching Node:** `0.8 mm to 1.2 mm` (32 to 48 mils). Keep the node connecting the IC's `SW` pin, the inductor, and the Schottky diode **as short and wide as physically possible**. Do not route this trace near sensitive analog signals.
* **LED Output & Feedback Loop:** `0.25 mm to 0.4 mm` (10 to 16 mils). Safe for the 40mA–60mA smooth DC running out to the LED panels and returning back to the `FB` pin.

### Grounding & Component Placement
* Implement a continuous, unbroken **Ground Plane** on the bottom layer of the board.
* Connect the ground pads of \(C_{IN}\), \(C_{OUT}\), and \(R_{SET}\) straight down into the ground plane using short, fat traces or dedicated ground stitching vias.
* Place the loop consisting of the IC, Inductor, Diode, and Output Capacitor tightly clustered together. Keep the loop area minimized.

---

## 🎛️ Control & Dimming
Each driver can be turned on/off or dimmed independently using the `CTRL` pin. 
* **ON/OFF:** Pull `CTRL` High (\(\ge\) 1.2V) to enable, pull Low (\(\le\) 0.4V) to disable.
* **Dimming:** Apply a standard logic-level PWM signal from a microcontroller to the `CTRL` pin (acceptable frequency range: 5kHz to 100kHz).
