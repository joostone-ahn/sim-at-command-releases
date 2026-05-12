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

## 💻 Download

Download the latest exe from [Releases](../../releases).

---

## 📖 How to Use

See the User Guide for detailed instructions:
- [English](manual/user_guide_en_v1.0.0.md)
- [한국어](manual/user_guide_kr_v1.0.0.md)

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

**JUSEOK AHN (안주석)**  
**Email**: ajs3013@lguplus.co.kr  
**Organization**: LG U+  
**Role**: Technical Specialist, Telecommunications Engineer

---

## 📄 License

© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr>. All rights reserved.
