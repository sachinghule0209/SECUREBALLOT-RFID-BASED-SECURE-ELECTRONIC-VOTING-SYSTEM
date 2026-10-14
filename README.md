🗳️️ SecureBallot : RFID-Based Electronic Voting System
An embedded voting system prototype engineered with ARM7 LPC2148 and Embedded c

Platform Protocol Language IDE Status

📌 Overview
SecureBallot is a real-time embedded application engineered to secure, automate, and streamline electoral operations. Built on the NXP LPC2148 ARM7TDMI-S microcontroller, it integrates RFID card authentication, matrix keypad password protection, I²C external EEPROM storage, an RTC clock, and an LCD dashboard interface to manage everything from voter registration to real-time vote tallying and tie detection.

The system enforces strict multi-role separation between Election Officers (admin operational mode) and Voters (polling mode), ensuring one-vote-per-voter enforcement backed by non-volatile storage.

💡 Disclaimer: SecureBallot is an academic/embedded-systems prototype designed to demonstrate multi-peripheral interfacing, state machines, and non-volatile data management. Real elections require independent security assessments, tamper resistance, and formal regulatory compliance.

✨ Features
👮 Officer Interface (Administrative Mode)
🔐 Dual-Factor Authentication: Verified via Officer RFID card and a 4-digit security password.
👥 Voter Management: Add new voter records, remove existing voters, or reactivate previously deactivated voters.
⏱️ Election Schedule Control: Set exact election start and stop times using the internal RTC.
📊 Result Tallying & Tie Detection: Automatically computes winner party counts and detects ties among leading candidates.
🧹 System Reset: Clear all cast votes and reset voter status flags for a new election cycle.
🔑 Credential Management: Change administrative officer password stored in persistent EEPROM.
🗳️ Voter Interface (Polling Mode)
🪪 RFID Identification: Instant voter verification via EM-18 RFID reader on UART.
🛑 Anti-Double Voting Guard: Checks EEPROM active status flags to prevent duplicate votes.
🔐 Password Verification: 4-digit voter PIN authentication prior to ballot access.
🗳️ Multi-Party Ballot: Supports selection across up to 8 registered parties/candidates.
✅ Vote Confirmation: Interactive menu prompt to confirm candidate selection before committing to EEPROM.
🔑 PIN Management: Change voter PIN securely after authentication.
⚙️ Embedded Systems Integration
📡 UART Dual-Port Operations: UART1 for RFID card scanning; UART0 for event logging & telemetry.
💾 I²C Serial EEPROM Interface: High-endurance AT24C256 communication for non-volatile record storage.
⏱️ RTC Time Management: Real-time clock syncing for automated election start/stop windows.
🚦 System Security & State Logic

Voter Authentication State                 System Action & Response
──────────────────────────────────────     ──────────────────────────────────────────────────
RFID Card Scanned                     →    Read 10-Byte Card ID via UART1
Card Found in EEPROM                  →    Check Active Flag (+0x0F)
Active Flag == 0x00 (Inactive)        →    🔴 Display "VOTER INACTIVE" & Reject
Active Flag == 0x01 (Active)          →    Check Voted Flag (+0x0E)
Voted Flag == 0x01 (Already Voted)    →    🔴 Display "ALREADY VOTED" & Reject
Voted Flag == 0x00 (Not Voted)        →    🟡 Prompt for 4-Digit Voter PIN
PIN Correct                           →    🟢 Grant Ballot Access → Record Vote → Mark Flag

🏗️ System Architecture

                +---------------------------------------+
                |      NXP LPC2148 Microcontroller      |
                |            (ARM7TDMI-S Core)          |
                +---+-------+-------+-------+-------+---+
                    |       |       |       |       |
+---------------------+ | | | +---------------------+ | (UART1) | | | (UART0) | v v v v v


+-----------+                 +---------+ +---+ +-------+                   +-----------+
| EM-18     |                 |  20x4   | |4x4| | RTC   |                   |  UART0    |
| RFID      |                 | Character | |Key| |Clock  |                   | System    |
| Reader    |                 |  LCD    | |pad| |Module |                   | Event Log |
+-----------+                 +---------+ +---+ +-------+                   +-----------+
|
| (I²C Bus)
v
+-----------+
| AT24C256  |
| External  |
| I²C EEPROM|
+-----------+

🔌 Hardware Components
#	Component	Model / Spec	Purpose
1	Microcontroller	NXP LPC2148 (ARM7TDMI-S, 60 MHz)	Core processing and peripheral orchestration
2	RFID Reader	EM-18 RFID Module (9600 Baud, UART)	Officer and voter RFID tag scanning
3	Storage	AT24C256 Serial I²C EEPROM	Persistent storage for records, flags, and vote counts
4	User Display	20×4 Character LCD (HD44780 Driver)	Interactive menus, status prompts, and result display
5	Input Interface	4×4 Matrix Keypad	Password entry, menu navigation, and candidate selection
6	Real-Time Clock	LPC2148 On-Chip RTC / External DS1307	Election timing, start/stop logic
7	Serial Interface	USB-to-UART Bridge (UART0)	Real-time system logging and event debugging
📍 Pin Configuration
GPIO & Communication Peripherals
Peripheral	LPC2148 Pin	Function / Target Line
UART1 (RFID)	P0.8 / RxD1	Connected to EM-18 TX pin
UART0 (Logging)	P0.0 / TxD0	Serial terminal output (9600 baud)
I²C0 (EEPROM)	P0.2 / SCL0	Serial Clock for AT24C256
P0.3 / SDA0	Serial Data for AT24C256
20x4 LCD	P0.8 – P0.15	LCD 8-bit Data Bus (D0–D7)
P0.16	LCD Register Select (RS)
P0.17	LCD Enable (EN)
4x4 Keypad	P1.16 – P1.19	Keypad Row Outputs (Row 0 – Row 3)
P1.20 – P1.23	Keypad Column Inputs (Col 0 – Col 3)
🗃️ EEPROM Memory Mapping
The external AT24C256 EEPROM stores administrative credentials, voter records, election scheduling, and vote counts. Voter records follow a strict 20-byte record layout:


Record Offset        Size (Bytes)        Stored Information
────────────────     ────────────        ───────────────────────────────────────
+0x00                10 Bytes            Voter RFID Tag ID
+0x0A                 4 Bytes            Voter 4-Digit Password PIN
+0x0E                 1 Byte             Voting-Status Flag (0x00: No, 0x01: Voted)
+0x0F                 1 Byte             Active Status Flag (0x01: Active, 0x00: Removed)
+0x10                 4 Bytes            Reserved / Future Metadata

📁 Project Structure

SecureBallot-System/
│
├── inc/                          ← Header files & driver APIs
│   ├── types.h                   ← Custom data type definitions (u8, u16, u32)
│   ├── defines.h                 ← Bitwise manipulation macros
│   ├── uart1.h                   ← RFID UART1 communication interface
│   ├── uart0.h                   ← Logging & debugging UART0 interface
│   ├── i2c.h                     ← I²C protocol driver API
│   ├── i2c_eeprom.h              ← AT24C256 EEPROM read/write routines
│   ├── lcd.h                     ← 20x4 LCD display driver API
│   ├── kpm.h                     ← 4x4 Matrix keypad scanner API
│   ├── password.h                ← Password entry & masked verification API
│   └── data_location_defines.h   ← EEPROM address offsets and status flags
│
├── src/                          ← Peripheral driver implementations
│   ├── uart1.c                   ← RFID packet reception routines
│   ├── uart0.c                   ← Serial log transmitter
│   ├── i2c.c                     ← I²C bus start/stop/ack primitives
│   ├── i2c_eeprom.c              ← Byte and page read/write routines
│   ├── lcd.c                     ← HD44780 LCD control routines
│   ├── kpm.c                     ← Keypad scan & debounce logic
│   └── password.c                ← Masked PIN handling & verification
│
├── apps/                         ← User & Administrative Interfaces
│   ├── Officer_interface.c       ← Administrative menu, timing, & tallying
│   ├── Voter_interface.c         ← Voter verification, voting menu, & flag updates
│   └── main.c                    ← System initialization & main event loop
│
└── Makefile / .uvproj            ← Keil µVision Build Configuration

🧭 System Workflow
WhatsApp Image 2026-10-03 at 12 50 18 AM
👮 Officer Workflow

Officer RFID Card
│
v
Receive RFID Data (UART1)
│
v
Verify Officer RFID
│
v
Enter Officer Password
│
v
Officer Menu
│        │        │         │         │
v        v        v         v         v
Manage   Set Time  View     Reset     Change
Voters            Result   Election  Password

🗳️ Voter Workflow

Voter RFID Card
│
v
Receive and Identify Card
│
v
Find Voter Record in EEPROM
│
v
Check Active Status
│
v
Check Voting Status
│
v
Enter Voter Password
│
v
Voter Menu
│
v
Select Party/Candidate
│
v
Confirm Vote
│
v
Increment Party Count in EEPROM
│
v
Mark Voter as Already Voted

💻 Software and Tools
Tool / Software	Purpose / Details
IDE	Keil µVision 4 / 5 (ARM-MDK)
Compiler	Keil ARM C Compiler
Microcontroller	NXP LPC2148 (ARM7TDMI-S @ 60 MHz)
Simulation Tool	Proteus VSM (Simulation & Circuit Verification)
Flash Tool	Flash Magic (UART0 ISP Flashing)
Language	Embedded C
🧪 Suggested Test Cases
✔️ Officer Access: Test authentication with valid vs. invalid Officer RFID cards and passwords.
✔️ Voter Duplicate Guard: Cast a vote for a registered user and attempt to scan the same RFID card again.
✔️ Voter Removal: Deactivate a voter record through the Officer menu and attempt to vote with that card.
✔️ Incorrect Credentials: Enter incorrect PINs during voter/officer authentication and verify lock-out behavior.
✔️ Tally & Tie Detection: Cast equal votes for two candidates and inspect the tie detection output on the Officer results screen.
✔️ Power Cycle Persistence: Disconnect power mid-election and verify that vote counts and voter status flags persist in EEPROM upon reboot.
📷 System Screenshots & Hardware Setup
![SecureBallot Hardware Setup](images/hardware-setup.jpg)
![SecureBallot Proteus Simulation](images/proteus-simulation.png)
(Place your images in an images/ directory within the repository).

🔮 Future Improvements
🔐 Cryptographic PIN Storage: Hash voter PINs in EEPROM rather than storing plaintext values.
🧾 Audit Trail: Log all voter events with RTC timestamps to an encrypted transaction log.
🔋 Power Loss Resilience: Implement atomic double-buffered EEPROM writes to prevent data corruption during mid-write power loss.
🌐 Wireless Dashboard: Integrate an ESP32 bridge over UART to broadcast live election status to a remote monitoring site.
🎓 Learning Outcomes
🛠️ ARM7 LPC2148 hardware peripheral configuration (UART, I²C, RTC, GPIO).
💾 Low-level I²C protocol handling for external serial EEPROM read/write routines.
📊 Multi-role state machine design for secure embedded applications.
📡 Serial interrupt-driven data acquisition via RFID readers.
🖥️ Modular Embedded C system development and debugging.
👨‍💻 Author
Sachin Ghule

Embedded Systems | ARM7 | Embedded C | Microcontroller Interfacing
Built with ❤️ on ARM7 | LPC2148 | Embedded C
