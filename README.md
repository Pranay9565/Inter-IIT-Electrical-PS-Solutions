# Inter-IIT-Electrical-PS-Solutions
# Protocol Comparison Table

| Protocol | Wires Needed | Speed | Practical Distance | Number of Devices | Noise Immunity | Needed on Each End |
|----------|-------------|-------|---------------------|--------------------|-----------------|---------------------|
| **PWM** | 1 (+ GND) | N/A (duty-cycle based, typically 50 Hz–20 kHz) | A few meters | 1 transmitter → 1 receiver | Low (analog duty-cycle, sensitive to noise) | A PWM-capable timer/output pin on sender; a PWM-capable input/capture pin or filter+ADC on receiver |
| **UART** | 2 (TX/RX) | Up to ~1–3 Mbps (commonly 9.6k–115.2k) | Up to ~15 m (RS-232) or longer with drivers | 2 (point-to-point only) | Moderate | Matching baud rate/frame settings on both ends; UART peripheral (TX/RX pins) |
| **I2C** | 2 (SDA/SCL) | Up to 400 kHz (Fast Mode), 1–3.4 Mbps (Fast+/HS) | Up to ~1 m (limited by bus capacitance) | Many (7-bit: 112 usable addresses) | Low–Moderate (needs pull-ups, sensitive to long lines) | Pull-up resistors, unique address per device, I2C controller/peripheral |
| **SPI** | 4 (MOSI/MISO/SCLK/CS)sometimes 5 or 6 | Very high (10–50+ Mbps) | Short, <1 m typically (board-level) | Multiple (1 CS line per device) | Low (no built-in error checking) | SPI controller/peripheral, separate CS line per slave, matching clock polarity/phase |
| **CAN** | 2 (CAN_H/CAN_L, differential) | Up to 1 Mbps (Classical), up to 5–8 Mbps (CAN FD) | Up to ~40 m at 1 Mbps, 1000+ m at lower speeds | Many (theoretically unlimited, practically ~100+) | High (differential signaling, built for harsh environments) | CAN controller + transceiver, 120 Ω termination resistors at both bus ends |