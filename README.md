# 📡 SIM AT Command Tool

A web-based tool for reading, writing, and decoding SIM/USIM/ISIM card files via `AT+CSIM` (raw APDU) over USB modem port.

![License](https://img.shields.io/badge/License-All%20Rights%20Reserved-red)

---

## 💡 Why This Tool?

As eSIM-only devices become the industry standard, physically removing a SIM card for external reader access is no longer possible. This tool accesses SIM files directly through the device's USB modem port using AT commands — no card removal, no external reader hardware. It works with the SIM in its live operating state inside the device, supporting both physical SIM and eSIM environments across all major modem chipsets (Qualcomm, Samsung LSI, MediaTek, Apple).

---

## ⚡ Key Features

- **Logical channel auto-scan** — automatically identifies USIM/ISIM across channels 0–19
- **Multi-chipset support** — Qualcomm, Samsung LSI, MediaTek, Apple (with auto-recovery)
- **ISIM dual-path access** — scanned channel or AT+CCHO/CGLA fallback
- **Auto-decoding** — 100+ EF files decoded to structured JSON (PLMN, service tables, URSP, etc.)
- **ARR security conditions** — shows required ADM key before writing
- **BER-TLV read/write** — tag-based RETRIEVE DATA + DELETE/SET DATA (URSP, IMSConfigData)
- **Real-time APDU log** — full hex trace with CLA/INS color coding
- **EF write editors** — PLMN table, service table, ACC, hex, BER-TLV editors

---

## 🌐 Online Demo

Try the URSP Rule Analyzer (decode-only):  
🔗 https://huggingface.co/spaces/Joostone/ursp-rule-analyzer

---

## 🚀 Download

Download the latest Windows exe and user guides from [Releases](../../releases).

No installation required — just run the exe.

---

## 📖 How to Use

### Device Setup

#### Android (Samsung)

1. Enable **Developer mode** and **USB debugging**
2. Enable **3GPP AT commands** in developer options
3. Set USB mode to **DM + MODEM + ADB**

#### iOS (iPhone)

Carrier software version upgrade required, then enable AT modem logging in Carrier Settings.

### Quick Start

1. Connect device via USB
2. Select modem port → click **Connect**
3. Browse and click any EF file to read
4. Verify ADM key → click **Write** to modify

> Full user guide (Korean / English) is included in each release as PDF.

---

## 📋 Modem Compatibility

| Platform | Chipset | AT+CSIM | Channel Scan | ISIM Access | RETRIEVE DATA |
|----------|---------|---------|--------------|-------------|---------------|
| Android | Qualcomm | ✅ | ✅ | Scanned channel | ⚠️ (auto CFUN reset) |
| Android | Samsung LSI | ✅ | ✅ (proprietary CLA) | Scanned channel | ✅ |
| Android | MediaTek | ✅ | ❌ | AT+CCHO/CGLA | ✅ |
| iOS | Qualcomm | ✅ | ✅ | Scanned channel | ⚠️ (auto CFUN reset) |
| iOS | Apple Modem | ⚠️ | ✅ | Scanned channel | ✅ |

---

## 📚 References

- 3GPP TS 31.102 — USIM application
- 3GPP TS 31.103 — ISIM application
- 3GPP TS 102.221 — UICC-Terminal interface
- 3GPP TS 24.526 — UE policies (URSP)
- 3GPP TS 27.007 — AT command set (AT+CSIM, AT+CCHO, AT+CGLA)
- ISO/IEC 7816-4 — Smart card commands

---

## 👤 Author

**JUSEOK AHN**  
LG U+ / Network Infra Technology  
ajs3013@lguplus.co.kr

---

## 📄 License

© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr>. All rights reserved.
