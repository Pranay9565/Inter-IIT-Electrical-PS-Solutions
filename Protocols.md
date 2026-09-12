# Inter-IIT-Electrical-PS-Solutions
# 4th Problem Statement, Part 4
## Protocol Comparison Table:

| Protocol | Wires Needed | Speed | Practical Distance | Number of Devices | Noise Immunity | Needed on Each End |
|----------|-------------|-------|---------------------|--------------------|-----------------|---------------------|
| **PWM** | 1(Square wave) | Can be set to any frequency depending on hardware limit | For arduino, few meters | Only 1 device can be controlled | low | A PWM-capable timer/output pin on sender; a PWM-capable input/capture pin or filter+ADC on receiver |
| **UART** | 2 (TX/RX) | 450kbps, according to what i found on arduino forum | max 1 to 2 meters | only 2 devices | data corrupted easily | same baud rates should be on both the devices |
| **I2C** | 2 (SDA/SCL) | Up to 400 kHz (Fast Mode), 1–3.4 Mbps (Fast+/HS) | Up to ~1 m (limited by bus capacitance) | Many (7-bit: 112 usable addresses) | Low–Moderate (needs pull-ups, sensitive to long lines) | Pull-up resistors, unique address per device, I2C controller/peripheral |
| **SPI** | either 3 or 4 wire | Very high (10–50+ Mbps) | less than 1 m | Multiple (1 CS line per device) | Low (no built-in error checking) | SPI controller/peripheral, separate CS line per slave, matching clock polarity/phase |
| **CAN** | 2 (CAN_H/CAN_L, differential) | Up to 1 Mbps (Classical), up to 5–8 Mbps (CAN FD) | Up to ~40 m at 1 Mbps, 1000+ m at lower speeds | Many (theoretically unlimited, practically ~100+) | High (differential signaling, built for harsh environments) | CAN controller + transceiver, 120 Ω termination resistors at both bus ends |