# Fault-tolerant-Uart-telemetry-Using-ATmega328P
Dual-ATmega328P fault-tolerant UART telemetry simulator in Proteus 8 with CRC-8 error detection and ACK/NAK retries.

Fault-Tolerant UART Telemetry & CRC-8 SimulatorA dual-microcontroller (ATmega328P @ 8 MHz) embedded communication system simulated in Proteus 8, featuring structured binary packet framing, CRC-8 error detection, ACK/NAK automatic retries, and live hardware fault injection.

Why This Project?
In real-world embedded systems (automotive, robotics, industrial sensors), electrical noise on serial lines can corrupt sensor readings. Instead of transmitting raw text, this project implements a reliable binary protocol in bare-metal Embedded C that detects corrupted frames and prevents invalid data from reaching the display.

System ArchitectureSender (U1 — ATmega328P):
Samples an analog potentiometer (POT-HG) via 10-bit ADC0 (PC0), packs the reading into a 5-byte struct, computes a CRC-8 checksum, and transmits over UART at 9600 bps. Retries up to 3 times on NAK (0x15).
Hardware Fault Injector: A LOGICSTATE toggle gates a 500 Hz DCLOCK through an AND + XOR stage on the U1 TX -> U2 RX line to inject deterministic bit-flip noise mid-packet.Receiver (U2 — ATmega328P): Synchronizes on 0xAA, reads the 5-byte frame with byte-level timeouts, recomputes CRC-8, and updates a 16x2 LCD (LM016L), Green/Red status LEDs, and Proteus Virtual Terminal logs.5-Byte Telemetry Packet FormatByteFieldValue / Purpose0start_byte0xAA — Frame synchronization marker1node_id0x01 — 

Sender node identifier2data_highUpper bits of 10-bit ADC reading3data_lowLower 8 bits of 10-bit ADC reading4crc8CRC-8 (0x07 polynomial) computed over bytes 1–3Demo ScreenshotsAdd two screenshots from your Proteus simulation inside a docs/ folder:Normal Operation (LOGICSTATE = 0): docs/normal_mode.png — Green LED ON, LCD showing live ADC value and Status: OK.Fault Injection (LOGICSTATE = 1): docs/fault_mode.png — Red LED ON, LCD showing CRC ERR without overwriting the last valid reading, Virtual Terminal logging [ERR] Bad CRC!.Repository Structurefault-tolerant-uart-telemetry/

├── firmware/
│   ├── sender.c          # U1 firmware (ADC, packet framing, CRC-8, retry logic)
│   └── receiver.c        # U2 firmware (UART RX timeout, CRC-8 check, 4-bit LCD driver)
├── proteus/
│   └── Fault_tolerant_simulator.pdsprj
├── docs/
│   ├── normal_mode.png
│   └── fault_mode.png
└── README.md

How to Run in Proteus 8Open proteus/Fault_tolerant_simulator.pdsprj in Proteus 8 Professional
.Ensure both U1 and U2 are configured with Internal RC Osc. 8 MHz, CLKDIV8 Unprogrammed, and WinAVR (AVR-GCC) firmware builds.Click Play (▶) with LOGICSTATE = 0 to observe normal telemetry, adjust RV1 (POT-HG) to vary the sensor value, and toggle LOGICSTATE = 1 to test live CRC error detection.
