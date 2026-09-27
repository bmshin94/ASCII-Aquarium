# 🐠 ASCII Aquarium 프로젝트 전수조사 & 분석 노트

> 작성일: 2026-09-27
> 작성: Claude Code (카리나 페르소나) + @bmshin94
> 대상 저장소: <https://github.com/bmshin94/ASCII-Aquarium>
> 원본(Upstream): <https://github.com/POWER-PILL/ASCII-Aquarium>
> 웹 플래셔: <https://power-pill.github.io/ASCII-Aquarium/>
> 언론 보도: [Hackaday — Adorable ASCII Aquarium Lives On Your Desk](https://hackaday.com/2026/05/24/adorable-ascii-aquarium-lives-on-your-desk)

---

## 목차
1. [프로젝트 정체](#1-프로젝트-정체)
2. [저장소 전수조사](#2-저장소-전수조사)
3. [코드 구조 분석](#3-코드-구조-분석)
4. [쉬운 설명 (초보자용)](#4-쉬운-설명-초보자용)
5. [설치 및 사용법](#5-설치-및-사용법)
6. [플러그인 / 스킬 / MCP 여부](#6-플러그인--스킬--mcp-여부)
7. [API 토큰 필요 여부](#7-api-토큰-필요-여부)
8. [GitHub에서 유명한 이유](#8-github에서-유명한-이유)
9. [로컬 에이전트 구축과의 관계](#9-로컬-에이전트-구축과의-관계)
10. [React / PHP 포팅 가능성](#10-react--php-포팅-가능성)
11. [수익화 아이디어](#11-수익화-아이디어)
12. [발견된 이슈](#12-발견된-이슈)

---

## 1. 프로젝트 정체

**ASCII Aquarium**은 소프트웨어 라이브러리가 아니라 **ESP32 기반 터치스크린 보드(CYD)에서 동작하는 Arduino 펌웨어**다.

`><(((°>` 같은 ASCII 문자로 만든 물고기가 320x240 화면에서 실시간으로 헤엄친다.
**영상 재생이 아니라 매 프레임 물리 연산을 수행하는 실시간 시뮬레이션**이다.

### 대상 하드웨어

| 항목 | 사양 |
|---|---|
| 보드 | ESP32-2432S028R "Cheap Yellow Display" (CYD) |
| 디스플레이 | ILI9341 320x240 컬러 LCD |
| 터치 | XPT2046 저항막 |
| 추가 | SD카드 슬롯, RGB 앰비언트 LED, WiFi 내장 |
| 가격 | 약 15,000 ~ 20,000원 (AliExpress) |
| 지원 변종 | Standard CYD / CYD2USB / jc3248w535 (ESP32-S3) |

---

## 2. 저장소 전수조사

총 **28개 파일** (`.git` 제외).

| 경로 | 역할 | 비고 |
|---|---|---|
| `ASCII_Aquarium_CYD/ASCII_Aquarium_CYD_2.39.ino` | **메인 소스** | 7,616줄 단일 파일 |
| `ASCII_Aquarium_CYD/ASCII_Aquarium_CYD_2.20.ino` | 구버전 소스 | 6,320줄 |
| `docs/index.html` | 웹 플래셔 페이지 | 213줄, esp-web-tools v10 사용 |
| `docs/manifest.json` | 기본 매니페스트 | v2.39 (CYD) |
| `docs/manifest-cyd.json` | Standard CYD | ESP32, v2.39 |
| `docs/manifest-cyd2usb.json` | CYD2USB 변종 | ESP32, v2.39 |
| `docs/manifest-jc3248w535.json` | JC3248w535 | ESP32-S3, v2.33_JC |
| `docs/firmware/*.bin` | 컴파일된 펌웨어 8개 | **총 약 37MB** |
| `docs/assets/*.gif` | 데모 GIF 7개 | README 홍보용 |
| `User_Setup.h` | TFT_eSPI 핀맵 설정 | ILI9341, SPI 40MHz |
| `User_Setup_Select_CYD.h` | TFT_eSPI 패치 안내 | 10줄 주석 문서 |
| `README.md` | 메인 문서 | 기능/설치/하드웨어 안내 |
| `ASCII_Aquarium_Release_Notes_v2.20.md` | v1.67 → v2.20 변경사항 | |
| `ASCII_Aquarium_Release_Notes_v2.39.md` | v2.20 → v2.39 변경사항 | |
| `CLAUDE.md` | Claude Code 프로젝트 지침 | 카리나 페르소나 설정 |

### 펌웨어 바이너리 목록

```
ascii-aquaruim-cyd-v1.66.bin                    4.0M
ascii-aquaruim-cyd-v1.67.bin                    4.0M
ascii-aquarium-cyd-v2.20.bin                    4.0M
ascii-aquarium-cyd2usb-v2.20.bin                4.0M
ascii-aquarium-cyd-v2.39-merged.bin             4.0M
ascii-aquarium-cyd2usb-v2.39.bin.zip            705K
ascii-aquarium-jc3248w535-v2.30_JC-merged.bin   8.0M
ascii-aquarium-jc3248w535-v2.33_JC-merged.bin   8.0M
```

---

## 3. 코드 구조 분석

### 메인 루프 (`loop()`)

```
updateClock()           → 시계 갱신
serviceBootButton()     → BOOT 버튼 (스크린샷)
processTouch()          → 터치 입력
serviceLightSchedule()  → 조명 스케줄
serviceAutoSky()        → 시간대별 하늘색 블렌딩
serviceBackgroundRainbow() → 무지개 색상 순환
serviceWifi()           → WiFi / NTP
serviceSettingsPersistence() → 설정 저장
serviceAutoFeed()       → 자동 급여
updateFlakes(dt)        → 먹이 물리
updateBubbles(dt)       → 거품
updateFish(dt)          → 물고기 AI
updateOctopus/Seahorse/Snail/Jellyfish/Squid(dt)
keepVisitorsSeparated() → 충돌 회피
renderFrame()           → 스프라이트 렌더 + 화면 출력
```

### 주요 구조체

| 구조체 | 설명 |
|---|---|
| `Fish` | 위치, 속도, 방황 바이어스, 깊이 밝기, 렌더 색상 |
| `Bubble` | 위치, 흔들림 진폭, 위상 |
| `Flake` | 먹이 입자 (하강 속도) |
| `Octopus` | 문어 + `cthulhu` 희귀 변종 플래그 |
| `Seahorse` / `Snail` / `Jellyfish` / `Squid` | 방문 생물들 |
| `FishSpecies` | ASCII 글리프 + 기본 색상 |
| `AsciiClockFont` / `AsciiClockGlyph` | ASCII 시계 폰트 20종 |
| `TimezoneOption` | 타임존 라벨 + POSIX 문자열 |

### 물고기 12종 글리프

| 이름 | 글리프 | 색상 |
|---|---|---|
| Small Green | `><>` | Emerald |
| Blue Dart | `>)))'>` | Azure |
| Pink Bubble | `oO0` | Pink |
| Golden Emperor | `><((( '>` | Amber |
| Purple Jellyfish | `~~{o}` | Violet |
| Red Snapper | `><(((o>` | Red |
| Orange Wrasse | `` ><((((>` `` | Orange |
| Teal Glider | `><((( '>` | Teal |
| Royal Indigo | `}>{{{{* >` | Indigo |
| Lilac Starfish | `><((( *>` | Lilac |
| Pink Tetra | `>(')>` | Pink |
| Yellow Minnow | `>'>` | Yellow |

### 기술적으로 주목할 점

1. **더블 버퍼링** — `TFT_eSprite` 캔버스에 전체 프레임을 그린 뒤 한 번에 전송 (깜빡임 제거)
2. **메모리 선점 전략** — `setup()` 최상단에서 캔버스부터 할당
   > `// Allocate the biggest RAM block before NVS/WiFi/TFT setup can fragment heap.`
3. **좌우 반전 렌더링** — `mirrorAsciiBracket()`으로 `>`↔`<`, `(`↔`)` 변환 후 캐싱
4. **깊이 셰이딩** — Y좌표 기반 밝기 조절로 원근감 표현
5. **군집 행동** — schooling + separation + wandering (보이드 알고리즘 간소화)
6. **dt 기반 물리** — `clampVal(elapsedMs, 1, 50)`으로 프레임률 독립 시뮬레이션
7. **캡처 모드 분리** — SD 쓰기 지연이 애니메이션에 영향 주지 않도록 `aquariumNowMs` 별도 관리

---

## 4. 쉬운 설명 (초보자용)

### 비유

> **"전자 액자에 그림 대신 물고기가 사는 프로그램"**

```
1단계  알리에서 2만원짜리 노란 보드 구매
2단계  브라우저에서 버튼 클릭 → 펌웨어 설치
3단계  USB 꽂아서 책상에 놓으면 끝
```

게임기와 카트리지 관계와 동일하다.
- 게임기 = CYD 보드 (하드웨어)
- 카트리지 = 이 프로젝트 (펌웨어)

### 왜 ASCII인가?

작은 마이크로컨트롤러는 RAM이 극히 적어 이미지 파일을 다룰 수 없다.
문자는 매우 가볍기 때문에 **제약 속에서 나온 예술적 선택**이다.

### 영상과 다른 점

| | 영상 재생 | ASCII Aquarium |
|---|---|---|
| 방식 | 녹화본 반복 | 매 프레임 재계산 |
| 터치 반응 | 없음 | 물고기가 먹이로 이동 |
| 물고기 위치 | 항상 동일 | 매번 다름 |
| 비유 | 유튜브 영상 | 심즈 게임 |

---

## 5. 설치 및 사용법

### 방법 A: 웹 플래셔 (권장, 약 5분)

**준비물**
- CYD 보드
- **데이터 전송 가능한** USB 케이블 (충전 전용 케이블 주의)
- Chrome / Edge / Firefox 151+ **데스크톱** 브라우저 (WebSerial API 필요)

**절차**
1. 보드를 USB로 PC에 연결
2. <https://power-pill.github.io/ASCII-Aquarium/> 접속
3. 보드에 맞는 버튼 선택 (Standard CYD / CYD2USB / jc3248w535)
4. 시리얼 포트 선택 (CH340 또는 CP2102)
5. Install → 완료

**주의**: Arduino IDE 시리얼 모니터가 열려 있으면 포트 충돌로 실패한다.

**트러블슈팅**

| 증상 | 해결 |
|---|---|
| 포트가 보이지 않음 | CH340 / CP2102 USB 드라이버 설치 |
| "This page needs HTTPS" | HTTPS로 접속 |
| "Unsupported browser" | Chrome 사용 |
| 플래싱 중 중단 | USB 포트 변경 또는 케이블 교체 |

### 방법 B: 소스 빌드

1. Arduino IDE 설치
2. 보드 매니저에서 `esp32` (Espressif) 설치
3. 라이브러리 설치: `TFT_eSPI` (Bodmer), `XPT2046_Touchscreen`
4. `User_Setup.h`를 `Documents/Arduino/libraries/TFT_eSPI/`에 복사
   - `User_Setup_Select.h`는 통째로 덮어쓰지 말고 `#include <User_Setup.h>` 줄만 확인
5. `ASCII_Aquarium_CYD_2.39.ino` 열기
6. 보드: "ESP32 Dev Module" 선택 후 업로드

### 조작법

| 조작 | 기능 |
|---|---|
| 화면 탭 | 먹이 투하 |
| **좌측 상단 구석** 탭 | 숨겨진 HUD 메뉴 |
| 설정 > Tank | 물고기 수(6~36), 거품(0~50), 화면 뒤집기, 타임드 이벤트 |
| 설정 > Seaweed | 해초 길이 / 흔들림 |
| 설정 > Clock | 시계 ON/OFF, 12·24h, 타임존, ASCII 폰트 20종 |
| 설정 > Background | 배경 스타일, Auto Sky, 레인보우 |
| 숨김 **B** 버튼 | LCD 밝기, RGB 앰비언트 LED, 조명 스케줄 |
| WiFi 패널 | 네트워크 스캔, 온스크린 키보드, NTP 동기화 |
| 뒷면 BOOT 버튼 길게 | SD카드로 BMP 스크린샷 저장 |

---

## 6. 플러그인 / 스킬 / MCP 여부

| 구분 | 해당 여부 | 근거 |
|---|:---:|---|
| Claude 플러그인 | ❌ | `.claude-plugin/` 디렉터리 없음 |
| Claude 스킬 | ❌ | `SKILL.md` 없음 |
| MCP 서버 | ❌ | MCP 설정·서버 코드 없음 |
| **Arduino 펌웨어** | ✅ | C++ 하드웨어 코드 |

**결론: AI와 무관한 순수 임베디드 하드웨어 프로젝트다.**

### 예외: `CLAUDE.md`

커밋 `580f660 docs: created CLAUDE.md persona guide`로 추가된 파일로,
Claude Code가 자동으로 읽는 **프로젝트 지침서(메모리 파일)** 이다.
플러그인·스킬·MCP 중 어느 것에도 해당하지 않는다.

| 구분 | 정체 | 형태 |
|---|---|---|
| CLAUDE.md | 프로젝트 규칙 / 맥락 메모 | 마크다운 파일 1개 |
| Skill | 특정 작업 절차서 (필요 시 로드) | `SKILL.md` + 폴더 |
| Plugin | 스킬·명령어·훅 묶음 배포 패키지 | 폴더 구조 |
| MCP | 외부 도구·데이터 연결 서버 | 실행되는 서버 프로세스 |

---

## 7. API 토큰 필요 여부

### 결론: **불필요하다.**

소스 전체 검색 결과 `token`, `API_KEY`, `Bearer` 등의 인증 코드가 **전혀 없다.**

네트워크를 사용하는 유일한 지점은 **NTP 시간 동기화**다.

```c
static constexpr const char* CLOCK_NTP_1 = "pool.ntp.org";
static constexpr const char* CLOCK_NTP_2 = "time.nist.gov";
static constexpr const char* CLOCK_NTP_3 = "time.google.com";
configTzTime(tz.posix, CLOCK_NTP_1, CLOCK_NTP_2, CLOCK_NTP_3);
```

NTP는 무료 공개 프로토콜로 인증 개념 자체가 없다.

| 입력 정보 | 용도 | 저장 위치 |
|---|---|---|
| WiFi SSID / 비밀번호 | 인터넷 연결 | ESP32 Preferences (NVS 플래시, 로컬) |

- 외부 서버로 데이터를 전송하는 코드가 없다 → **개인정보 유출 위험 없음**
- 구독료·API 과금 **없음**
- 토큰이 필요한 유일한 경우: 저장소를 fork해서 GitHub에 push할 때의 GitHub 토큰 (프로젝트와 무관)

---

## 8. GitHub에서 유명한 이유

1. **Hackaday 메인 기사 게재** — 메이커 커뮤니티 최대 매체, 대량 유입 트리거
2. **설치 마찰 제거** — `esp-web-tools` + GitHub Pages로 "사이트 접속 → 버튼 클릭" 한 번에 설치
3. **시각적 임팩트** — README 상단 GIF 7개로 3초 만에 이해 가능
4. **극한의 가성비** — 2만원 보드 + 무료 펌웨어로 완성품
5. **감성과 향수** — 80~90년대 터미널 ASCII 아트 + 물갈이 없는 어항의 힐링
6. **CYD 커뮤니티 특수** — 이미 큰 팬덤이 있으나 완성형 앱이 부족했던 공백을 채움
7. **지속적 업데이트 + 커뮤니티 기여 수용** — v1.66 → v2.39, 기여자(@mjpcomp) 크레딧 명시
8. **유머러스한 릴리즈 노트** — 문서 자체가 콘텐츠
   > *"The tank now contains fish, plants, bubbles, time, weather-ish sky moods, and mild cosmic dread. As one does."*

---

## 9. 로컬 에이전트 구축과의 관계

### 직접적 도움: 거의 없음

- LLM / MCP / 에이전트 루프 관련 코드 전무
- ESP32는 RAM 약 320KB로 로컬 LLM 구동 불가
- 도메인이 완전히 다름 (하드웨어 렌더링 vs AI 오케스트레이션)

### 간접적 활용 가치

| # | 배울 점 | 에이전트 적용 |
|---|---|---|
| 1 | `CLAUDE.md` 실전 페르소나 주입 | 에이전트 프롬프트 설계 연습 |
| 2 | 7,616줄 단일 파일 | 롱컨텍스트 에이전트 **벤치마크 테스트베드** |
| 3 | `loop()` 서비스 순회 스케줄러 | 에이전트 이벤트 루프 설계와 구조 동일 |
| 4 | Preferences(NVS) 영속 설정 분리 | 에이전트 메모리/상태 저장 설계 |
| 5 | 웹 플래셔 배포 UX | 에이전트 도구 배포 시 설치 마찰 제거 패턴 |

### 응용 아이디어: 에이전트의 물리적 출력 장치

```
로컬 에이전트 (PC)
   ↓ HTTP / MQTT / WebSocket
CYD 보드 (저가 위성 디스플레이)
   └─ 어항 배경 + 실시간 정보 오버레이
      ├─ CI 빌드 상태 (초록 물고기 / 빨강 물고기)
      ├─ 작업 진행률 (거품 개수)
      ├─ 알림 도착 → 문어 등장
      └─ 에러 발생 → 크툴루 문어 소환
```

ESP32에 WiFi HTTP 클라이언트를 붙이는 것은 수십 줄 수준이라 구현 난이도가 낮다.

---

## 10. React / PHP 포팅 가능성

### React — 가능하며, 오히려 더 쉽다

브라우저는 ESP32보다 훨씬 강력하다. 메모리 제약이 없고 GPU 가속도 사용 가능하다.

**추천 스택**
```
React 18 + TypeScript
  렌더링: <canvas> 2D  (DOM 기반은 60fps 유지 불가)
  루프:   requestAnimationFrame + dt 계산
  상태:   Zustand (설정) / useRef (물고기 배열)
  스타일: Tailwind CSS
```

**핵심 구현 골격**
```tsx
// 물고기 배열은 useState로 관리하면 매 프레임 리렌더가 발생하므로 useRef 사용
const fishRef = useRef<Fish[]>([]);

useEffect(() => {
  let raf: number;
  let last = performance.now();
  const tick = (now: number) => {
    const dt = Math.min((now - last) / 1000, 0.05); // .ino의 clampVal과 동일
    last = now;
    updateFish(fishRef.current, dt);
    updateBubbles(bubblesRef.current, dt);
    render(ctx);
    raf = requestAnimationFrame(tick);
  };
  raf = requestAnimationFrame(tick);
  return () => cancelAnimationFrame(raf);
}, []);
```

**포팅 난이도 매핑**

| .ino 원본 | React 대응 | 난이도 |
|---|---|---|
| `Fish[]` 구조체 배열 | TS `interface Fish` + 배열 | 쉬움 |
| `updateFish(dt)` | 로직 거의 그대로 이식 | 쉬움 |
| `TFT_eSprite` 더블버퍼 | Canvas 기본 제공 | 불필요 |
| `tft.drawString()` | `ctx.fillText()` | 쉬움 |
| `Preferences` (NVS) | `localStorage` | 쉬움 |
| 터치 이벤트 | `onPointerDown` | 쉬움 |
| NTP 시간 동기화 | `new Date()` | 불필요 |
| ASCII 시계 폰트 20종 | 데이터 그대로 이식 | 보통(분량) |
| 물고기 좌우 반전 | 동일한 mirror 함수 | 쉬움 |

**예상 작업량**: 핵심 어항 2~3일, 설정 UI 포함 풀 포팅 1~2주

### PHP — 가능하지만 역할이 다르다

**PHP로 불가능/부적합**
- 실시간 애니메이션 (서버가 매 프레임 계산하는 구조는 비현실적)
- 클라이언트 렌더링 (PHP는 브라우저에서 실행되지 않음)

**PHP의 적합한 역할 — 백엔드**
```
[프론트] React / Canvas   ← 애니메이션
     ↕ REST API
[백엔드] PHP (Laravel)
     ├─ 회원가입 / 로그인
     ├─ 어항 설정 프리셋 저장·공유
     ├─ 커스텀 물고기 ASCII 업로드 & 갤러리
     ├─ 결제 (프리미엄 테마)
     ├─ 인기 어항 랭킹
     └─ CYD 보드 설정 푸시 (IoT 엔드포인트)
```

**PHP 단독으로 가능한 활용 — 동적 SVG 위젯**
```php
<?php
// GitHub 프로필 README에 삽입 가능한 동적 어항 이미지
header('Content-Type: image/svg+xml');
echo renderAquariumSVG(seed: $_GET['seed'], fishCount: 12);
```

**결론**
- 애니메이션: React / Canvas
- 회원·결제·공유: PHP (Laravel) 백엔드
- 둘을 조합하면 완성형 SaaS 구성 가능

---

## 11. 수익화 아이디어

> ⚠️ **선행 조건**: 이 저장소에는 **LICENSE 파일이 없다.**
> 라이선스가 명시되지 않은 코드는 법적으로 "모든 권리 유보" 상태이므로,
> **상업적 이용 전 원작자(POWER-PILL)의 허락이 반드시 필요하다.**
> 아이디어 ⑤⑥⑦은 새로 제작하는 것이므로 라이선스 이슈에서 자유롭다.

### 티어 1 — 즉시 시작 가능

#### ① 완제품 판매 (최우선 추천)

| 항목 | 금액 |
|---|---|
| CYD 보드 | 15,000원 |
| 3D 프린팅 케이스 | 3,000원 |
| USB-C 케이블 | 2,000원 |
| 패키지 + 스티커 | 2,000원 |
| **원가 합계** | **22,000원** |
| **판매가** | **59,000 ~ 79,000원** |
| **마진** | **37,000 ~ 57,000원 (63~72%)** |

- 채널: 스마트스토어 / 텐바이텐 / 아이디어스 / 와디즈 / Etsy
- 포지셔닝: "ESP32 개발보드"(X) → **"물갈이 없는 미니 디지털 어항"**(O)
- 타겟: 사무직 직장인, 개발자, 반려동물 못 키우는 1인 가구

**업셀 옵션**

| 옵션 | 추가가 | 원가 |
|---|---|---|
| 빔스플리터 큐브 (홀로그램 효과) | +30,000원 | 12,000원 |
| 케이스 색상 커스텀 | +5,000원 | 0원 |
| 이름 각인 | +10,000원 | 0원 |
| 커플 2개 세트 | 110,000원 | 44,000원 |

#### ② 3D 프린팅 케이스 (디지털 굿즈)
- 원가 0원, 순마진 100%, 한 번 제작 후 영구 수익
- 플랫폼: MakerWorld / Cults3D / Printables / Thingiverse
- MakerWorld는 다운로드 기반 포인트를 현금으로 전환 가능
- 차별화 포인트: 무드등 겸용 / 시계 각도 / 케이블 정리 / 빔스플리터 전용

#### ③ 유튜브 · 틱톡 콘텐츠
- 수익: 애드센스 + 쿠팡파트너스 + 알리 어필리에이트 + 자사 완제품 유입
- 시리즈 기획
  1. "2만원 디지털 어항 만들기" (입문)
  2. "코드 수정해서 내 물고기 만들기" (개발자 타겟)
  3. "홀로그램 어항 만들기" (빔스플리터)
  4. "React로 웹 버전 포팅하기" (기술 채널)

### 티어 2 — 중기 (개발 필요)

#### ④ 커스텀 펌웨어 / B2B

| 상품 | 타겟 | 가격대 |
|---|---|---|
| 카페 테이블 사이니지 | 카페·식당 | 8~12만원 |
| 기업 굿즈 (로고 각인) | 마케팅팀 | 100개 단위 |
| 사무실 KPI 대시보드 | 스타트업 | 15만원+ |
| 웨딩 답례품 (커플 ASCII) | 예비부부 | 3~5만원 |
| 수족관·과학관 전시 소품 | 기관 | 견적 |

B2B는 개당 마진과 물량 모두 유리하다.

#### ⑤ React 웹 SaaS (가장 추천)

```
Free        : 기본 어항, 물고기 6마리, 워터마크
Pro  $3/월  : 물고기 36마리, 전체 테마, 커스텀 ASCII, 워터마크 제거
Team $10/월 : 팀 대시보드 연동, 알림 물고기
```

**수익원**
1. 구독 (Stripe / 토스페이먼츠)
2. 테마 판매 (크리스마스·심해·우주 어항, 개당 $2)
3. 임베드 위젯 (블로그·노션 삽입, 무료는 워터마크)
4. 커스텀 물고기 디자인 마켓 (수수료 30%)
5. 데스크톱 스크린세이버 앱 (Mac/Win, $4.99 — 웹 코드 재활용)

**바이럴 후크 — GitHub README 위젯**
```markdown
![My Aquarium](https://asciiaquarium.app/w/bmshin94.svg)
```
개발자 프로필에 확산되면 무료 바이럴 효과가 크다.

#### ⑥ 디지털 다운로드 상품

| 상품 | 가격 | 마진 |
|---|---|---|
| ASCII 물고기 팩 (50종) | $5 | 100% |
| ASCII 시계 폰트 팩 | $3 | 100% |
| Notion / Obsidian 위젯 템플릿 | $7 | 100% |
| 프리미엄 배경 테마 팩 | $4 | 100% |

플랫폼: Gumroad / Lemon Squeezy / 크몽

### 티어 3 — 장기 / 고위험

#### ⑦ 크라우드펀딩 → 자체 하드웨어
- 와디즈 / 킥스타터, 목표 3,000만원 (89,000원 × 400개)
- 전용 PCB + 아크릴 케이스 + 빔스플리터 + 전용 앱
- 리스크: 재고, KC 인증, 배송, CS 전부 직접 부담

#### ⑧ 오픈소스 스폰서십
- GitHub Sponsors / Patreon / Buy Me a Coffee
- 수익 자체는 작지만 브랜딩 효과가 크고 ①②⑤로 유입되는 깔때기 역할

### 추천 로드맵

```
1개월차   라이선스 확인 + 보드 구매 후 직접 제작 → 제작기 콘텐츠 업로드
2~3개월   차별화된 3D 케이스 설계 → MakerWorld 무료 배포 (수요 검증 + 무료 마케팅)
3~4개월   스마트스토어 완제품 소량 테스트 판매 (10개) → 반응 확인 후 확대
4~6개월   React 웹 버전 개발 → GitHub README 위젯 바이럴 시도
6개월+    SaaS 구독 모델 + B2B 영업 병행
```

**단일 추천**: ⑤ React 웹 버전 + GitHub README 위젯
(원가 0원, 보유 스택 일치, 무한 확장, 바이럴 가능, 라이선스 이슈 없음)

---

## 12. 발견된 이슈

| 심각도 | 이슈 | 설명 | 권장 조치 |
|---|---|---|---|
| 🚨 높음 | **LICENSE 파일 없음** | 법적으로 "모든 권리 유보" 상태. 재배포·상업이용 불가 | 원작자에게 라이선스 명시 요청 (Issue 등록) |
| ⚠️ 중간 | **저장소에 37MB 바이너리** | `.bin` 8개가 Git 히스토리에 포함되어 clone이 무거움 | GitHub Releases 또는 Git LFS로 이전 |
| ℹ️ 낮음 | **Fork 출처 혼재** | README 내 링크가 전부 원본(POWER-PILL)을 가리킴 | fork 명시 또는 링크 정리 |
| ℹ️ 낮음 | **하드코딩된 개인 경로** | `User_Setup.h`에 `/Users/kertgartner/...` 경로 주석 잔존 | 일반화된 경로 안내로 수정 |
| ℹ️ 낮음 | **7,616줄 단일 파일** | 유지보수·협업 난이도 높음 | 기능별 모듈 분리 (리팩토링 연습 소재로 적합) |
| ℹ️ 낮음 | **manifest 문구 오류** | CYD2USB 카드에 "Installs the older v2.39" 표기 (v2.39는 최신) | 문구 수정 |

---

## 참고 링크

- 이 저장소: <https://github.com/bmshin94/ASCII-Aquarium>
- 원본 저장소: <https://github.com/POWER-PILL/ASCII-Aquarium>
- 웹 플래셔: <https://power-pill.github.io/ASCII-Aquarium/>
- Hackaday 기사: <https://hackaday.com/2026/05/24/adorable-ascii-aquarium-lives-on-your-desk>
- v2.20 릴리즈 노트: <https://github.com/POWER-PILL/ASCII-Aquarium/blob/main/ASCII_Aquarium_Release_Notes_v2.20.md>
- v2.39 릴리즈 노트: <https://github.com/POWER-PILL/ASCII-Aquarium/blob/main/ASCII_Aquarium_Release_Notes_v2.39.md>
- CYD 보드 (AliExpress): <https://www.aliexpress.com/item/1005004971720824.html>
- jc3248w535 보드: <https://www.aliexpress.com/item/1005007566332450.html>
- 3D 케이스 - Basic Snap-fit: <https://makerworld.com/en/models/2835243>
- 3D 케이스 - CYD Desk Buddy: <https://makerworld.com/en/models/2787810>
- esp-web-tools: <https://github.com/esphome/esp-web-tools>
- TFT_eSPI: <https://github.com/Bodmer/TFT_eSPI>

---

```
><(((°>   ><>   >)))'>   ~~{o}   >'>
```
