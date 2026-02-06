# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZMK firmware configuration for the **Eyelash Sofle** split ergonomic keyboard (nRF52840-based). This is not application code — it's keyboard firmware config that gets compiled via CI into .uf2 files for flashing.

## Build & Deploy

There is no local build. Firmware is compiled by GitHub Actions on push:

- **build.yml**: Compiles firmware using `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`. Triggered on any push (except `keymap-drawer/` changes). Produces .uf2 artifacts downloadable from the Actions tab.
- **draw.yml**: Auto-generates keymap SVG visualization via `caksoylar/keymap-drawer`. Triggered on changes to `config/`, the workflow file, or `keymap_drawer.config.yaml`. Auto-commits the updated SVG with prefix `[Draw]`.

Build targets (defined in `build.yaml`):
- `eyelash_sofle_right` — right half with nice_view display
- `eyelash_sofle_left` — left half with nice_view display + ZMK Studio support (studio-rpc-usb-uart snippet, CONFIG_ZMK_STUDIO=y)
- `nice_nano_v2` with `settings_reset` shield — for factory reset

Flashing: download .uf2 from GitHub Actions artifacts, put keyboard in bootloader mode, copy .uf2 to the mass-storage device.

## Architecture

### Dependencies (`config/west.yml`)
- **ZMK firmware** v0.3.0 from zmkfirmware
- **eyelash_sofle module** from `https://github.com/a741725193/zmk-sofle` (board-specific drivers)
- Self-path `/boards` exposes the local board definitions in `boards/arm/eyelash_sofle/`

### Key Files to Edit

| File | Purpose |
|------|---------|
| `config/eyelash_sofle.keymap` | Keymap layers, combos, encoder bindings, mouse config |
| `config/eyelash_sofle.conf` | Feature toggles (RGB, sleep, BT power, debounce, backlight, pointing) |
| `build.yaml` | Build targets and shields |
| `config/west.yml` | ZMK and module version pins |
| `boards/arm/eyelash_sofle/*.dtsi` | Hardware pin definitions, matrix, peripherals |

### Keymap Structure

The keymap (`config/eyelash_sofle.keymap`) uses ZMK devicetree syntax with `#include` for behaviors and bindings. The layout is a 5-row split with 13 columns per row (58 keys total including encoder button).

- **Layer 0**: QWERTY base. Encoder = volume. `mo 1` and `mo 2` on thumb cluster for momentary layer access.
- **Layer 1**: F-keys, navigation, mouse clicks, RGB controls. Encoder = scroll.
- **Layer 2**: Bluetooth profiles (BT_SEL 0-4), BT_CLR, output toggle (USB/BLE), sys_reset, bootloader, soft_off. Encoder = scroll.
- **Layers 3-4**: Empty (transparent), available for customization.

### Special Behaviors

- **Soft-off combo**: key-positions `<14 28 40>` (Q + S + Z) with 2-second hold triggers deep sleep. Requires hardware reset button to wake.
- **Mouse movement**: `&mmv` and `&msc` behaviors with custom scaling (`zip_xy_scaler 2 1`, `zip_scroll_scaler 2 1`) and tuned acceleration.
- **Encoder**: EC11 rotary, volume on layer 0, scroll on layers 1+.

### Board Hardware (`boards/arm/eyelash_sofle/`)

Device tree files define the nRF52840 pin matrix, encoder GPIO, WS2812 LED strip, external power control, PWM backlight, and battery monitoring. Left and right halves have separate `.dts` files sharing a common `.dtsi`.

## Conventions

- Commits that only touch keymap/config trigger both build and draw workflows. The draw workflow auto-commits SVG updates — expect `[Draw]` prefixed commits after keymap changes.
- The `feat/colemak` branch is for Colemak layout work; `main` is the stable branch.
- Board definitions follow ZMK's standard module structure under `boards/arm/<board_name>/`.

## ⌨️ ZMK Configuration & Collaboration Rules

### 🛠️ Strategic Approach
* **Decision Flow**: 조사 우선 (Research) --> 계획 수립 (Planning) --> 순차적 접근 (Execution).
* **Tone & Style**: 감상적 표현(칭찬, 반성 등)은 배제하고, 해결책 위주의 간결하고 명확한 기술적 답변을 지향한다.

### 🇰🇷 Communication & Documentation
* **Language**: 복잡한 로직(매크로, 콤보, 비헤이비어)에는 반드시 **한국어 주석**을 병행 표기하여 가독성을 높인다.
* **Git Commits**: 대장의 지시 전까지 커밋 문구를 먼저 작성하지 않는다. 요청 시 한국어로 작성하되 컨벤션은 영어를 유지한다.
* **Visuals**: Mermaid 다이어그램 작성 시 특수문자(괄호 등)는 반드시 **큰따옴표("")**로 감싸 오류를 방지한다.

### 🌿 Branching Strategy
* **`feat/qwerty`**: 표준 QWERTY 배열 및 기본 엄지 최적화 베이스라인.
* **`feat/colemak`**: Colemak 배열 실험 브랜치. 쿼티의 엄지 UX 로직을 이식하여 관리.
* **`feat/workman`**: Workman 배열 실험 브랜치.
* **Constraint**: 모든 브랜치는 공통 비헤이비어(Behaviors)와 엄지 최적화 설정을 공유하며, ZMK Studio 수정 사항은 반드시 코드로 역포팅한다.

### 🎯 Keymap Best Practices (Thumb-Centric)
* **Left Thumb (Inner)**: `Mod-Tap (LShift / LANG1)`. 탭 시 한/영 전환, 홀드 시 Shift로 동작.
* **Left Thumb (Middle)**: `Layer-Tap (layer1 / Keyboard Spacebar)`. 탭 시 스페이스, 홀드 시 NAV 레이어 진입.
* **Right Roller Click**: `Key Press (BSPC)`. 조이스틱 클릭 시 Backspace로 동작하여 새끼손가락 피로도 감소.
* **Right Thumb (Inner)**: `Layer-Tap (layer2 / Keyboard Return Enter)`. 표준 엔터 신호 사용 권장.

### 🏗️ Build & Sync
* **Studio-Code Sync**: ZMK Studio(GUI) 변경 사항은 펌웨어 빌드 시 초기화될 수 있으므로, 반드시 `.keymap` 소스에 직접 반영한다.
* **Branch CI**: `build.yml`과 `draw.yml` 모두 `main` 및 `feat/*` 브랜치 push 시 자동 실행된다.
* **Environment**: 특정 OS에 종속되지 않는 보편적인 `West` 빌드 및 `Devicetree` 분석 원칙을 준수한다.

### 🚨 작업 원칙
* **수정 범위**: 모든 수정은 `config/eyelash_sofle.keymap` 위주로 진행한다. `.conf`나 보드 정의 파일은 필요 최소한으로만 변경.
* **엄지·조이스틱 일관성**: 베이스 배열(QWERTY/Colemak/Workman 등)이 바뀌어도 엄지 클러스터와 조이스틱(롤러) 설정은 브랜치 간 동일하게 유지한다.
* **충돌 분석 우선**: 새로운 로직(콤보, 매크로, 레이어 등)을 추가하기 전에 기존 마우스 비헤이비어(`&mmv`, `&msc`, `&mkp`, 인풋 프로세서)와의 충돌 여부를 먼저 분석한다.