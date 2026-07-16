# Open-Source Hearing Assistive Device

##  WORK IN PROGRESS

This project is currently under active development. Expected completion: Early 2027

## Project Overview

An open-source hearing assistive device designed to provide high-quality audio amplification and processing at a fraction of the cost of commercial hearing aids.

### Key Features (Target)
- High-quality audio processing with DSP
- Bluetooth Low Energy (BLE) connectivity
- Compact Behind-The-Ear (BTE) form factor
- Rechargeable LiPo battery with temperature monitoring
- Open-source hardware and firmware
- Cost-effective design (< $150-200 BOM at scale)

## Current Status (June 2026)

### Completed
- Component selection and evaluation
- Initial schematic design (audio path, power management)
- Battery management architecture

###  In Progress
- Complete schematic review and finalization
- Battery integration and charging circuit design
- PCB layout (4-layer HDI)
- Firmware development (DSP algorithms, BLE stack)

###  Planned
- Prototype manufacturing and testing
- RF matching and optimization
- Audio quality tuning
- Documentation and build guide

## Technical Specifications

### Audio Processing
- Audio SoC: Nordic nrf5340 
- DSP capabilities: ANC, adaptive feedback cancellation
- Sample rate: 48kHz
- Latency: < 20ms

### Connectivity
- Bluetooth 5.4 LE Audio
- Range: ~10m

### Power
- Battery: LIR2450 (120mAh,24x5mm)
- Charging: Pogo-pin contact charging
- Battery protection: Over-charge, over-discharge, short-circuit, temperature monitoring (NTC)

### Physical
- Form factor: Behind-The-Ear (BTE)
- Target dimensions: 55x15x25mm

## Why Open Source?

Commercial hearing aids cost $1,000-$6,000 per pair. This project aims to:
1. Make hearing assistance accessible to everyone
2. Provide a platform for customization and research
3. Enable community-driven improvements
4. Reduce e-waste through repairability

## How to Contribute

This project is in early stages. Contributions welcome in:
- Hardware design review
- Firmware development (DSP, BLE)
- Documentation
- Testing and feedback

## License

- Hardware: CERN Open Hardware Licence v2 (Strongly Reciprocal)
- Firmware/Software: MIT License

## Contact

- GitHub: Emi4538 https://github.com/Emi4538
- Email: contact.opencircuit@proton.me
- YouTube: OpenCircuit (coming soon)

---

*Last updated: July 2026*
