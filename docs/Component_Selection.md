# Component Selection & Design Decisions

*Last updated: June 2026*

## Project Philosophy

Unlike existing open-source hearing aid projects (Tympan, openMHA, openHA), which are primarily development platforms for research, this project aims to create a real, everyday-wearable Behind-The-Ear (BTE) hearing assistive device.

### Key Design Goals:
- **Wearable form factor**: Must be comfortable to wear all day, discreet enough for daily use
- **High-quality audio**: Balanced armature speakers, DSP processing, low latency (<20ms)
- **Modern connectivity**: Bluetooth 5.4 LE Audio for smartphone streaming
- **User-replaceable battery**: Case battery charges 2× hearing aids
- **Open source**: Hardware (CERN OHL v2) + Firmware (MIT License)

### Target Specifications:
- Form factor: BTE (Behind-The-Ear)
- Dimensions: 55x15x25mm (estimated)
- Battery life: 5.7 hours (with LIR2450)
- Audio: 4× MEMS microphones, 2-3× balanced armature speakers
- Connectivity: Bluetooth 5.4 LE Audio

---

## Audio Processing

### Audio SoC: Nordic nRF5340

**Selected:** Nordic nRF5340 (WLCSP package)

**Why:**
- **Dual-core architecture**: Application core (Cortex-M33, 128MHz) + Network core (Cortex-M33, 64MHz)
- **Integrated DSP**: Hardware-accelerated audio processing
- **Bluetooth 5.4 LE Audio**: Native support for LC3 codec, low-latency audio streaming
- **Ultra-low power**: Optimized for battery-operated wearables
- **Excellent SDK support**: nRF Connect SDK with Zephyr RTOS
- **Small package**: WLCSP 4.4 × 4.0 mm

**Alternatives considered:**
- **Qualcomm QCC5181**: Good audio features, but more complex licensing and larger package
- **Realtek RTL8763E Series**: availability constraints
**Trade-offs:**
- WLCSP package requires professional PCB assembly (JLCPCB SMD service)
- Steeper learning curve compared to Arduino-based platforms

---

## Power Management

### PMIC: Nordic nPM1300

**Selected:** Nordic nPM1300

**Why:**
- **Optimized for nRF5340**: Designed by Nordic specifically for nRF5-series
- **Integrated charger**: LiPo/Li-Ion charging with programmable current
- **Multiple voltage rails**: 1.8V, 3.0V, 3.3V outputs
- **Ultra-low quiescent current**: ~1.5µA
- **Small package**: WLCSP 3.1 × 2.4 mm

**Alternatives considered:**
- **TPS62740**: Good efficiency, but no integrated charger
- **MCP73831**: Simple linear charger, but inefficient and no voltage regulation

---

### Hearing Aid Battery: LIR2450


**Selected:** LIR2450 rechargeable coin cell (⌀24 × 5mm, 120mAh, 3.6V)

**Why:**
- **User-replaceable**: Standard coin cell form factor, easy to swap
- **Rechargeable**: Li-Ion chemistry, can be charged in case
- **Compact**: ⌀24mm fits in BTE housing (mounted vertically: 5mm width × 24mm height)
- **Good capacity**: 120mAh provides realistic runtime for daily use

**Realistic Battery Life (calculated):**
- **Idle (99% idle, standby):** ~12 hours
- **Normal use (continuous audio streaming):** ~5.7 hours
- **Heavy use (BLE + DSP + high volume):** ~2.1 hours

**Power Consumption Breakdown:**
- nRF5340 (BLE Audio): ~10-15mA
- SSM2518 Amplifier: ~5-10mA
- 4× IMP34DT05 Microphones: ~4mA
- nPM1300 PMIC: ~1-2mA
- Other (LED, etc.): ~1mA
- **Total average:** ~21-32mA

**Charging Configuration (nPM1300):**
- **Recommended charge current:** 0.5C = 60mA (cell-friendly)
- **Maximum charge current:** 1C = 120mA (faster but stresses cell)
- **Realistic charging time:**
  - Conservative (0.5C): ~2.5 hours
  - Fast (1C): ~1.2 hours
  - *(Includes CC/CV curve factor of 1.2-1.5×)*

**Design decision for v1:** Configure nPM1300 for 60mA charging (0.5C) to prioritize cell longevity (400-500 cycles target) over fast charging.


**Design note:** Battery is mounted vertically in the housing (5mm width × 24mm height), fitting within the 15mm housing width. User swaps battery in charging case overnight.

---

### Case Battery: LP102550

**Selected:** LP102550 (50×25×10mm, 1200mAh, 3.7V)

**Why:**
- **Charges 2× hearing aids**: 1200mAh can fully charge 2× LIR2450 (120mAh each) multiple times
- **Compact case form factor**: 50×25×10mm fits in pocket-sized charging case
- **Standard LiPo**: Easy to source, well-understood chemistry

---

### Battery Protection: S-8261 + External MOSFETs

**Selected:** ABLIC S-8261 Battery Protection IC + 2× external MOSFETs

**Why:**
- **Overcharge protection**: Cuts off at 4.28V
- **Over-discharge protection**: Cuts off at 2.4V
- **Short-circuit protection**: Fast response time
- **Low quiescent current**: ~3µA

**Critical note:** S-8261 requires 2× external MOSFETs (high-side + low-side switching):
- **High-side MOSFET**: P-Channel (e.g., SI2301, SOT-23)
- **Low-side MOSFET**: N-Channel (e.g., SI2302, SOT-23)

**Alternatives considered:**
- **DW01A + FS8205A**: Cheaper, but larger footprint and less precise thresholds
- **Integrated protection (e.g., BQ29700)**: More expensive, overkill for single-cell

---

## Audio Input

### Microphones: 4× INVENSENSE IMP34DT05

**Selected:** 4× IMP34DT05 MEMS microphones (digital PDM output)

**Why:**
- **Digital PDM output**: Direct connection to nRF5340, no ADC needed
- **High SNR**: 64dB, excellent for hearing aid applications
- **Small package**: 3.0 × 4.0 × 1.0 mm
- **Low power**: ~1mA per microphone
- **4-microphone array**: Enables beamforming and advanced noise reduction

**Why 4 microphones (not 2)?**
- **Beamforming**: Directional audio capture, focus on speaker in front
- **Noise cancellation**: Adaptive noise reduction in noisy environments
- **Spatial audio**: Better localization of sound sources
- **Future-proof**: Enables advanced DSP algorithms (ANC, wind noise reduction)

**Alternatives considered:**
- **Analog microphones (e.g., CMM-4030D)**: Require ADC, more complex design
- **2× microphones**: Simpler, but no beamforming capability

---

## Audio Output

### Speakers: 2-3× Knowles WBFK-30095-000

**Selected:** Knowles WBFK-30095-000 Balanced Armature receivers

**Why:**
- **Balanced armature technology**: Superior audio quality, low distortion
- **Compact**: 5.0 × 2.73 × 1.98 mm
- **High efficiency**: Good sensitivity for low-power operation
- **Wide frequency response**: Optimized for speech intelligibility
- **2-3 units**: Stereo + dedicated mid-range for better clarity

**Alternatives considered:**
- **Dynamic receivers (e.g., Knowles ED series)**: Cheaper, but larger and lower quality
- **Single speaker**: Simpler, but no stereo or dedicated frequency bands

---

### Amplifier: Analog Devices SSM2518

**Selected:** SSM2518 Stereo Class-D amplifier (I2S input)

**Why:**
- **Digital I2S input**: Direct connection from nRF5340, no DAC needed
- **Class-D efficiency**: >90% efficiency, critical for battery life
- **Integrated DSP**: Optional EQ and DRC (Dynamic Range Compression)
- **Small package**: WLCSP 4.0 × 4.0 mm
- **Stereo output**: Drives 2× speakers directly

**Required external components:**
- Decoupling capacitors: 1µF + 100nF (PVDD), 100nF (DVDD)
- I2C pull-up resistors: 4.7kΩ (if using I2C control mode)
- Optional output LC filter (if speaker cable >20cm)

**Alternatives considered:**
- **MAX98357A**: Similar features, but SSM2518 has better Nordic integration
- **Analog amplifier (e.g., LM4871)**: Less efficient, requires DAC

---

## Connectivity
### Component: Johanson 2450AT18B100 (2.4 GHz Ceramic Chip Antenna)

**Why this component was chosen:**
* **Compact Footprint:** The ultra-small 3.2 x 1.6 mm SMD package saves critical PCB real estate compared to traditional PCB trace antennas.
* **Design Simplicity:** Frees up board space by eliminating the large keep-out zones required by inverted-F or meander trace antennas.
* **Reliable Performance:** Specifically optimized for 2.4 GHz ISM bands (BLE, Wi-Fi, Zigbee), offering high radiation efficiency and stable return loss in dense layouts.

**Required External Components & Layout Rules:**
* **Impedance Matching Network:** A Pi-network (typically 2 capacitors and 1 inductor) is required to tune the antenna to 50Ω. *Note: Exact values must be tuned based on the final PCB ground plane.*
* **DC Blocking Capacitor:** A series capacitor (typically 100pF – 1nF) is required if the RF transceiver output pin carries a DC bias.
* **Strict Keep-out Zone:** All inner/outer layer copper, traces, and ground planes must be completely cleared beneath the antenna element to prevent detuning.


### Crystals

**Main clock:** ABM8-32.000MHz-B2-T (3.2 × 2.5 × 0.8 mm)
**RTC clock:** ABS07-32.768KHZ-7-T (3.2 × 1.5 × 0.9 mm)

**Why:**
- **Standard frequencies**: 32MHz for nRF5340, 32.768kHz for RTC
- **Small packages**: Fit in compact design
- **Good stability**: ±20ppm, sufficient for BLE timing

---

## ESD Protection

### Ladeport / VBUS: PRTR5V0U2X (NXP)

**Selected:** PRTR5V0U2X (SOT363: 2.0 × 2.1 mm)

**Why:**
- **Dual channel**: Protects both VBUS and data lines
- **Low clamping voltage**: <10V at 3A
- **Small package**: Fits in tight space

---

### RF-Pfad: RCLAMP0521P (Semtech)

**Selected:** RCLAMP0521P (SOD-523: 1.2 × 0.8 mm)

**Why:**
- **Ultra-low capacitance**: <0.5pF, doesn't degrade RF performance
- **Fast response**: <1ns
- **Tiny package**: Minimal PCB footprint

---

### SWD / I/O: ESD5Z-Serie (TVS)

**Selected:** 2-3× ESD5Z (SOD-323: 1.7 × 1.25 mm)

**Why:**
- **General-purpose protection**: For debug port, buttons, LEDs
- **Low clamping voltage**: Good protection level
- **Standard package**: Easy to source

---

## Physical Interface

### Pogo-Pins: 3× CPG-80-SMT-B (Same Sky)

**Selected:** 3× CPG-80-SMT-B (1.85 × 1.48 mm SMD)

**Why:**
- **Multiplexed functionality**:
  - Pin 1: VBUS (charging input)
  - Pin 2: GND
  - Pin 3: NTC temperature measurement / diagnostic
- **No mechanical wear**: Spring-loaded contacts
- **Compact**: Small SMD footprint
- **Reliable**: Good mating cycle life

**Design note:** Pogo-pins are used instead of USB-C or mechanical connectors to save space and improve reliability.

---

### Debug Connector: Tag-Connect TC2030

**Selected:** Tag-Connect TC2030 (3.81 × 9.27 mm)

**Why:**
- **No header needed**: Saves PCB space
- **Standard SWD**: Compatible with J-Link, ST-Link
- **Spring-loaded pins**: Reliable connection during programming

---

### Button: Tactile button (e.g., Alps SKRPABE010)

**Selected:** 3.0 × 2.0 × 1.5 mm tactile button

**Why:**
- **Power / Pairing**: Single button for multiple functions (short press, long press, double press)
- **Compact**: Small footprint
- **Reliable**: 100k+ cycle life

---

### Status LED: 0402 LED

**Selected:** Standard 0402 LED (1.0 × 0.5 mm)

**Why:**
- **Power / BT status**: Visual feedback
- **Ultra-compact**: Smallest standard LED package
- **Low current**: ~2mA with appropriate resistor

---

## Passive Components

### Ferrite Beads: 3-5× BLM15 series (Murata)

**Selected:** 0402 ferrite beads for power line EMI filtering

**Why:**
- **EMI suppression**: Reduces high-frequency noise on power lines
- **Low DC resistance**: Minimal voltage drop
- **Small package**: 0402 (1.0 × 0.5 mm)

---

### Decoupling Capacitors: ~20× 0402 MLCC

**Selected:** 100nF + 1µF per IC (0402 package)

**Why:**
- **Power integrity**: Stable voltage rails
- **High-frequency bypass**: Reduces noise
- **Standard values**: Easy to source

---

### I2C Pull-up Resistors: 2× 4.7kΩ (0402)

**Selected:** 4.7kΩ for SDA/SCL lines

**Why:**
- **I2C standard**: Required for I2C bus operation
- **Optimal value**: 4.7kΩ balances speed and power consumption

---

### Diverse Pull-up/down: ~6× 0402 resistors

**Selected:** Various values for microphone LR pins, SWD, button

**Why:**
- **Defined logic levels**: Prevent floating inputs
- **Configurable**: Different values for different functions

---

## Manufacturing

### PCB Assembly: JLCPCB SMD Service

**Selected:** JLCPCB for PCB fabrication + SMD assembly

**Why:**
- **WLCSP capability**: Can assemble fine-pitch packages (nRF5340, nPM1300)
- **Cost-effective**: Much cheaper than manual assembly
- **Quick turnaround**: ~1-2 weeks
- **Good quality**: Suitable for prototypes and small batches

**Design considerations:**
- All BOM components must be available in JLCPCB library
- Fiducial marks required for pick-and-place
- Panelization for efficient assembly

---

## Related Projects

This project is inspired by and builds upon existing open-source hearing aid projects:

- **Tympan** (github.com/tympan): Open-source hearing aid based on Teensy 3.6
- **openMHA** (github.com/HoerTech-gGmbH/openMHA): Open-source hearing aid algorithm platform
- **OpenHA** (github.com/openaudiology/openha): Open-source hearing aid device

### Key Differentiators:

| Feature | Existing Projects | This Project |
|---------|------------------|--------------|
| **Form factor** | Development board (hand-held) | Real BTE hearing aid (wearable) |
| **Size** | 100×80×30mm+ | ~33×40×12mm |
| **Use case** | Lab research | Daily wear |
| **Audio SoC** | Teensy 3.6 (older) | Nordic nRF5340 (modern) |
| **Connectivity** | None or basic BLE | Bluetooth 5.3 LE Audio |
| **Battery** | USB charging | User-replaceable coin cell + charging case |
| **Microphones** | 1-2× | 4× (beamforming capable) |
| **Speakers** | Dynamic receiver | Balanced armature (high-end) |

---

## Bill of Materials (BOM) Summary

| Category | Component Count | Estimated Cost (1×) | Estimated Cost (100×) |
|----------|----------------|---------------------|----------------------|
| Audio SoC + PMIC | 2 | ~$15 | ~$12 |
| Battery + Protection | 4 | ~$8 | ~$6 |
| Microphones | 4 | ~$6 | ~$4 |
| Speakers | 2-3 | ~$20 | ~$15 |
| Antenna + RF | 5-6 | ~$3 | ~$2 |
| ESD Protection | 5-6 | ~$2 | ~$1.5 |
| Passives | ~40 | ~$3 | ~$1 |
| Connectors + Interface | 5 | ~$5 | ~$3 |
| PCB (4-layer) | 1 | ~$10 | ~$5 |
| **TOTAL** | **~70** | **~$72** | **~$49.5** |

**Note:** Costs are estimates based on DigiKey/Mouser pricing (June 2026). Actual costs may vary based on supplier, quantity, and shipping.

---

## Design Constraints & Challenges

### 1. Space Constraints
- **Target volume**: ~15 cm³ (33×40×12mm)
- **Challenge**: Fit all electronics + battery in compact BTE form factor
- **Solution**: Aggressive component placement, 4-layer PCB, vertical battery mounting

### 2. RF Performance
- **Challenge**: Antenna ground plane (25×10mm) takes significant PCB area
- **Solution**: Place antenna at PCB edge, optimize ground plane layout

### 3. Thermal Management
- **Challenge**: LiPo charging + audio processing generates heat in sealed enclosure
- **Solution**: Thermal vias under hot components, copper pours for heat spreading

### 4. Acoustic Design
- **Challenge**: Microphone-speaker isolation to prevent feedback
- **Solution**: Physical separation, acoustic damping, DSP feedback cancellation

### 5. Assembly Complexity
- **Challenge**: WLCSP packages require professional assembly
- **Solution**: Use JLCPCB SMD assembly service

---

## Revision History

- **June 2026**: Initial component selection, design philosophy defined
- **TODO**: Add RF matching network values after antenna testing
- **TODO**: Finalize BOM after prototype testing

---

*This document will be updated as the design progresses. All component choices are subject to change based on prototype testing and validation.*
