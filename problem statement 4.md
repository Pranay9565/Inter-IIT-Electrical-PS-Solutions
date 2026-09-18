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
#
#
#
# Part 2, Choose the Bus

| # | Situation | Bus Chosen | Hardware on Each End | What Would Change the Pick |
|---|-----------|-----------|------------------------|------------------------------|
| 1 | IMU sitting 3 cm from an STM32 on the same board, read at 500 Hz. | **I2C** | Pull-up resistors on SDA/SCL, unique I2C address on IMU, I2C peripheral on STM32 | If sample rate needed to go much higher (multi-kHz) or multiple IMUs shared the bus with address conflicts, switch to **SPI** for guaranteed timing and no address limits |
| 2 | Four motor-controller nodes spread over a 1.5 m chassis, next to the power wiring, needing commands at 100 Hz and reporting back. | **CAN** | CAN controller + transceiver on each node, 120 Ω termination resistors at both physical ends of the bus | If it were just 1–2 nodes at short range with clean power (no noisy motor wiring nearby), **I2C** or **UART** could work instead — CAN's expense/complexity wouldn't be justified |
| 3 | An ESP32 sending telemetry to a Jetson over a 30 cm cable. | **UART** | TX↔RX cross-wired, common GND, matched baud rate/frame settings on both sides | If multiple devices needed to share the link, or distance/noise increased, switch to **CAN**; if much higher throughput was needed point-to-point, switch to **SPI** |
| 4 | A LiDAR streaming a few megabits per second continuously. | **UART (high baud)** or **SPI** if very short/on-board | High-baud UART transceiver + matched baud rate (most real LiDAR modules do this); or SPI controller/peripheral pair if the LiDAR is board-mounted | If the LiDAR were farther away or in a noisy environment, this would push toward a differential/framed link (or an interface outside this list, like USB/Ethernet, since UART/SPI have no built-in error checking at these speeds) |

# Part 3 wiring diagram
The data wires should be put in a twisted configuration which naturally cancels out noise or the wires can be wrapped in a metal foil which will block exteral electric field.
# Part 4 interfacing note
Arduino runs on 5v whereas stm32 and esp32 run on 3.3v, hence for stm32/esp32 to communicate with arduino we need a logic level shifter for arduino which converts 5v logic to 3.3v logic
