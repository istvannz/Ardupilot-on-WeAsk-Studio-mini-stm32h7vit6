# Installation of Ardupilot on WeAct Studio Mini STM32H743VIT6

This project looks at configuring the Ardupilot onto hte WeAct Studio Mini STM32H743VIT6
M32H743VIT6

## Hardware

| Component | Details |
|-----------|---------|
| MCU | STM32H743VIT6 — 480MHz, 2MB Flash, 1MB RAM |
| Crystal | 25MHz HSE |
| LED | PE3 (active HIGH) |
| USB | PA11/PA12 OTG Full Speed |
| User button | PC13 |
| Onboard flash | 8MB W25Q64 (QSPI) |
| SD card | SDMMC1 |

### Sensor Configuration

| Sensor | Interface | Pins |
|--------|-----------|------|
| ICM42688 IMU | SPI1 | SCK=PA5, MISO=PA6, MOSI=PA7, CS=PB12, INT=PB0 |
| BMP388 Barometer | I2C1 | SCL=PB8, SDA=PB9, addr=0x77 |
| BMM150 Compass | I2C1 | SCL=PB8, SDA=PB9, addr=0x10 |

### Pinout

| Function | Pin | Notes |
|----------|-----|-------|
| GPS | USART1 | TX=PA9, RX=PA10 |
| RC Receiver | USART2 | TX=PA2, RX=PA3 (CRSF/ELRS/SBUS) |
| Motor 1 (FR) | PE9 | TIM1_CH1 |
| Motor 2 (RL) | PE11 | TIM1_CH2 |
| Motor 3 (FL) | PE13 | TIM1_CH3 |
| Motor 4 (RR) | PE14 | TIM1_CH4 |
| SWDIO | PA13 | Debug |
| SWCLK | PA14 | Debug |

---

## Prerequisites

- Ubuntu 22.04 or later (or WSL2)
- `arm-none-eabi-gcc` 13.2.1
- `dfu-util`
- Python 3 with `pymavlink` installed
- STM32CubeProgrammer (for initial DFU flash)

```bash
sudo apt install gcc-arm-none-eabi dfu-util python3-pip
pip3 install pymavlink
```

---

## Build Instructions

### 1. Clone ArduPilot

```bash
git clone --recurse-submodules https://github.com/ArduPilot/ardupilot.git
cd ardupilot
Tools/environment_install/install-prereqs-ubuntu.sh -y
. ~/.profile
```

### 2. Apply Required Patches

These patches are mandatory — without them the bootloader will not run and USB will not enumerate.

#### 2a. Fix GCC 13.2 strlen miscompilation (critical — causes bootloader crash)

```bash
sed -i 's/^size_t strlen(const char \*s1)$/__attribute__((optimize("O0"))) size_t strlen(const char *s1)/' \
    Tools/AP_Bootloader/support.cpp
```

#### 2b. USB OTG reset + CRS init in board.c (critical — required for USB enumeration)

Find `void boardInit(void)` in `libraries/AP_HAL_ChibiOS/hwdef/common/board.c` and add inside the function body:

```c
void boardInit(void) {
  HAL_BOARD_INIT_HOOK_CALL

#if defined(STM32H723xx) || defined(STM32H7xx)
  // Reset USB OTG_HS peripheral to clear bootloader state
  RCC->AHB1RSTR |= RCC_AHB1RSTR_USB1OTGHSRST;
  volatile uint32_t dummy = RCC->AHB1RSTR;
  (void)dummy;
  RCC->AHB1RSTR &= ~RCC_AHB1RSTR_USB1OTGHSRST;

  // Enable CRS: sync HSI48 to USB SOF for stable enumeration
  RCC->APB1HENR |= RCC_APB1HENR_CRSEN;
  CRS->CFGR = (2U << 28);  // SYNCSRC = USB SOF
  CRS->CR |= CRS_CR_AUTOTRIMEN | CRS_CR_CEN;
#endif
}
```

#### 2c. USB turnaround time fix (required for stable USB on H7)

```bash
sed -i 's/#define TRDT_VALUE_FS           5/#define TRDT_VALUE_FS           9/' \
    modules/ChibiOS/os/hal/ports/STM32/LLD/OTGv1/hal_usb_lld.c
```

#### 2d. Add #ifndef guards to stm32h7_mcuconf.h

Several defines in `libraries/AP_HAL_ChibiOS/hwdef/common/stm32h7_mcuconf.h` lack `#ifndef` guards, causing redefinition errors. Wrap the following defines:

- `STM32_VOS` (line ~109)
- `STM32_PLL1_DIVM_VALUE`, `STM32_PLL1_DIVN_VALUE`, `STM32_PLL1_DIVP_VALUE`, `STM32_PLL1_DIVR_VALUE` (25MHz block)
- `STM32_PLL3_DIVN_VALUE`, `STM32_PLL3_DIVQ_VALUE`, `STM32_PLL3_DIVR_VALUE` (25MHz block)
- `STM32_USBSEL`
- `STM32_ADC_ADC12_DMA_STREAM`

For each, replace `#define FOO bar` with:
```c
#ifndef FOO
#define FOO bar
#endif
```

#### 2e. Guard ADCD3 references in AnalogIn.cpp

Apply the patch from `patches/AnalogIn_cpp.patch` or wrap all occurrences of `ADCD3` in `libraries/AP_HAL_ChibiOS/AnalogIn.cpp` with `#ifdef ADCD3` / `#endif` guards.

### 3. Create Board Directory

```bash
mkdir -p libraries/AP_HAL_ChibiOS/hwdef/WeActH743
```

Copy `hwdef.dat` and `hwdef-bl.dat` from this repository into that directory.

### 4. Build Bootloader

```bash
./waf configure --board WeActH743 --bootloader
./waf bootloader
```

### 5. Flash Bootloader via DFU

Put the board into DFU mode: hold **BOOT0**, press **RESET**, release RESET, release BOOT0.

```bash
dfu-util -a 0 -s 0x08000000:leave -D build/WeActH743/bin/AP_Bootloader.bin
```

Verify success — the LED on PE3 should blink after reset.

### 6. Build ArduCopter Firmware

```bash
./waf configure --board WeActH743
./waf copter
```

### 7. Flash Firmware via Bootloader

```bash
python3 Tools/scripts/uploader.py --port /dev/ttyACM0 build/WeActH743/bin/arducopter.apj
```

Or use QGroundControl: **Vehicle Setup → Firmware → Custom firmware** and select `arducopter.apj`.

---

## Flashing Workflow (After Initial Setup)

> **Important:** Do NOT use `arducopter_with_bl.hex` — flashing the combined file causes USB enumeration issues. Always flash bootloader and firmware separately.

```
Bootloader (once):  DFU → 0x08000000 → AP_Bootloader.bin
Firmware (updates): USB → ttyACM0   → arducopter.apj
```

---

## Connecting to QGroundControl

1. Power the board via USB
2. Bootloader blinks PE3 LED for ~5 seconds
3. ArduCopter starts — LED behaviour changes
4. QGroundControl auto-detects on `/dev/ttyACM0`
5. "Vehicle not ready" is normal until sensors are connected and calibrated

---

## Troubleshooting

### LED stays solid on, no USB enumeration (bootloader)

**Cause:** GCC 13.2.1 miscompiles the custom `strlen()` function in `Tools/AP_Bootloader/support.cpp` with `-O2`, causing a hard fault before the blink loop runs.

**Fix:** Add `__attribute__((optimize("O0")))` to the strlen function:
```cpp
__attribute__((optimize("O0"))) size_t strlen(const char *s1)
```

---

### LED never comes on at all (bootloader)

**Cause:** Binary flashed to wrong address. The STM32H743 internal flash starts at `0x08000000` — a common typo is `0x8000000` (7 digits) or `0x80000000` (wrong address entirely).

**Fix:** Verify the address in STM32CubeProgrammer character by character: `0x08000000` — exactly 8 hex digits after `0x`, starting with `08`.

---

### USB does not enumerate after bootloader blinks

**Causes and fixes:**

1. **Missing board.c USB reset** — the OTG peripheral retains state from the STM32 ROM DFU bootloader. Apply the `boardInit()` patch in `board.c` to reset it on startup.

2. **Wrong USB clock** — the STM32H743 needs exactly 48MHz on the USB clock. For a 25MHz crystal, the mcuconf 25MHz block uses PLL3Q which gives exactly 48MHz. Ensure `OSCILLATOR_HZ 25000000` is set and the mcuconf `#ifndef` guards are in place so the correct PLL values are used.

3. **VBUS sensing** — the WeAct board has no VBUS sense pin. Use `define BOARD_OTG_NOVBUSSENS 1` in hwdef (not `HAL_USB_FORCE_CONNECTED`).

4. **USB turnaround time** — H7 at high AHB frequency needs `TRDT_VALUE_FS 9` not `5`. Apply the `hal_usb_lld.c` patch.

---

### Build error: "STM32_USBSEL redefined"

**Cause:** `stm32h7_mcuconf.h` defines `STM32_USBSEL` without a `#ifndef` guard, conflicting with hwdef-generated `hwdef.h`.

**Fix:** Wrap the define in `stm32h7_mcuconf.h` with `#ifndef`/`#endif`. Do the same for all PLL1, PLL3, and VOS defines in the 25MHz block.

---

### Build error: "STM32_VOS_SCALE0 is not defined"

**Cause:** This ChibiOS version does not define `STM32_VOS_SCALE0` — only SCALE1/2/3 exist. VOS0 (480MHz boost) requires a different mechanism.

**Fix:** Do not set `STM32_VOS` in hwdef. The bootloader runs at 400MHz on VOS1 which is sufficient. The PLL3Q path gives 48MHz USB regardless.

---

### Build error: "STM32_ADC_ADC12_DMA_STREAM not defined"

**Cause:** Neither `STM32_ADC_ADC1_DMA_STREAM` nor `STM32_ADC_ADC2_DMA_STREAM` is defined for this board, so the conditional in `stm32h7_mcuconf.h` produces nothing.

**Fix:** Add to `hwdef.dat`:
```
define HAL_USE_ADC FALSE
```
This disables ADC entirely since no analog sensors are connected. Re-enable when adding battery voltage monitoring.

---

### Build error: "SERIAL driver activated but no USART/UART peripheral assigned"

**Cause:** `SERIAL_ORDER` includes `OTG1` but the ChibiOS serial driver still requires at least one physical UART to be defined.

**Fix:** Always include at least one UART in `hwdef.dat`:
```
SERIAL_ORDER OTG1 USART1
PA9  USART1_TX USART1
PA10 USART1_RX USART1
```

---

### Build error: mavlink version.h not found

**Cause:** MAVLink headers are generated during configure but the pymavlink submodule or headers are missing.

**Fix:**
```bash
git submodule sync --recursive
git submodule update --init --recursive --force
python3 -m pymavlink.tools.mavgen \
    --lang C --wire-protocol 2.0 \
    --output libraries/GCS_MAVLink/include/mavlink/v2.0 \
    modules/mavlink/message_definitions/v1.0/all.xml
```

---

### Firmware flashed but no USB / QGroundControl connection

**Cause:** Most likely the `arducopter_with_bl.hex` combined file was used, causing a conflict between the embedded bootloader and the manually flashed one.

**Fix:** Never use the combined `_with_bl.hex`. Flash bootloader via DFU once, then upload firmware via `.apj` only:
```bash
python3 Tools/scripts/uploader.py --port /dev/ttyACM0 build/WeActH743/bin/arducopter.apj
```

---

### QGroundControl shows "Vehicle not ready"

**Cause:** Normal — pre-arm checks are failing because sensors are not connected or calibrated.

**Fix:** Connect sensors and run calibration in QGroundControl under **Vehicle Setup**:
- Accelerometer calibration
- Compass calibration  
- Radio calibration (once RC receiver connected)

To bypass checks temporarily for bench testing:
`Parameters → ARMING_CHECK → 0`

---

## Files in This Repository

```
WeActH743/
├── hwdef.dat          # Main firmware hardware definition
├── hwdef-bl.dat       # Bootloader hardware definition
└── README.md          # This file

patches/
├── support_cpp.patch        # strlen GCC 13.2 fix
├── board_c.patch            # USB OTG reset + CRS init
├── hal_usb_lld_c.patch      # USB turnaround time
└── AnalogIn_cpp.patch       # ADCD3 guard (from WeAct H723 port)
```

---

## Credits

- WeAct H723 ArduPilot port by [Er-utpal](https://github.com/Er-utpal/WeAct723-Ardupilot) — several patches and insights directly applicable to the H743
- ArduPilot community porting documentation at [ardupilot.org/dev](https://ardupilot.org/dev/docs/porting.html)

---

## Licence

This board definition is released under the GNU General Public License v3.0, consistent with the ArduPilot project licence.
