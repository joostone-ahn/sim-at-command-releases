# SIM AT Command Tool — User Guide

> Version: v1.0.0  
> Last updated: 2026-05-12

---

## Table of Contents

1. [Overview](#1-overview)
2. [Screen Layout](#2-screen-layout)
3. [Device Setup](#3-device-setup)
4. [Connecting to a Device](#4-connecting-to-a-device)
5. [Browsing SIM Files](#5-browsing-sim-files)
6. [Reading Files](#6-reading-files)
7. [Decoded Views](#7-decoded-views)
8. [APDU Log](#8-apdu-log)
9. [ADM Verification](#9-adm-verification)
10. [Writing to Files](#10-writing-to-files)
11. [BER-TLV Files (URSP)](#11-ber-tlv-files-ursp)
12. [Modem Compatibility](#12-modem-compatibility)
13. [Troubleshooting](#13-troubleshooting)

---

## 1. Overview

SIM AT Command Tool is a web-based application for reading, writing, and decoding SIM/USIM/ISIM card files via AT+CSIM (raw APDU) over USB modem port. Run the exe file and it opens automatically in your browser.

**Key capabilities:**
- Read and auto-decode 100+ EF files
- Write to EF files with ARR-based access control enforcement
- BER-TLV tag-based read/write (URSP, IMSConfigData, etc.)
- Real-time APDU communication trace
- Multi-chipset auto-detection (Qualcomm, Samsung LSI, MediaTek, Apple)

---

## 2. Screen Layout

Launch the exe and the browser opens automatically at `http://127.0.0.1:8083`.

### Header Bar

From left to right:

| Element | Description |
|---------|-------------|
| 📡 SIM AT Command Tool | App title and version |
| 📋 Device Setup | Opens device setup guide popup |

Second header bar:

| Element | Description |
|---------|-------------|
| Serial Port dropdown | Lists available modem ports |
| 🔄 button | Refreshes port list (disconnects if currently connected) |
| 🔌 Connect | Initiates connection to the selected port |
| IMSI / MSISDN | Subscriber info read from SIM after connection |
| 🔐 VERIFY ADM | Opens ADM key verification popup |
| ADM1–4 dots | Gray = not verified, Green = verified |

### Main 3-Panel Layout

The screen is divided into three panels from left to right:

1. **💻 APDU Log** (left) — Real-time APDU communication trace
2. **🗂️ SIM Files** (center) — File tree browser
3. **📄 File Contents** (right) — Displays content of the selected file

---

## 3. Device Setup

Click the **📋 Device Setup** button in the header to open the setup guide popup.

### Android (Samsung)

1. **USB Driver** (Windows only): Install [Samsung Android USB Driver](https://developer.samsung.com/android-usb-driver)
2. **Developer mode**: Settings → About phone → Software information → tap **Build number** 5 times
3. **Developer options**:
   - Enable **USB debugging** → connect USB → allow RSA key
   - Enable **3GPP AT commands** (required for AT+CSIM access)
4. **USB setting**: Select **DM + MODEM + ADB** mode
   - Setup method varies by device model and carrier

### iOS (iPhone)

Carrier software version upgrade required. Then configure:

- Carrier Settings → Baseband Manager → Logging Settings
  - **Qualcomm chipset**: Mode → Passive, External Hardware (QXDM) — Windows only
  - **Apple Modem**: Mode → Passive, External Hardware (AT Only) — macOS only

---

## 4. Connecting to a Device

### Selecting a Port

1. Connect the device via USB
2. The **Serial Port** dropdown shows available modem ports
   - Modem ports are automatically sorted to the top
   - If ADB is connected, the device model name appears as `[Model Name]` next to the port
3. If no ports appear, click the 🔄 button to refresh

### Connection Sequence

Click **🔌 Connect** to start the automatic connection sequence:

1. Open serial port and verify AT response
2. Scan logical channels 0–19 → identify USIM/ISIM locations
3. If ISIM is not found during scan:
   - Read EF.DIR → search for ISIM AID
   - Attempt ISIM access via AT+CCHO/CGLA fallback
4. AT+CFUN reset (Qualcomm/Samsung LSI — stabilizes RETRIEVE DATA)
5. Read EF.ARR (MF, USIM, ISIM — caches security conditions)
6. Read IMSI and MSISDN → display in header

When connection completes:
- Button changes to **⚡ Connected**
- Port dropdown becomes disabled
- ADM verification area appears
- File tree loads automatically

### Apple Modem (iPhone)

For iPhones with Apple Modem, the port description contains "iPhone", which activates Apple modem mode automatically:
- If channel scan fails, **Apple Reset** (AT+CFUN 0/1 power cycle) is performed automatically
- AT polling continues for up to 35 seconds until modem recovers, then re-scans

> **Note**: Qualcomm-based iPhones (iPhone 15 and earlier) enumerate as "Modem" and are not affected by this behavior. Only Apple Modem models (iPhone 16e / 17 Air, etc.) are detected as iPhone ports.

### Disconnecting

Click the 🔄 button to disconnect the current session and refresh the port list.

---

## 5. Browsing SIM Files

### File Tree Panel

The SIM Files panel displays files grouped by application:

- **MF** — Master File (ICCID, DIR, ARR)
- **ADF.USIM** — USIM application files
- **ADF.ISIM** — ISIM application files (shown only when ISIM access is available)

Each group header shows the EF file count for that group.

### Basic / All Toggle

Use the toggle switch at the top of the panel to change the display scope:

- **Basic**: Shows only commonly used EF files (~25 files)
- **All**: Shows all defined EF files (100+ files)

> **⚠️ Apple Modem note**: Use caution with All mode on Apple Modem devices. Selecting unsupported or non-existent files (SW=6A82) often triggers an AT command ERROR that requires a modem reset. **Basic mode is recommended** for Apple Modem environments.

### Search

Type in the search box to filter in real-time by:
- File name (e.g., "URSP", "PLMN")
- FID (e.g., "6F38", "4F0B")
- Path (e.g., "5GS")

Click the **✕** button to clear the search.

### Collapsing / Expanding Groups

Click any group header (MF, ADF.USIM, ADF.ISIM) to collapse or expand that group's file list.

### Status Dots

A colored dot appears to the left of each file entry:
- ⚪ Gray: Not yet read
- 🟢 Green: Read successfully (clicking again shows cached value instantly)
- 🔴 Red: Read failed (clicking again retries the read)

### File Entry Information

Each file entry displays:
- FID (4-digit hex)
- File name
- Structure type abbreviation (TF = Transparent, LF = Linear Fixed, CF = Cyclic, BER-TLV)

---

## 6. Reading Files

Click any file in the tree to read it.

### Read Process

1. SELECT the file via the correct path (AID → DF → EF)
2. Read method is determined automatically by file structure:
   - **Transparent**: READ BINARY (split into 255-byte chunks if needed)
   - **Linear Fixed / Cyclic**: READ RECORD (sequential read of each record)
   - **BER-TLV**: RETRIEVE DATA (automatic continuation on SW=62xx)
3. Raw hex is decoded into structured data
4. Result is displayed in the File Contents panel

### Caching

Once a file is read, it is cached:
- Clicking the same file again displays the cached value instantly without a server request
- APDU log shows "cached" label
- After a write operation, the cache is cleared and the file is automatically re-read

### Clicking Another File During Read

If you click another file while a read is in progress, the tool queues the request and automatically reads the last-clicked file after the current read completes.

---

## 7. Decoded Views

### Decode / Raw Toggle

Use the toggle at the top of the File Contents panel to switch display modes:

- **🔍 Decode**: Structured view tailored to the file type
- **🔢 Raw**: Original hex data (Transparent shows hex string, Linear Fixed shows per-record table)

### File-Specific Decode Views

| File Type | Display Format |
|-----------|---------------|
| PLMN files (PLMNwAcT, OPLMNwAcT, HPLMNwAcT, FPLMN, EHPLMN) | MCC / MNC / AcT table |
| Service tables (UST, IST, EST) | Service number + name + True/False status |
| ACC (Access Control Class) | Class 0–15 + True/False status |
| ARR (Access Rule Reference) | Read/Update/Write/Activate/Deactivate conditions table |
| URSP | Tree-formatted rule display (Precedence, TD, RSD) |
| Other EFs | JSON structure (pySim-based decoding) |

### Error Display

When a read fails, the SW code and its description are shown automatically:

| SW | Description |
|----|-------------|
| 6982 | Security status not satisfied |
| 6A82 | File not found |
| 6981 | Command incompatible |
| 6983 | Authentication blocked |
| 6700 | Wrong length |
| 6A83 | Record not found |
| 6A84 | Not enough memory |
| 6D00 | INS not supported |
| 6E00 | Class not supported |

**Additional info on SW=6982**: When a security-not-satisfied error occurs, the file's ARR access conditions (Read/Update) are displayed alongside the error. This helps identify which ADM key is required at a glance.

### Copy Button

Click **📋 Copy** to copy the current view to clipboard:
- Table views: Copied as TSV (paste directly into Excel)
- JSON views: Copied as formatted JSON text
- Raw view: Copied as hex string

---

## 8. APDU Log

The APDU Log panel on the left shows all AT+CSIM communication in real-time.

### Log Format

- `── text` (italic): Operation label (SELECT, READ BINARY, UPDATE RECORD, etc.)
- `>> hex` (blue): Sent APDU
  - CLA/INS bytes: Purple highlight
  - GET RESPONSE (C0): Shown in gray
- `<< hex`: Received response
  - SW=9000: Green highlight
  - SW=61xx/9Fxx: Default color
  - Other SW: Red highlight

### Hex Formatting

- 16 bytes per line with aligned indentation
- SW (Status Word) is color-highlighted at the end of each response

### Auto-scroll

The log automatically scrolls to the bottom when new entries are added.

### Copy / Clear

- **📋 Copy**: Copies the entire log as text to clipboard
- **🗑 Clear**: Clears the log buffer

---

## 9. ADM Verification

### Opening the ADM Popup

Click the **🔐 VERIFY ADM** button in the header bar.

### ADM1–ADM4 Verification

The popup shows four independent ADM key input fields:
- Each ADM key is 16 hex characters (8 bytes)
- Character count is shown in real-time (e.g., `(12/16)`)
- The **Verify** button enables when 16 valid hex characters are entered
- On successful verification:
  - Button turns green and shows "Verified"
  - Input field becomes read-only
  - Corresponding ADM dot in the header turns green
  - Write button state for the currently selected file is immediately refreshed

### Auto-fill (test_profile.json)

If `test_profile.json` exists and contains a profile matching the current MSISDN, ADM keys are automatically filled when the popup opens.

File location: `test_profile.json` in the same folder as the exe

File format:
```json
{
  "profiles": [
    {
      "msisdn": "821012345678",
      "adm1": "0123456789ABCDEF"
    }
  ]
}
```

Supported fields:
- `msisdn` (required): matching key
- `adm1`–`adm4` (optional): 16-digit hex, auto-filled when matched

Matching logic:
- Full MSISDN match

### Write Button Control

After reading a file, the **✏️ Write** button state is automatically determined by ARR conditions:

| Condition | Write Button | Tooltip (on hover) |
|-----------|-------------|---------|
| ALWAYS | ✅ Enabled | (none) |
| ADM verified | ✅ Enabled | (none) |
| ADM not verified | ❌ Disabled | "🔐 ADM1 verification required" |
| NEVER | ❌ Disabled | "🚫 Write not allowed (NEVER)" |
| No ARR info | ❌ Disabled | "🔐 ARR info not available" |
| Read failed | ❌ Disabled | "⚠️ Read failed — cannot write" |

Hover over a disabled Write button to see the orange tooltip explaining why.

---

## 10. Writing to Files

### Prerequisites

1. The file must be read first (loads current data and metadata)
2. Write availability is determined by ARR access conditions:
   - **ALWAYS** condition: Write is possible without ADM verification (Write button enabled immediately)
   - **ADM** condition: The required ADM key must be verified first (when the Write button is disabled, the tooltip shows which ADM key is needed)
   - **NEVER** condition: Write is not allowed

### Opening the Write Popup

Click the **✏️ Write** button in the File Contents panel header to open the Write popup.

### Editor Types

The appropriate editor is automatically selected based on file type:

#### Hex Editor (default)

- Direct hex input
- Real-time byte count display (e.g., `(20/20)`)
- Write button enables only when data length matches the expected file size
- **↩ Restore** button reverts to the original value

#### PLMN Editor

Applies to PLMNwAcT, OPLMNwAcT, HPLMNwAcT, FPLMN, and EHPLMN files:

- **Table mode**: MCC / MNC / AcT (when applicable) input table
  - Enter values directly in each row
  - Empty rows are encoded as FFFFFF
  - MCC and MNC must both be filled or both be empty (error shown if only one is filled)
- **Hex mode**: Direct hex editing
- **Table ↔ Hex toggle**: Bidirectional sync between modes

#### Service Table Editor (UST/IST/EST)

- **Table mode**: Service number + name + True/False dropdown
  - 3GPP standard service names loaded automatically
  - Toggle individual services ON/OFF via dropdown
- **Hex mode**: Direct hex editing
- **Table ↔ Hex toggle**: Bidirectional sync between modes

#### ACC Editor

- True/False dropdown for each of Class 0–15
- Automatically encoded as 16-bit bitmap

### Executing a Write

Once you modify a value, the **✏️ Write** button enables. Click it to:

1. Send UPDATE BINARY or UPDATE RECORD via AT+CSIM
2. SW result appears in the APDU log
3. After 1 second, the file is automatically re-read to confirm the write
4. On success: "✅ Done" is displayed, and the File Contents panel updates

### Record Selection (Linear Fixed)

For Linear Fixed files, a **Record Number** dropdown appears at the top of the popup:
- Selecting a record loads that record's hex data into the editor
- Each record can be written individually

---

## 11. BER-TLV Files (URSP)

### Reading

BER-TLV files (e.g., EF.URSP) are read using RETRIEVE DATA (INS=CB):

1. Send proprietary CLA + RETRIEVE DATA to request tag 0x80 data (`CB 00 80 01 80 00`)
2. Check response SW:
   - **SW=9000**: Data received completely
   - **SW=62xx**: More data remaining → repeat RETRIEVE DATA (`CB 00 00 00`) to fetch next chunk
   - Other error SW: Read failed
3. All chunks are concatenated and decoded as a URSP rule tree per 3GPP TS 24.526

### URSP Tree View

The decoded view displays rules in tree format:
```
URSP Rules
├─ URSP rule 1
│  ├─ Precedence: 0
│  ├─ Traffic descriptor
│  │  └─ OS Id + OS App Id
│  │     ├─ OS: Android
│  │     └─ Slice Category: ENTERPRISE
│  └─ Route selection descriptor list
│     └─ Route selection descriptor 1
│        ├─ Precedence: 255
│        └─ Route selection descriptor contents
│           └─ S-NSSAI
│              ├─ SST: 01 (eMBB)
│              └─ SD: 000001
└─ URSP rule 2
   ├─ Precedence: 255
   ├─ Traffic descriptor
   │  └─ Match-all
   └─ Route selection descriptor list
      └─ Route selection descriptor 1
         ├─ Precedence: 255
         └─ Route selection descriptor contents
            └─ S-NSSAI
               └─ SST: 01 (eMBB)
```

### BER-TLV Write

The Write popup shows a dedicated BER-TLV editor:

1. **Tag selection**: Choose an existing tag from the dropdown (or new tag 0x80)
2. **Hex data editing**: Full TLV format (tag + BER-length + value)
3. **Validation**:
   - Hex data must start with the selected tag
   - BER-length must match the actual value length
   - Error message shown if validation fails
4. **Write execution**: DELETE DATA + SET DATA executed in sequence
5. **↩ Restore**: Reverts to the original TLV value

### URSP Rule Analyzer Link

A link to the external URSP Rule Analyzer tool is provided at the bottom of the Write popup:
- 🔗 https://huggingface.co/spaces/Joostone/ursp-rule-analyzer

---

## 12. Modem Compatibility

| Platform | Chipset | AT+CSIM | Channel Scan | ISIM Access | RETRIEVE DATA |
|----------|---------|---------|--------------|-------------|---------------|
| Android | Qualcomm | ✅ | ✅ | Scanned channel | ⚠️ (OK after CFUN reset) |
| Android | Samsung LSI | ✅ | ✅ (proprietary CLA) | Scanned channel | ✅ |
| Android | MediaTek | ✅ | ❌ | AT+CCHO/CGLA | ✅ |
| iOS | Qualcomm | ✅ | ✅ | Scanned channel | ⚠️ (OK after CFUN reset) |
| iOS | Apple Modem | ⚠️ (intermittent ERROR) | ✅ | Scanned channel | ✅ |

### Chipset-Specific Notes

- **Qualcomm**: MMGSDI internal state may block RETRIEVE DATA (INS=CB) with SW=6981. Resolved by automatic CFUN reset during connection.
- **Samsung LSI**: STATUS command only responds to proprietary CLA (bit8=1), so the tool uses proprietary CLA for channel scanning.
- **MediaTek**: Logical channel scan not supported. ISIM is accessed via AT+CCHO/CGLA fallback automatically (see Section 4).
- **Apple Modem**: AT+CSIM may intermittently return ERROR, especially when selecting non-existent files. Detected and recovered via CFUN cycle + GTI polling (see Section 4).

> For details on the connection sequence (channel scan, CCHO fallback, CFUN reset, Apple Reset), see [Section 4: Connecting to a Device](#4-connecting-to-a-device).

---

## 13. Troubleshooting

### Connection fails: "SIM access failed"

- Verify the device is in modem mode (Samsung: DM + MODEM + ADB)
- Check that 3GPP AT commands is enabled in developer options
- Disconnect and reconnect USB

### "USIM not found on any channel"

- Apple modem: Apple Reset is attempted automatically. If it fails, reconnect USB
- Other chipsets: Verify you selected the correct modem port (not the diagnostics port)

### Write button disabled

- Hover over the button to see the tooltip — it shows which ADM key is required
- Verify the correct ADM key in the ADM popup
- Files with NEVER condition cannot be written

### RETRIEVE DATA fails (SW=6981)

- Qualcomm MMGSDI issue
- Should be resolved by the automatic CFUN reset during connection
- If it persists, disconnect USB and reconnect

### USB cable disconnected during operation

- Serial port loss is detected automatically
- **⚠️ Connection Lost** popup appears
- Connection UI resets, but the read cache is preserved
- Reconnect USB and click Connect again — previously read data is still visible

---

**© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr> All rights reserved.**
