## RFID Unified citizen service system

![Microcontroller](https://img.shields.io/badge/Microcontroller-LPC2148-blue.svg)
![Language](https://img.shields.io/badge/Language-Embedded%20C-orange.svg)
![IDE](https://img.shields.io/badge/IDE-Keil%20uVision-green.svg)
![Status](https://img.shields.io/badge/Status-Complete-success.svg)

An advanced embedded system project built on the **LPC2148 (ARM7TDMI-S)** microcontroller. This project implements a comprehensive **Unified Citizen System**, integrating critical utility databases and financial services into a single multi-purpose RFID smart card interface. 

By scanning a unique citizen RFID card, authenticated users can access their Banking (ATM), Voting, Driving License status, and PAN details seamlessly through an interactive LCD and Keypad interface.

---

- **RFID authentication** — valid citizen, officer, and unrecognized card flows with LCD, LED, buzzer, and UART feedback.
- **Citizen dashboard** — PAN-style details, ATM balance/withdrawal/deposit flow, voting status, and driving-licence information.
- **Officer controls** — voting reset, RTC adjustment, and driving-licence-expiry maintenance.
- **Persistent data** — account balances, voting flags, and PINs are retained in a 25LC512-compatible SPI EEPROM.
- **Responsive UI** — keypad input uses debounce handling and timeout-aware scanning; RFID reception is interrupt-driven.

## Hardware architecture
The overall system layout connects the main controller block to various sensors, displays, inputs, and memory storage.
<p align="center">
 <img width="1536" height="1024" alt="new_block_diagram" src="https://github.com/user-attachments/assets/93e0912b-75d0-496d-9ebc-911945c95ba4" />

| Block | Role |
| --- | --- |
| LPC21xx ARM7 MCU | Runs the embedded-C firmware, UI, authentication, and peripheral control. |
| RFID reader | Sends card frames to UART0 at 9,600 baud. |
| 4×4 keypad | Inputs menu options, PINs, amounts, and date/time values. |
| 20×4 HD44780 LCD | Displays prompts, authentication status, menus, and citizen records. |
| 25LC512 EEPROM | Preserves balances, votes, and PIN data across power cycles. |
| LEDs and buzzer | Give immediate success, failure, and officer-access feedback. |
| RTC | Maintains date and time, with officer-controlled editing. |

## System Architecture
The codebase is structured modularly to separate the low-level peripheral drivers (UART, SPI, Keypad, LCD, RTC) from the high-level application menus.
<p align="center">
 <img width="1536" height="1024" alt="software rfid image" src="https://github.com/user-attachments/assets/3b6ebbd8-ca0d-414b-9658-1aa57fa1afda" />
</p>

## Firmware Modules and Responsibilites 
Each C file is compiled and linked with specific functional responsibilities to form the unified binary:
<img width="1536" height="1024" alt="Modules menu" src="https://github.com/user-attachments/assets/c19456e9-f42f-4cae-b396-5cd5ab32e9eb" />


## Demonstrated workflow

<p align="center">
 <img width="1600" height="878" alt="WhatsApp Image 2026-09-09 at 10 35 02 AM" src="https://github.com/user-attachments/assets/e56b7281-3ca6-4d62-adc6-b6fe4c506c53" />
 <img width="1600" height="858" alt="WhatsApp Image 2026-09-09 at 10 37 04 AM" src="https://github.com/user-attachments/assets/fa709ad2-a9fd-4413-9e81-31f21835876a" />
</p>

1. Power on the system; it shows the RFID scan prompt.
2. Present a registered citizen card to open the citizen dashboard.
3. Use the keypad to view records or use the ATM, voting, and licence services.
4. Present the officer card for administrative controls.
5. Unknown cards are rejected with the red LED and buzzer.
## LCD interface
<p align="center">
  <img width="1268" height="772" alt="citizen_service_menu" src="https://github.com/user-attachments/assets/4c234ab6-8281-4124-9c2c-67e096568098" />
  <img width="1599" height="920" alt="Pan" src="https://github.com/user-attachments/assets/ee1c4891-aed1-42ca-906c-1462aaa80ebf" />
  <img width="1280" height="766" alt="ATM_interfcae" src="https://github.com/user-attachments/assets/8e3e2b73-26e0-4027-87b3-6b985dbfd923" />
  <img width="1280" height="785" alt="officer_status" src="https://github.com/user-attachments/assets/c1493aa2-5867-4c3b-ba65-00fe2aa5fd16" />

  <img width="1280" height="760" alt="RTC" src="https://github.com/user-attachments/assets/d2266567-1daf-4a95-9d46-713b00b89614" />

</p>

## ✨ System Features in Detail

The system relies on an external **AT25LC512 (512Kbit / 64KB)** EEPROM over SPI0 to maintain persistent user states.
### 1. 🪪 Universal RFID Authentication & Card Security
*   **Dual Roles:** Distinguishes between standard citizens (cards registered in parallel arrays) and system administrators (Officer Master Card).
*   **Loss Prevention / Blocking Mechanism:** The Officer Menu allows administrators to select any citizen by index and mark their status as `BLOCKED` (stores `0x01` at their EEPROM index). If a citizen's physical card is stolen/lost and scanned afterwards, access is immediately blocked, trigger outputs (Red LED and continuous Buzzer) are activated, and the incident is logged via serial telemetry.
*   **Inactivity & Redraw Resiliency:** Protects user sessions with a 20-second inactivity timeout for all keypad input operations, returning to the login loop if abandoned.

### 2. 🏦 Secure ATM (Automated Teller Machine) Module
*   **Encrypted Storage:** Balances and individual ATM PINs are stored securely in external non-volatile memory.
*   **Transactional Guards:**
    *   Restricts withdrawals to multiples of `100`, `200`, and `500` rupees.
    *   Enforces a minimum balance limit (`Rs. 500`) and a maximum storage boundary (`Rs. 65,535` based on 16-bit uint representation).
    *   Authenticates with a distinct ATM-only PIN before loading the transaction screen.

### 3. 🗳️ Double-Voting Prevention System
*   **Authentication Check:** Asks for login verification before allowing voting access.
*   **Non-Volatile Registry:** Once a citizen casts their vote for any of the 4 configured parties (**BJP, INC, AAP, BSP**), the vote state is written to EEPROM. If the user tries to access the voting screen again, the system queries the EEPROM and immediately blocks the operation with an "Already Voted!" error message.

### 4. 🚗 Driving License Expiry Engine (RTC & CGRAM)
*   **Real-time Expiry Calculation:** Automatically compares the expiration date parameters (`exp_days`, `exp_months`, `exp_years` stored per user) against the running internal Real-Time Clock (RTC) registers.
*   **Hardware Signaling:** 
    *   **Valid License:** Displays license details, illuminates the Green LED, and silences any buzzer alarms.
    *   **Expired License:** Flashes a custom-built CGRAM "Bold Cross (✖)" icon on the LCD, turns on the Red LED, and sounds a warning Buzzer.
*   **Custom Graphics:** Built-in CGRAM design templates inject custom validation marks directly into the HD44780 LCD module memory:
  
---
## 🛠️ Low-Level Drivers & Implementation

### 1. UART0 (RFID Transceiver Driver)
*   **Baud Rate & Frame Configurations:** Runs at a baud rate of `9600` configured for an ARM core peripheral clock (Pclk) of `15MHz` (`U0DLL = 97`, `U0DLM = 0`, `8N1` Mode).
*   **Asynchronous Interrupt Handler:** Leverages the LPC2148 UART0 RX line interrupt (`UART0_ISR` mapped in VIC slot 5) to intercept RFID frames on the fly without blocking execution. 
*   **Framing Delimiters:** Incoming RFID streams are parsed between the standard Serial Start-of-Text (`STX = 0x02`) and End-of-Text (`ETX = 0x03`) bytes to ensure packet transmission integrity.
*   
### 2. Interrupt Management
The Vectored Interrupt Controller (VIC) maps incoming hardware triggers efficiently (e.g., UART0 and EINT3)

### 3. LCD Driver (8-Bit Mode)
*   **Layout:** Operates in 8-bit bus configuration utilizing pins `P0.8 - P0.15` for data and control pins `P0.16` (RS), `P0.17` (R/W), and `P0.18` (EN).
*   **CGRAM Interface:** Contains functions to reprogram the internal CGRAM tables of the LCD display on-the-fly, allowing graphics manipulation of custom display metrics.

### 4. SPI0 Engine & Calender
*   **SPI0 Init:** Standard 8-bit write-only/read SPI routines mapped directly to hardware peripheral registers (`S0SPCR`, `S0SPSR`, `S0SPDR`).
*   **Calender:** Integrated mathematically to keep the RTC day register (`DOW`) calculated dynamically when administrative edits occur:



## Wiring reference

This table is derived from the firmware in this repository—not copied from the reference project. Verify every connection against the labels on your specific development board before applying power.

| Peripheral | LPC21xx pins / interface | Firmware location |
| --- | --- | --- |
| RFID reader | UART0: **P0.0 / TxD0**, **P0.1 / RxD0**, 9,600 baud | `uart.c` |
| SPI EEPROM | SPI0: **P0.4 / SCK0**, **P0.5 / MISO0**, **P0.6 / MOSI0**; **P0.7** chip select | `spi.c`, `spi_eeprom.c` |
| LCD data bus | **P0.8–P0.15** (D0–D7) | `lcd_defines.h` |
| LCD control | **P0.16 / RS**, **P0.17 / RW**, **P0.18 / EN** | `lcd_defines.h` |
| Status outputs | **P0.19** buzzer, **P0.20** red LED, **P0.21** green LED | `Rfid_test.c` |
| 4×4 keypad | **P1.16–P1.19** rows, **P1.20–P1.23** columns | `kpm_defines.h` |
| Officer/RTC-edit interrupt | **P0.30 / EINT3**, falling edge | `Rfid_test.c` |

> **Configured target:** this repository is configured for the **NXP LPC2148** (ARM7TDMI, 512 KB flash, 32 KB RAM), matching the photographed development board. Before flashing, still confirm that your board carries an LPC2148 and that its ISP/programmer settings match your hardware.

## Build and flash

### Requirements

- Keil µVision / MDK with ARM7 and LPC21xx device support
- An LPC21xx board, RFID reader, 20×4 HD44780-compatible LCD, 4×4 matrix keypad, SPI EEPROM, LEDs, and buzzer
- A compatible programmer or the board's ISP bootloader connection

### Steps

1. Clone or download this repository.
2. Open `RFID_PROJECT.uvproj` in Keil µVision.
3. Confirm that **Target 1** is set to LPC2148.
4. Confirm the connections in the wiring table, then build **Target 1**.
5. Flash the generated `RFID_PROJECT.hex` using your board's supported programming method.
6. Power the hardware and test a known citizen or officer card.

## Source layout

| File | Responsibility |
| --- | --- |
| `Rfid_test.c` | Application entry point, initialisation, RFID authentication, indicators, and interrupt setup. |
| `user_menu.c` | Citizen/officer menus, PIN flow, voting, ATM operations, driving-licence data, and RTC editing. |
| `uart.c` | UART0 setup and interrupt-driven RFID frame reception. |
| `spi.c`, `spi_eeprom.c` | SPI0 transport and 25LC512 EEPROM read/write operations. |
| `lcd.c` | HD44780 LCD driver. |
| `keypad.c` | 4×4 keypad scanning and debounce handling. |
| `delay.c` | Timing helpers. |

## Important notes

- The sample card IDs, names, balances, and PIN values are demonstration data. Replace them before any real deployment.
- This project is appropriate for education and laboratory demonstration. It is **not** a production-grade identity, banking, or voting system: card IDs and PIN storage need stronger security for real-world use.
- Do not commit generated Keil binaries or per-user IDE settings. The included `.gitignore` already excludes them.
