# 🗳️ SecureBallot — RFID-Based Electronic Voting System

### An Embedded Voting System Prototype Using ARM7 LPC2148 and Embedded C



---

## 📌 Overview

**SecureBallot** is an embedded electronic voting system prototype designed to demonstrate voter authentication, controlled election management, duplicate-vote prevention, and automated vote counting.

Built using the **NXP LPC2148 ARM7TDMI-S microcontroller** and Embedded C, the system integrates RFID authentication, a matrix keypad, an external I²C EEPROM, a real-time clock (RTC), and an LCD interface.

The system provides separate interfaces for **Election Officers and Voters**. Election Officers can manage voter records, configure election timings, view results, detect ties, and reset election data. Voters can authenticate using their RFID cards and PINs before accessing the ballot.

Non-volatile EEPROM storage maintains voter records, voting-status flags, credentials, and election data across power cycles.

> **Disclaimer:** SecureBallot is an academic embedded-systems prototype intended for educational purposes. It is not certified for real-world elections. Production voting systems require independent security audits, cryptographic protections, tamper resistance, accessibility testing, and regulatory compliance.

## ✨ Key Features

### 👮 Officer Interface — Administrative Mode

- 🔐 **Dual-Factor Authentication:** Officer RFID card and four-digit password verification.
- 👥 **Voter Management:** Add, deactivate, and reactivate voter records.
- ⏱️ **Election Scheduling:** Configure election start and stop times using the RTC.
- 📊 **Automated Results:** Calculate candidate or party vote counts.
- ⚖️ **Tie Detection:** Detect ties among candidates with the highest vote count.
- 🧹 **Election Reset:** Reset vote counts and voter voting-status flags.
- 🔑 **Password Management:** Change the administrative password.

### 🗳️ Voter Interface — Polling Mode

- 🪪 **RFID Authentication:** Identify voters using the EM-18 RFID reader.
- 🛑 **Duplicate-Vote Prevention:** Reject voters whose records indicate that they have already voted.
- 🔒 **PIN Verification:** Require a four-digit voter PIN before ballot access.
- 🗳️ **Multiple Candidates:** Support selection among up to eight registered parties or candidates.
- ✅ **Vote Confirmation:** Confirm the selected candidate before saving the vote.
- 🔑 **PIN Management:** Allow authenticated voters to change their PIN.

### ⚙️ Embedded Systems Integration

- 📡 Dual UART communication for RFID input and serial debugging.
- 💾 I²C communication with AT24C256 external EEPROM.
- ⏰ RTC-based election scheduling.
- 🖥️ 20×4 LCD menu and status display.
- 🔢 4×4 matrix keypad for PIN entry and navigation.
- 🔄 Modular Embedded C drivers and state-based application logic.

## 🏗️ System Architecture

```text
                     +--------------------------+
                     |     NXP LPC2148           |
                     |   ARM7TDMI-S Controller   |
                     +------------+-------------+
                                  |
              +-------------------+-------------------+
              |                   |                   |
          UART1 / RX          GPIO / LCD           I2C0
              |                   |                   |
       +-------------+      +-----------+       +------------+
       | EM-18 RFID  |      | 20x4 LCD  |       | AT24C256   |
       | Reader      |      +-----------+       | EEPROM     |
       +-------------+            |             +------------+
                            +-------------+
                            | 4x4 Keypad  |
                            +-------------+

                     Additional Interfaces
                 +-----------------------------+
                 | RTC: Election Time Control  |
                 | UART0: Debugging / Logging  |
                 | Buzzer / LEDs: Status Alert  |
                 +-----------------------------+
```

### 🔌 Hardware Components

| No. | Component | Model / Specification | Purpose |
|---|---|---|---|
| 1 | Microcontroller | NXP LPC2148, ARM7TDMI-S | Main processing and peripheral control |
| 2 | RFID Reader | EM-18 | RFID card identification |
| 3 | RFID Cards | Compatible RFID tags | Officer and voter identification |
| 4 | External EEPROM | AT24C256, I²C | Persistent record and vote storage |
| 5 | LCD | 20×4 character LCD | Menus, prompts, and election results |
| 6 | Keypad | 4×4 matrix keypad | PIN entry and menu navigation |
| 7 | RTC | On-chip RTC or external DS1307 | Election scheduling |
| 8 | Serial Interface | USB-to-UART converter | Debugging and event logging |
| 9 | Indicators | Buzzer / LEDs | Status and error indication |

## 📍 Pin Configuration

The following is the proposed pin mapping. Verify it against your actual schematic and source code before connecting the hardware.

| Peripheral | LPC2148 Pin | Function |
|---|---|---|
| UART0 | P0.0 / TXD0 | Serial logging output |
| UART0 | P0.1 / RXD0 | Serial input, if required |
| UART1 | P0.8 / TXD1 | UART1 transmit |
| UART1 | P0.9 / RXD1 | RFID data reception |
| I²C0 | P0.2 / SCL0 | EEPROM serial clock |
| I²C0 | P0.3 / SDA0 | EEPROM serial data |
| LCD | P0.8–P0.15 | Proposed 8-bit LCD data bus |
| LCD | P0.16 | Register Select (RS) |
| LCD | P0.17 | Enable (EN) |
| Keypad | P1.16–P1.19 | Row connections |
| Keypad | P1.20–P1.23 | Column connections |

**Hardware note:** The proposed LCD data mapping overlaps with UART1 pins P0.8 and P0.9. These peripherals cannot independently use the same pins in this configuration. Review the pin assignments and adjust the LCD connections or peripheral allocation before hardware implementation.

## 🔐 Security Logic and Voter Authentication

The voter authentication process follows these steps:

1. The RFID reader scans a voter's card.
2. The microcontroller receives the card ID through UART1.
3. The system searches the voter's record in EEPROM.
4. The active-status flag is checked.
5. The voting-status flag is checked.
6. The voter enters a four-digit PIN.
7. The system verifies the PIN.
8. The authenticated voter selects a candidate or party.
9. The voter confirms the selection.
10. The vote count and voter status are updated in EEPROM.

### Voter Authentication Decision Table

| Condition | System Response |
|---|---|
| RFID card is not registered | Reject access |
| Active flag = `0x00` | Display `VOTER INACTIVE` |
| Active flag = `0x01` | Continue authentication |
| Voted flag = `0x01` | Display `ALREADY VOTED` |
| Voted flag = `0x00` | Request voter PIN |
| PIN is incorrect | Reject ballot access |
| PIN is correct | Grant ballot access |
| Vote is confirmed | Update vote data and voter status |

**Status flags:**

- `0x00` — No / inactive, depending on the field.
- `0x01` — Yes / active, depending on the field.

The interpretation depends on the specific EEPROM field.

## 🗃️ EEPROM Memory Mapping

The AT24C256 external EEPROM provides non-volatile storage for voter information and election data.

### Voter Record Layout

Each voter record uses a 20-byte structure.

| Offset | Size | Stored Information |
|---|---:|---|
| `+0x00` | 10 bytes | RFID card ID |
| `+0x0A` | 4 bytes | Voter PIN |
| `+0x0E` | 1 byte | Voting-status flag |
| `+0x0F` | 1 byte | Active-status flag |
| `+0x10` | 4 bytes | Reserved metadata |

The exact EEPROM addresses for officer credentials, election schedules, candidate counts, and other global configuration data should be defined separately in the project configuration headers.

> **Security consideration:** This prototype's described record format stores PINs directly. A production design should use appropriate password hashing or keyed verification, access controls, protected audit logs, and a carefully designed write-validation strategy.

## 🧭 System Workflow

### 👮 Officer Workflow

```text
Start System
     |
     v
Scan Officer RFID Card
     |
     v
Verify Officer Identity
     |
     v
Enter Officer Password
     |
     v
Validate Credentials
     |
     v
Open Officer Menu
     |
     +---- Manage Voters
     |
     +---- Configure Election Time
     |
     +---- View Results / Detect Ties
     |
     +---- Reset Election Data
     |
     +---- Change Officer Password
```

### 🗳️ Voter Workflow

```text
Scan Voter RFID Card
          |
          v
Search Voter Record
          |
          v
Check Active Status
          |
          v
Check Voting Status
          |
          v
Verify Voter PIN
          |
          v
Display Candidate Menu
          |
          v
Select Candidate / Party
          |
          v
Confirm Selection
          |
          v
Update Vote Count
          |
          v
Mark Voter as Voted
```

The actual implementation should validate the order of EEPROM writes and handle interrupted writes so a power failure cannot leave vote counts and voter status inconsistent.

## 💻 Software and Development Tools

| Tool / Technology | Purpose |
|---|---|
| Programming Language | Embedded C |
| Microcontroller | NXP LPC2148 / ARM7TDMI-S |
| IDE | Keil µVision 4 / 5 |
| Compiler | Keil ARM Compiler |
| Simulation | Proteus VSM, if supported by the project model |
| Flashing Utility | Flash Magic |
| Debugging | UART serial terminal |
| Communication Protocols | UART, I²C, GPIO |

## 📁 Project Structure

```text
SecureBallot-System/
│
├── inc/
│   ├── types.h
│   ├── defines.h
│   ├── uart1.h
│   ├── uart0.h
│   ├── i2c.h
│   ├── i2c_eeprom.h
│   ├── lcd.h
│   ├── kpm.h
│   ├── password.h
│   └── data_location_defines.h
│
├── src/
│   ├── uart1.c
│   ├── uart0.c
│   ├── i2c.c
│   ├── i2c_eeprom.c
│   ├── lcd.c
│   ├── kpm.c
│   └── password.c
│
├── apps/
│   ├── Officer_interface.c
│   ├── Voter_interface.c
│   └── main.c
│
├── images/
│   ├── hardware-setup.jpg
│   └── proteus-simulation.png
│
├── Makefile
├── README.md
└── SecureBallot.uvproj
```

*Note: This structure represents the intended organization. Keep only the files and directories that actually exist in your repository, and use the correct Keil project filename.*

## 🧪 Suggested Test Cases

| Test Case | Test Procedure | Expected Result |
|---|---|---|
| Officer Authentication | Scan valid and invalid officer cards | Only valid credentials grant access |
| Voter Authentication | Scan a registered voter card | Request the correct voter PIN |
| Duplicate Voting | Attempt to vote twice using one card | Second attempt is rejected |
| Inactive Voter | Deactivate a voter and scan the card | Access is rejected |
| Incorrect PIN | Enter an invalid PIN | Ballot access is denied |
| Vote Counting | Cast votes for multiple candidates | Counts update correctly |
| Tie Detection | Create equal leading vote counts | Tie is reported |
| Election Timing | Attempt voting outside the configured period | Voting is rejected, if implemented |
| Power-Cycle Persistence | Restart the system after casting votes | Stored records and counts are retained |

## 📷 Screenshots and Hardware Setup

Add your actual project photos and simulation screenshots to the `images/` directory.

### Hardware Setup

![SecureBallot Hardware Setup](images/hardware-setup.jpg)

### Proteus Simulation

![SecureBallot Proteus Simulation](images/proteus-simulation.png)

If you have not added these image files yet, upload them to the repository or remove the corresponding image references until they are available.

## 🔮 Future Enhancements

- 🔐 Improve credential protection and authentication security.
- 🧾 Add a tamper-evident audit trail with RTC timestamps.
- 🔋 Improve power-failure recovery and EEPROM write consistency.
- 🌐 Add an optional ESP32-based monitoring dashboard.
- 🛡️ Add stronger input validation and fault handling.
- 🧪 Expand automated testing for communication errors and interrupted writes.
- 📋 Document threat models, hardware limitations, and security assumptions.

## 🎓 Learning Outcomes

Through this project, the following embedded-systems concepts can be demonstrated:

- ARM7 LPC2148 peripheral configuration and register-level programming.
- UART communication and RFID data acquisition.
- I²C protocol implementation and EEPROM interfacing.
- LCD and matrix-keypad interfacing.
- RTC-based scheduling and time-dependent control logic.
- Non-volatile data storage and record management.
- State-machine design for multi-role embedded applications.
- Modular Embedded C development, debugging, and testing.

## 👨‍💻 Author

**Sachin Ghule**

Embedded Systems | ARM7 | Embedded C | Microcontroller Interfacing


---

<p align="center">
  Built with ❤️ using ARM7 LPC2148 and Embedded C
</p>
