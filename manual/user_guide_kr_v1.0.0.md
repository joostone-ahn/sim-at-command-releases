# SIM AT Command Tool — 사용자 가이드

> 버전: v1.0.0  
> 최종 수정: 2026-05-12

---

## 목차

1. [개요](#1-개요)
2. [화면 구성](#2-화면-구성)
3. [디바이스 준비](#3-디바이스-준비)
4. [연결하기](#4-연결하기)
5. [파일 탐색](#5-파일-탐색)
6. [파일 읽기](#6-파일-읽기)
7. [디코딩 뷰](#7-디코딩-뷰)
8. [APDU 로그](#8-apdu-로그)
9. [ADM 인증](#9-adm-인증)
10. [파일 쓰기](#10-파일-쓰기)
11. [BER-TLV 파일 (URSP)](#11-ber-tlv-파일-ursp)
12. [모뎀 호환성](#12-모뎀-호환성)
13. [문제 해결](#13-문제-해결)

---

## 1. 개요

SIM AT Command Tool은 USB 모뎀 포트를 통해 AT+CSIM (Raw APDU) 명령으로 SIM/USIM/ISIM 카드 파일을 읽고, 쓰고, 디코딩하는 도구입니다. exe 파일을 실행하면 웹 브라우저에서 동작합니다.

**주요 기능:**
- 100개 이상의 EF 파일 읽기 및 자동 디코딩
- ARR 기반 접근 제어를 적용한 EF 파일 쓰기
- BER-TLV 태그 기반 읽기/쓰기 (URSP, IMSConfigData 등)
- 실시간 APDU 통신 로그
- 멀티 칩셋 자동 감지 (Qualcomm, Samsung LSI, MediaTek, Apple)

---

## 2. 화면 구성

exe를 실행하면 자동으로 브라우저가 열리며 `http://127.0.0.1:8083` 에 접속됩니다.

### 상단 헤더 바

왼쪽부터 순서대로:

| 영역 | 설명 |
|------|------|
| 📡 SIM AT Command Tool | 앱 타이틀 및 버전 표시 |
| 📋 Device Setup | 디바이스 설정 가이드 팝업 열기 |

두 번째 헤더 바:

| 영역 | 설명 |
|------|------|
| Serial Port 드롭다운 | 사용 가능한 모뎀 포트 목록 |
| 🔄 버튼 | 포트 목록 새로고침 (연결 중이면 연결 해제 후 새로고침) |
| 🔌 Connect | 선택한 포트로 연결 시작 |
| IMSI / MSISDN | 연결 후 SIM에서 읽은 가입자 정보 |
| 🔐 VERIFY ADM | ADM 키 인증 팝업 열기 |
| ADM1~4 상태 점 | 회색=미인증, 초록=인증 완료 |

### 메인 3패널 레이아웃

화면은 좌측부터 3개 패널로 구성됩니다:

1. **💻 APDU Log** (좌측) — 실시간 APDU 통신 기록
2. **🗂️ SIM Files** (중앙) — 파일 트리 탐색기
3. **📄 File Contents** (우측) — 선택한 파일의 내용 표시

---

## 3. 디바이스 준비

상단의 **📋 Device Setup** 버튼을 클릭하면 설정 가이드 팝업이 표시됩니다.

### Android (Samsung)

1. **USB 드라이버** (Windows만 해당): Samsung Android USB Driver 설치
2. **개발자 모드**: 설정 → 휴대전화 정보 → 소프트웨어 정보 → 빌드번호 5회 탭
3. **개발자 옵션**:
   - USB 디버깅 활성화 → USB 연결 → RSA 키 허용
   - 3GPP AT commands 활성화 (AT+CSIM 접근에 필수)
4. **USB 모드**: DM + MODEM + ADB 모드 선택
   - 설정 방법은 기기 모델 및 통신사에 따라 다름

### iOS (iPhone)

캐리어 소프트웨어 버전 업그레이드 필요. 이후 설정:

- Carrier Settings → Baseband Manager → Logging Settings
  - **Qualcomm 칩셋**: Mode → Passive, External Hardware (QXDM) — Windows 전용
  - **Apple Modem**: Mode → Passive, External Hardware (AT Only) — macOS 전용

---

## 4. 연결하기

### 포트 선택

1. USB로 디바이스를 연결합니다
2. **Serial Port** 드롭다운에 사용 가능한 모뎀 포트가 표시됩니다
   - 모뎀 포트가 자동으로 상단에 정렬됩니다
   - ADB가 연결된 경우 포트 옆에 기기 모델명이 `[모델명]` 형태로 표시됩니다
3. 포트가 보이지 않으면 🔄 버튼으로 새로고침합니다

### 연결 과정

**🔌 Connect** 버튼을 클릭하면 다음 과정이 자동으로 진행됩니다:

1. 시리얼 포트 열기 및 AT 응답 확인
2. 논리 채널 0~19 스캔 → USIM/ISIM 위치 식별
3. ISIM이 스캔에서 발견되지 않으면:
   - EF.DIR 읽기 → ISIM AID 검색
   - AT+CCHO/CGLA 폴백으로 ISIM 접근 시도
4. AT+CFUN 리셋 (Qualcomm/Samsung LSI — RETRIEVE DATA 안정화 목적)
5. EF.ARR 읽기 (MF, USIM, ISIM — 보안 조건 캐싱)
6. IMSI, MSISDN 읽기 → 헤더에 표시

연결이 완료되면:
- 버튼이 **⚡ Connected** 로 변경됩니다
- 포트 드롭다운이 비활성화됩니다
- ADM 인증 영역이 표시됩니다
- 파일 트리가 자동으로 로드됩니다

### Apple Modem (iPhone)

Apple Modem 탑재 iPhone의 경우, 포트 description에 "iPhone"이 포함되어 자동으로 Apple 모뎀 모드로 동작합니다:
- 채널 스캔 실패 시 자동으로 **Apple Reset** (AT+CFUN 0/1 전원 사이클)을 수행합니다
- 모뎀 복구까지 최대 35초간 AT 폴링 후 재스캔합니다

> **참고**: Qualcomm 칩셋 탑재 iPhone(iPhone 15 이전 모델)은 포트가 "Modem"으로 잡히므로 이 동작에 해당하지 않습니다. Apple Modem 탑재 모델(iPhone 16e / 17 Air 등)만 해당됩니다.

### 연결 해제

🔄 버튼을 클릭하면 현재 연결이 해제되고 포트 목록이 새로고침됩니다.

---

## 5. 파일 탐색

### 파일 트리 패널

SIM Files 패널에 파일이 애플리케이션별로 그룹화되어 표시됩니다:

- **MF** — Master File (ICCID, DIR, ARR)
- **ADF.USIM** — USIM 애플리케이션 파일
- **ADF.ISIM** — ISIM 애플리케이션 파일 (ISIM 접근 가능 시에만 표시)

각 그룹 헤더에는 해당 그룹의 EF 파일 수가 표시됩니다.

### Basic / All 토글

패널 상단의 토글 스위치로 표시 범위를 전환합니다:

- **Basic**: 자주 사용하는 주요 EF 파일만 표시 (~25개)
- **All**: 정의된 모든 EF 파일 표시 (100개 이상)

> **⚠️ Apple Modem 주의**: Apple Modem 환경에서는 All 모드 사용 시 주의가 필요합니다. 단말이 지원하지 않거나 SIM에 존재하지 않는 파일(SW=6A82)을 SELECT하면 AT command ERROR가 발생하여 모뎀 리셋이 필요한 상황이 빈번합니다. Apple Modem에서는 **Basic 모드 위주로 사용**할 것을 권장합니다.

### 검색

검색창에 입력하면 실시간으로 필터링됩니다:
- 파일명 (예: "URSP", "PLMN")
- FID (예: "6F38", "4F0B")
- 경로 (예: "5GS")

검색어를 지우려면 **✕** 버튼을 클릭합니다.

### 그룹 접기/펼치기

각 그룹 헤더(MF, ADF.USIM, ADF.ISIM)를 클릭하면 해당 그룹의 파일 목록을 접거나 펼 수 있습니다.

### 상태 점 (Status Dot)

각 파일 항목 왼쪽에 색상 점이 표시됩니다:
- ⚪ 회색: 아직 읽지 않음
- 🟢 초록: 읽기 성공 (클릭 시 캐시된 값 즉시 표시)
- 🔴 빨강: 읽기 실패 (클릭 시 재시도)

### 파일 항목 정보

각 파일 항목에는 다음이 표시됩니다:
- FID (4자리 hex)
- 파일명
- 구조 타입 약어 (TF=Transparent, LF=Linear Fixed, CF=Cyclic, BER-TLV)

---

## 6. 파일 읽기

파일 트리에서 파일을 클릭하면 읽기가 시작됩니다.

### 읽기 과정

1. 올바른 경로로 파일 SELECT (AID → DF → EF)
2. 파일 구조에 따라 자동으로 읽기 방식 결정:
   - **Transparent**: READ BINARY (255바이트 단위 분할)
   - **Linear Fixed / Cyclic**: READ RECORD (각 레코드 순차 읽기)
   - **BER-TLV**: RETRIEVE DATA (SW=62xx 연속 응답 자동 처리)
3. Raw hex를 구조화된 데이터로 디코딩
4. 우측 File Contents 패널에 결과 표시

### 캐싱

한 번 읽은 파일은 캐시됩니다:
- 같은 파일을 다시 클릭하면 서버 요청 없이 즉시 표시됩니다
- APDU 로그에 "cached" 표시됩니다
- 쓰기 후에는 캐시가 삭제되고 자동으로 다시 읽습니다

### 읽기 중 다른 파일 클릭

읽기 진행 중에 다른 파일을 클릭하면, 현재 읽기가 완료된 후 마지막으로 클릭한 파일을 자동으로 읽습니다.

---

## 7. 디코딩 뷰

### Decode / Raw 토글

File Contents 패널 상단의 토글로 표시 모드를 전환합니다:

- **🔍 Decode**: 파일 타입에 맞는 구조화된 뷰
- **🔢 Raw**: 원본 hex 데이터 (Transparent는 hex 문자열, Linear Fixed는 레코드별 테이블)

### 파일별 디코딩 뷰

| 파일 타입 | 표시 형식 |
|-----------|-----------|
| PLMN 파일 (PLMNwAcT, OPLMNwAcT, HPLMNwAcT, FPLMN, EHPLMN) | MCC / MNC / AcT 테이블 |
| 서비스 테이블 (UST, IST, EST) | 서비스 번호 + 이름 + True/False 상태 |
| ACC (Access Control Class) | Class 0~15 + True/False 상태 |
| ARR (Access Rule Reference) | Read/Update/Write/Activate/Deactivate 조건 테이블 |
| URSP | 트리 형태 규칙 표시 (Precedence, TD, RSD) |
| 기타 EF | JSON 구조 (pySim 기반 디코딩) |

### 에러 표시

읽기 실패 시 SW 코드와 자동 설명이 표시됩니다:

| SW | 설명 |
|----|------|
| 6982 | Security status not satisfied |
| 6A82 | File not found |
| 6981 | Command incompatible |
| 6983 | Authentication blocked |
| 6700 | Wrong length |
| 6A83 | Record not found |
| 6A84 | Not enough memory |
| 6D00 | INS not supported |
| 6E00 | Class not supported |

**SW=6982 시 추가 정보**: 보안 조건 미충족 에러가 발생하면, 해당 파일의 ARR 접근 조건(Read/Update)이 함께 표시됩니다. 이를 통해 어떤 ADM 키가 필요한지 바로 확인할 수 있습니다.

### Copy 버튼

**📋 Copy** 버튼으로 현재 표시된 내용을 클립보드에 복사합니다:
- 테이블 뷰: TSV 형식 (Excel에 붙여넣기 가능)
- JSON 뷰: 포맷된 JSON 텍스트
- Raw 뷰: hex 문자열

---

## 8. APDU 로그

좌측 APDU Log 패널에 모든 AT+CSIM 통신이 실시간으로 표시됩니다.

### 로그 형식

- `── 텍스트` (이탤릭): 작업 설명 (SELECT, READ BINARY, UPDATE RECORD 등)
- `>> hex` (파란색): 전송한 APDU
  - CLA/INS 바이트: 보라색 강조
  - GET RESPONSE(C0)는 회색으로 표시
- `<< hex`: 수신한 응답
  - SW=9000: 초록색 강조
  - SW=61xx/9Fxx: 기본색
  - 기타 SW: 빨간색 강조

### Hex 포맷팅

- 16바이트 단위로 줄바꿈 및 들여쓰기 정렬
- SW(Status Word)는 응답 끝에서 색상 강조

### 자동 스크롤

새 로그가 추가되면 자동으로 최하단으로 스크롤됩니다.

### Copy / Clear

- **📋 Copy**: 전체 로그를 텍스트로 클립보드에 복사
- **🗑 Clear**: 로그 버퍼 초기화

---

## 9. ADM 인증

### ADM 팝업 열기

헤더의 **🔐 VERIFY ADM** 버튼을 클릭합니다.

### ADM1~ADM4 인증

팝업에 4개의 독립적인 ADM 키 입력란이 표시됩니다:
- 각 ADM 키는 16자리 hex (8바이트)
- 입력 시 실시간으로 글자 수 카운트 표시 (예: `(12/16)`)
- 16자리 유효한 hex가 입력되면 **Verify** 버튼 활성화
- 인증 성공 시:
  - 버튼이 초록색 "Verified"로 변경
  - 입력란이 읽기 전용으로 변경
  - 헤더의 해당 ADM 상태 점이 초록색으로 변경
  - 현재 선택된 파일의 Write 버튼 상태가 즉시 갱신

### 자동 채우기 (test_profile.json)

`test_profile.json` 파일이 존재하고 현재 MSISDN과 매칭되는 프로파일이 있으면, 팝업을 열 때 ADM 키가 자동으로 채워집니다.

파일 위치: exe 파일과 같은 폴더의 `test_profile.json`

파일 형식:
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

지원 필드:
- `msisdn` (필수): 매칭 기준
- `adm1`~`adm4` (선택): 16자리 hex, 매칭 시 자동 채움

매칭 방식:
- MSISDN 전체 일치

### Write 버튼 제어

파일을 읽은 후, **✏️ Write** 버튼의 상태가 ARR 조건에 따라 자동으로 결정됩니다:

| 조건 | Write 버튼 | 툴팁 (마우스 오버 시) |
|------|-----------|------|
| ALWAYS | ✅ 활성화 | (없음) |
| ADM 인증 완료 | ✅ 활성화 | (없음) |
| ADM 미인증 | ❌ 비활성화 | "🔐 ADM1 verification required" |
| NEVER | ❌ 비활성화 | "🚫 Write not allowed (NEVER)" |
| ARR 정보 없음 | ❌ 비활성화 | "🔐 ARR info not available" |
| 읽기 실패 | ❌ 비활성화 | "⚠️ Read failed — cannot write" |

비활성화된 Write 버튼에 마우스를 올리면 주황색 툴팁이 표시됩니다.

---

## 10. 파일 쓰기

### 전제 조건

1. 파일을 먼저 읽어야 합니다 (현재 데이터 및 메타데이터 로드)
2. ARR 접근 조건에 따라 Write 가능 여부가 결정됩니다:
   - **ALWAYS** 조건인 파일: ADM 인증 없이 바로 쓰기 가능 (Write 버튼 즉시 활성화)
   - **ADM 조건인 파일**: 해당 ADM 키를 먼저 인증해야 합니다 (Write 버튼 비활성화 시 툴팁에 필요한 ADM 키가 표시됩니다)
   - **NEVER** 조건인 파일: 쓰기 불가

### Write 팝업 열기

File Contents 패널의 **✏️ Write** 버튼을 클릭하면 Write 팝업이 열립니다.

### 에디터 종류

파일 타입에 따라 적절한 에디터가 자동으로 선택됩니다:

#### Hex 에디터 (기본)

- 직접 hex 입력
- 바이트 수 실시간 표시 (예: `(20/20)`)
- 파일 크기와 일치해야 Write 버튼 활성화
- **↩ Restore** 버튼으로 원래 값 복원

#### PLMN 에디터

PLMNwAcT, OPLMNwAcT, HPLMNwAcT, FPLMN, EHPLMN 파일에 적용됩니다:

- **Table 모드**: MCC / MNC / AcT(해당 시) 입력 테이블
  - 각 행에 직접 값 입력
  - 빈 행은 FFFFFF로 인코딩
  - MCC와 MNC는 둘 다 입력하거나 둘 다 비워야 함 (한쪽만 입력 시 에러 표시)
- **Hex 모드**: 직접 hex 편집
- **Table ↔ Hex 토글**: 양방향 동기화

#### 서비스 테이블 에디터 (UST/IST/EST)

- **Table 모드**: 서비스 번호 + 이름 + True/False 드롭다운
  - 3GPP 표준 서비스명 자동 로드
  - 드롭다운으로 개별 서비스 ON/OFF 전환
- **Hex 모드**: 직접 hex 편집
- **Table ↔ Hex 토글**: 양방향 동기화

#### ACC 에디터

- Class 0~15 각각에 True/False 드롭다운
- 16비트 비트맵으로 자동 인코딩

### 쓰기 실행

값을 수정하면 **✏️ Write** 버튼이 활성화됩니다. 클릭하면:

1. UPDATE BINARY 또는 UPDATE RECORD를 AT+CSIM으로 전송
2. SW 결과가 APDU 로그에 표시
3. 1초 후 자동으로 파일을 다시 읽어 결과 확인 (Re-read)
4. 성공 시 "✅ Done" 표시, File Contents 패널도 갱신

### Record 선택 (Linear Fixed)

Linear Fixed 파일의 경우 팝업 상단에 **Record Number** 드롭다운이 표시됩니다:
- 레코드를 선택하면 해당 레코드의 hex 데이터가 에디터에 로드됩니다
- 레코드별로 개별 쓰기가 가능합니다

---

## 11. BER-TLV 파일 (URSP)

### 읽기

BER-TLV 파일(예: EF.URSP)은 RETRIEVE DATA (INS=CB)로 읽습니다:

1. Proprietary CLA + RETRIEVE DATA 명령으로 tag 0x80 데이터를 요청 (`CB 00 80 01 80 00`)
2. 응답 SW 확인:
   - **SW=9000**: 데이터 수신 완료
   - **SW=62xx**: 잔여 데이터 있음 → RETRIEVE DATA (`CB 00 00 00`)를 반복 전송하여 이어받기
   - SW=62xx가 아닌 에러: 읽기 실패
3. 모든 청크를 연결한 전체 데이터를 3GPP TS 24.526 기반으로 URSP 규칙 트리로 디코딩

### URSP 트리 뷰

디코딩된 뷰는 트리 형태로 표시됩니다:
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

### BER-TLV 쓰기

Write 팝업에서 BER-TLV 전용 에디터가 표시됩니다:

1. **Tag 선택**: 드롭다운에서 기존 태그 선택 (또는 새 태그 0x80)
2. **Hex 데이터 편집**: 전체 TLV 형식 (tag + BER-length + value)
3. **유효성 검증**:
   - hex 데이터가 선택한 태그로 시작하는지 확인
   - BER-length와 실제 value 길이 일치 확인
   - 불일치 시 에러 메시지 표시
4. **Write 실행**: DELETE DATA + SET DATA 순서로 실행
5. **↩ Restore**: 원래 TLV 값으로 복원

### URSP Rule Analyzer 링크

Write 팝업 하단에 외부 URSP Rule Analyzer 도구 링크가 제공됩니다:
- 🔗 https://huggingface.co/spaces/Joostone/ursp-rule-analyzer

---

## 12. 모뎀 호환성

| 플랫폼 | 칩셋 | AT+CSIM | 채널 스캔 | ISIM 접근 | RETRIEVE DATA |
|---------|-------|---------|-----------|-----------|---------------|
| Android | Qualcomm | ✅ | ✅ | 스캔 채널 | ⚠️ (CFUN 리셋 후 정상) |
| Android | Samsung LSI | ✅ | ✅ (proprietary CLA) | 스캔 채널 | ✅ |
| Android | MediaTek | ✅ | ❌ | AT+CCHO/CGLA | ✅ |
| iOS | Qualcomm | ✅ | ✅ | 스캔 채널 | ⚠️ (CFUN 리셋 후 정상) |
| iOS | Apple Modem | ⚠️ (간헐적 ERROR) | ✅ | 스캔 채널 | ✅ |

### 칩셋별 참고사항

- **Qualcomm**: MMGSDI 내부 상태로 인해 RETRIEVE DATA(INS=CB)가 SW=6981로 차단될 수 있음. 연결 시 자동 CFUN 리셋으로 해결
- **Samsung LSI**: STATUS 명령이 proprietary CLA (bit8=1)에서만 응답하므로, 채널 스캔 시 proprietary CLA를 사용
- **MediaTek**: 논리 채널 스캔 미지원. ISIM은 AT+CCHO/CGLA 폴백으로 자동 접근 (4장 참조)
- **Apple Modem**: AT+CSIM이 간헐적으로 ERROR를 반환하며, 존재하지 않는 파일 SELECT 시에도 발생 빈도가 높음. 감지 시 CFUN 사이클 + GTI 폴링으로 자동 복구 (4장 참조)

> 연결 시퀀스(채널 스캔, CCHO 폴백, CFUN 리셋, Apple Reset)의 상세 동작은 [4. 연결하기](#4-연결하기)를 참조하세요.

---

## 13. 문제 해결

### 연결 실패: "SIM access failed"

- 디바이스가 모뎀 모드인지 확인 (Samsung: DM + MODEM + ADB)
- 개발자 옵션에서 3GPP AT commands가 활성화되어 있는지 확인
- USB를 분리했다가 다시 연결

### "USIM not found on any channel"

- Apple 모뎀: 자동으로 Apple Reset이 시도됩니다. 실패 시 USB 재연결
- 기타 칩셋: 올바른 모뎀 포트를 선택했는지 확인 (진단 포트가 아닌 모뎀 포트)

### Write 버튼 비활성화

- 버튼에 마우스를 올려 툴팁 확인 — 필요한 ADM 키가 표시됩니다
- ADM 팝업에서 해당 키를 인증하세요
- NEVER 조건인 파일은 쓰기 불가

### RETRIEVE DATA 실패 (SW=6981)

- Qualcomm MMGSDI 이슈
- 연결 시 자동 CFUN 리셋으로 해결되어야 함
- 지속되면 USB 분리 후 재연결

### USB 케이블 분리 시

- 시리얼 포트 유실이 자동 감지됩니다
- **⚠️ Connection Lost** 팝업이 표시됩니다
- 연결 UI가 초기화되지만, 읽기 캐시는 보존됩니다
- USB 재연결 후 Connect를 다시 클릭하면 됩니다

---

**© 2026 JUSEOK AHN <ajs3013@lguplus.co.kr> All rights reserved.**
