# ArduPilot on WeAct Mini STM32H743VIT6

This project documents configuring ArduPilot on the WeAct Studio Mini STM32H743VIT6 development board as a custom quadcopter flight controller, including all patches required to get USB enumeration and the bootloader running correctly.

<img width="1440" height="1920" alt="ab2a0122-4928-4453-b431-a8e554d921f4" src="https://github.com/user-attachments/assets/4a2c7f74-c651-4bee-86f4-d2dba4b1fa38" />
<img width="1440" height="1920" alt="91e1c19b-4896-4ac7-87d8-c3fc123be029" src="https://github.com/user-attachments/assets/71a49211-18c6-4bb0-b609-b492d3b6570b" />

---

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

---

## Sensor Configuration

| Sensor | Interface | Pins | Notes |
|--------|-----------|------|-------|
| ICM42688 IMU | SPI1 | SCK=PA5, MISO=PA6, MOSI=PA7, CS=PB12, INT=PB0 | 4.7kΩ pullup on CS |
| BMP388 Barometer | I2C1 | SCL=PB8, SDA=PB9 | addr=0x77 (SDO to VCC) |
| BMM150 Compass | I2C1 | SCL=PB8, SDA=PB9 | addr=0x10–0x13 (see note) |

> **BMM150 I2C address** depends on the SDO/ADDR pin: GND=0x10, VCC=0x11, SDIO=0x12, SCK=0x13. Check your breakout board and update `hwdef.dat` accordingly.

> **I2C pullups** — both BMP388 and BMM150 share I2C1. Ensure 4.7kΩ pullups are present on SCL and SDA. Most breakout boards include these.

---

## Pinout

### Serial Ports

| Port | Function | TX Pin | RX Pin | Baud | Protocol |
|------|----------|--------|--------|------|----------|
| Serial0 | USB / GCS | PA11 | PA12 | — | MAVLink2 |
| Serial1 | GPS | PA9 | PA10 | 38400 | GPS auto |
| Serial2 | RC Receiver (iBUS) | PA2 | PA3 | 115200 | iBUS (RX only) |
| Serial3 | Jetson Orin Nano | PB10 | PB11 | 115200 | MAVLink2 |

### Motor Outputs

| Motor | Pin | Timer | Position |
|-------|-----|-------|----------|
| M1 | PE9 | TIM1_CH1 | Front-Right |
| M2 | PE11 | TIM1_CH2 | Rear-Left |
| M3 | PE13 | TIM1_CH3 | Front-Left |
| M4 | PE14 | TIM1_CH4 | Rear-Right |

### Other Connections

| Function | Pin | Notes |
|----------|-----|-------|
| Arming button | PE4 | Pull to GND, internal pullup enabled |
| Buzzer | PD14 | TIM4_CH3, active buzzer, + to pin, − to GND |
| SWDIO | PA13 | SWD debug |
| SWCLK | PA14 | SWD debug |

---

## Wiring Notes

### iBUS RC Receiver
iBUS is single-wire — only the RX pin (PA3) is used. TX (PA2) can be left unconnected. Connect the iBUS signal wire to PA3, VCC to 3.3V or 5V (depending on receiver), and GND to GND.

### Jetson Orin Nano Companion Computer
Connect via the 40-pin header UART (pins 8/10 = `/dev/ttyTHS0`):

```
Jetson pin 8  (TX) → FC PB11 (UART3 RX)
Jetson pin 10 (RX) → FC PB10 (UART3 TX)
Jetson pin 6  (GND) → FC GND
```

> **Important:** Share GND only. Do NOT connect 3.3V or 5V between the Jetson and flight controller — each has its own power supply.

### Motors / ESCs
ESCs run from a separate LiPo battery. Only the PWM signal wires (PE9/11/13/14) and a shared GND connect to the flight controller.

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

These patches are mandatory — without them the bootloader will crash and USB will not enumerate.

#### 2a. Fix GCC 13.2 strlen miscompilation (critical — causes bootloader hard fault)

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

Several defines in `libraries/AP_HAL_ChibiOS/hwdef/common/stm32h7_mcuconf.h` lack `#ifndef` guards, causing redefinition errors. Wrap each bare `#define` with:

```c
#ifndef FOO
#define FOO bar
#endif
```

Apply this to the following defines:

- `STM32_VOS` (line ~109)
- `STM32_PLL1_DIVM_VALUE`, `STM32_PLL1_DIVN_VALUE`, `STM32_PLL1_DIVP_VALUE`, `STM32_PLL1_DIVR_VALUE` (25MHz block)
- `STM32_PLL3_DIVN_VALUE`, `STM32_PLL3_DIVQ_VALUE`, `STM32_PLL3_DIVR_VALUE` (25MHz block)
- `STM32_USBSEL`
- `STM32_ADC_ADC12_DMA_STREAM`

#### 2e. Guard ADCD3 references in AnalogIn.cpp

Apply `patches/AnalogIn_cpp.patch` or wrap all occurrences of `ADCD3` in `libraries/AP_HAL_ChibiOS/AnalogIn.cpp` with `#ifdef ADCD3` / `#endif` guards.

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

Or via QGroundControl: **Vehicle Setup → Firmware → Custom firmware** and select `arducopter.apj`.

---

## Flashing Workflow (After Initial Setup)

> **Important:** Do NOT use `arducopter_with_bl.hex` — flashing the combined file causes USB enumeration issues. Always flash bootloader and firmware separately.

```
Bootloader (once):  DFU → 0x08000000 → AP_Bootloader.bin
Firmware (updates): USB → ttyACM0   → arducopter.apj
```

---

## QGroundControl Parameters

After flashing, set these parameters in QGroundControl under **Vehicle Setup → Parameters**:

### RC Receiver (iBUS)

| Parameter | Value | Description |
|-----------|-------|-------------|
| `SERIAL2_PROTOCOL` | 23 | RCInput |
| `SERIAL2_BAUD` | 115 | 115200 baud |
| `RC_PROTOCOLS` | 32 | iBUS |

### Companion Computer (Jetson Orin Nano)

| Parameter | Value | Description |
|-----------|-------|-------------|
| `SERIAL3_PROTOCOL` | 2 | MAVLink2 |
| `SERIAL3_BAUD` | 115 | 115200 baud |

### Buzzer

| Parameter | Value | Description |
|-----------|-------|-------------|
| `NOTIFY_BUZZ_ENABLE` | 1 | Enable buzzer |

### Arming Button

| Parameter | Value | Description |
|-----------|-------|-------------|
| `BTN_ENABLE` | 1 | Enable button |
| `BTN_FUNC1` | 41 | Arm/disarm toggle |

---

## Jetson Orin Nano Setup

### MAVProxy

```bash
pip3 install MAVProxy
mavproxy.py --master=/dev/ttyTHS0 --baudrate=115200 --console
```

### MAVSDK (Python)

```bash
pip3 install mavsdk
```

```python
import asyncio
from mavsdk import System

async def main():
    drone = System()
    await drone.connect(system_address="serial:///dev/ttyTHS0:115200")
    async for state in drone.core.connection_state():
        if state.is_connected:
            print("Connected to flight controller")
            break

asyncio.run(main())
```

### ROS2 with MAVROS

```bash
sudo apt install ros-humble-mavros ros-humble-mavros-extras
ros2 launch mavros apm.launch fcu_url:=/dev/ttyTHS0:115200
```

> **Note:** Use `/dev/ttyTHS0` or `/dev/ttyTHS1` depending on which UART header pins you use on the Orin Nano 40-pin connector.

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

2. **Wrong USB clock** — the STM32H743 needs exactly 48MHz on the USB clock. For a 25MHz crystal, the mcuconf 25MHz block uses PLL3Q which gives exactly 48MHz. Ensure `OSCILLATOR_HZ 25000000` is set and the mcuconf `#ifndef` guards are in place.

3. **VBUS sensing** — the WeAct board has no VBUS sense pin. Use `define BOARD_OTG_NOVBUSSENS 1` in hwdef (not `HAL_USB_FORCE_CONNECTED`).

4. **USB turnaround time** — H7 at high AHB frequency needs `TRDT_VALUE_FS 9` not `5`. Apply the `hal_usb_lld.c` patch.

---

### Build error: "STM32_USBSEL redefined"

**Cause:** `stm32h7_mcuconf.h` defines `STM32_USBSEL` without a `#ifndef` guard, conflicting with hwdef-generated `hwdef.h`.

**Fix:** Wrap the define in `stm32h7_mcuconf.h` with `#ifndef`/`#endif`. Do the same for all PLL1, PLL3, and VOS defines in the 25MHz block.

---

### Build error: "STM32_VOS_SCALE0 is not defined"

**Cause:** This ChibiOS version does not define `STM32_VOS_SCALE0` — only SCALE1/2/3 exist.

**Fix:** Do not set `STM32_VOS` in hwdef. The bootloader runs at 400MHz on VOS1 which is sufficient. PLL3Q gives 48MHz USB regardless.

---

### Build error: "STM32_ADC_ADC12_DMA_STREAM not defined"

**Cause:** Neither `STM32_ADC_ADC1_DMA_STREAM` nor `STM32_ADC_ADC2_DMA_STREAM` is defined for this board.

**Fix:** Add to `hwdef.dat`:
```
define HAL_USE_ADC FALSE
```
Re-enable when adding battery voltage monitoring.

---

### Build error: "SERIAL driver activated but no USART/UART peripheral assigned"

**Cause:** `SERIAL_ORDER` includes `OTG1` but no physical UART is defined.

**Fix:** Always include at least one UART:
```
SERIAL_ORDER OTG1 USART1
PA9  USART1_TX USART1
PA10 USART1_RX USART1
```

---

### Build error: mavlink version.h not found

**Cause:** MAVLink headers not generated.

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

**Cause:** The `arducopter_with_bl.hex` combined file was used.

**Fix:** Never use `_with_bl.hex`. Flash bootloader via DFU once, then upload firmware via `.apj`:
```bash
python3 Tools/scripts/uploader.py --port /dev/ttyACM0 build/WeActH743/bin/arducopter.apj
```

---

### INS: unable to initialise driver (IMU not detected)

**Causes and fixes:**

1. **CS pin floating** — PB12 needs a 4.7kΩ pullup to 3.3V if your breakout board lacks one.
2. **Wrong SPI mode** — ICM42688 uses MODE3. Verify in `hwdef.dat`: `SPIDEV icm42688 SPI1 DEVID1 ICM42688_CS MODE3 2*MHZ 16*MHZ`
3. **Wiring** — double-check PA5=SCK, PA6=MISO, PA7=MOSI, PB12=CS.
4. **Voltage** — ICM42688 logic must be 3.3V.

---

### Compass not detected (BMM150)

**Cause:** Wrong I2C address or missing pullups.

**Fix:** Verify the SDO/ADDR pin on your BMM150 module and set the matching address in `hwdef.dat`:

| SDO | Address |
|-----|---------|
| GND | 0x10 |
| VCC | 0x11 |
| SDIO | 0x12 |
| SCK | 0x13 |

Also ensure 4.7kΩ pullups are present on SCL (PB8) and SDA (PB9).

---

### QGroundControl shows "Vehicle not ready"

**Cause:** Normal — pre-arm checks failing because sensors are not connected or calibrated.

**Fix:** Connect sensors and run calibration under **Vehicle Setup**:
- Accelerometer calibration
- Compass calibration
- Radio calibration (once RC receiver connected)

To bypass temporarily for bench testing: `Parameters → ARMING_CHECK → 0`

---

### Jetson not receiving MAVLink data

**Causes and fixes:**

1. **TX/RX swapped** — Jetson TX must connect to FC PB11 (RX), Jetson RX to FC PB10 (TX).
2. **No shared GND** — ensure a GND wire connects Jetson and flight controller.
3. **Wrong serial port** — try `/dev/ttyTHS1` if `/dev/ttyTHS0` does not respond.
4. **Parameters not set** — confirm `SERIAL3_PROTOCOL=2` and `SERIAL3_BAUD=115` in QGC.

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
