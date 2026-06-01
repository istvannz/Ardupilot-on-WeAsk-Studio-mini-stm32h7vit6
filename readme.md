<img width="112" height="150" alt="weact_h743_wiring_diagram_jetson (1)" src="https://github.com/user-attachments/assets/20b0596c-c9e2-490a-a0ac-2494a33cfe15" /># ArduPilot on WeAct Mini STM32H743VIT6

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

### Wiring Diagram

![Uploa<svg width="100%" viewBox="0 0 687.51 920" role="img" style="" xmlns="http://www.w3.org/2000/svg">
  <title style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">WeAct Mini STM32H743VIT6 full wiring diagram with Jetson Orin Nano</title>
  <desc style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">Wiring diagram showing all connections including Jetson Orin Nano companion computer via USART3.</desc>
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  <mask id="imagine-text-gaps-eu23uq" maskUnits="userSpaceOnUse"><rect x="0" y="0" width="687.51" height="920" fill="white"/><rect x="300.7650451660156" y="327.9998779296875" width="78.4699478149414" height="20.000272750854492" fill="black" rx="2"/><rect x="289.7023620605469" y="346.9999084472656" width="100.92282104492188" height="18.0002384185791" fill="black" rx="2"/><rect x="224.00001525878906" y="378.9999084472656" width="79.39932250976562" height="18.0002384185791" fill="black" rx="2"/><rect x="224.00001525878906" y="400.9999084472656" width="57.30250549316406" height="18.0002384185791" fill="black" rx="2"/><rect x="224.00001525878906" y="422.9999084472656" width="69.62714767456055" height="18.0002384185791" fill="black" rx="2"/><rect x="224.00001525878906" y="444.9999084472656" width="89.16010284423828" height="18.0002384185791" fill="black" rx="2"/><rect x="224.00001525878906" y="466.9999084472656" width="92.15768432617188" height="18.0002384185791" fill="black" rx="2"/><rect x="224.00001525878906" y="488.9999084472656" width="95.82537841796875" height="18.0002384185791" fill="black" rx="2"/><rect x="224.00001525878906" y="510.9998779296875" width="57.953975677490234" height="18.0002384185791" fill="black" rx="2"/><rect x="224.00001525878906" y="532.9998779296875" width="72.6104736328125" height="18.0002384185791" fill="black" rx="2"/><rect x="376.248779296875" y="378.9999084472656" width="80.07813262939453" height="18.0002384185791" fill="black" rx="2"/><rect x="388.0771179199219" y="396.9999084472656" width="67.92289733886719" height="18.0002384185791" fill="black" rx="2"/><rect x="375.826904296875" y="422.9999084472656" width="80.17655181884766" height="18.0002384185791" fill="black" rx="2"/><rect x="386.71771240234375" y="444.9999084472656" width="69.6200065612793" height="18.0002384185791" fill="black" rx="2"/><rect x="42.97576141357422" y="31.999866485595703" width="102.04847717285156" height="20.000272750854492" fill="black" rx="2"/><rect x="-13.884583473205566" y="50.99988555908203" width="215.7705841064453" height="18.000234603881836" fill="black" rx="2"/><rect x="37.31160354614258" y="66.99988555908203" width="113.70285034179688" height="18.0002384185791" fill="black" rx="2"/><rect x="292.4680480957031" y="31.999866485595703" width="95.28502655029297" height="20.000272750854492" fill="black" rx="2"/><rect x="244.97500610351562" y="50.99988555908203" width="190.0499725341797" height="18.000234603881836" fill="black" rx="2"/><rect x="522.5380859375" y="31.999866485595703" width="126.92390441894531" height="20.000272750854492" fill="black" rx="2"/><rect x="490.97503662109375" y="50.99988555908203" width="190.0499725341797" height="18.000234603881836" fill="black" rx="2"/><rect x="177.3510284423828" y="190.99989318847656" width="73.29798889160156" height="18.0002384185791" fill="black" rx="2"/><rect x="50.350887298583984" y="211.9998779296875" width="87.5074462890625" height="20.000272750854492" fill="black" rx="2"/><rect x="6.85795783996582" y="230.99989318847656" width="174.61412048339844" height="18.0002384185791" fill="black" rx="2"/><rect x="44.53047561645508" y="391.9998779296875" width="99.27046203613281" height="20.000272750854492" fill="black" rx="2"/><rect x="31.608379364013672" y="410.9999084472656" width="124.78682708740234" height="18.0002384185791" fill="black" rx="2"/><rect x="62.36671447753906" y="428.9999084472656" width="63.61014175415039" height="18.0002384185791" fill="black" rx="2"/><rect x="54.1400146484375" y="442.9999084472656" width="80.05472564697266" height="18.0002384185791" fill="black" rx="2"/><rect x="166.34352111816406" y="431.9999084472656" width="35.312965393066406" height="18.0002384185791" fill="black" rx="2"/><rect x="35.58501052856445" y="545.9998779296875" width="117.04725646972656" height="20.000272750854492" fill="black" rx="2"/><rect x="35.14750289916992" y="564.9998779296875" width="118.37818145751953" height="18.0002384185791" fill="black" rx="2"/><rect x="33.60060119628906" y="582.9998779296875" width="120.8004150390625" height="18.0002384185791" fill="black" rx="2"/><rect x="29.819286346435547" y="600.9998779296875" width="128.3614273071289" height="18.0002384185791" fill="black" rx="2"/><rect x="38.71787643432617" y="614.9998779296875" width="110.89895629882812" height="18.0002384185791" fill="black" rx="2"/><rect x="130.03829956054688" y="618.9998779296875" width="99.9286880493164" height="18.0002384185791" fill="black" rx="2"/><rect x="170.4606170654297" y="577.9999389648438" width="47.407562255859375" height="18.0002384185791" fill="black" rx="2"/><rect x="45.686744689941406" y="669.9999389648438" width="96.62651062011719" height="20.000272750854492" fill="black" rx="2"/><rect x="34.72562026977539" y="688.9999389648438" width="118.88670349121094" height="18.0002384185791" fill="black" rx="2"/><rect x="289.3664245605469" y="689.9998779296875" width="101.2672119140625" height="20.000272750854492" fill="black" rx="2"/><rect x="267.8426208496094" y="708.9999389648438" width="144.65232849121094" height="18.0002384185791" fill="black" rx="2"/><rect x="540.4212036132812" y="307.9998779296875" width="91.15766906738281" height="20.000272750854492" fill="black" rx="2"/><rect x="548.4838256835938" y="326.9999084472656" width="75.35600280761719" height="18.0002384185791" fill="black" rx="2"/><rect x="542.0384521484375" y="358.9999084472656" width="88.58908081054688" height="18.0002384185791" fill="black" rx="2"/><rect x="536.483642578125" y="374.9999084472656" width="99.36026000976562" height="18.0002384185791" fill="black" rx="2"/><rect x="547.0307006835938" y="404.9999084472656" width="78.60581970214844" height="18.0002384185791" fill="black" rx="2"/><rect x="533.5929565429688" y="420.9999084472656" width="105.13562774658203" height="18.0002384185791" fill="black" rx="2"/><rect x="546.0307006835938" y="450.9999084472656" width="80.60133361816406" height="18.0002384185791" fill="black" rx="2"/><rect x="533.1476440429688" y="466.9999084472656" width="106.0255355834961" height="18.0002384185791" fill="black" rx="2"/><rect x="543.0306396484375" y="496.9999084472656" width="86.5936279296875" height="18.0002384185791" fill="black" rx="2"/><rect x="533.1476440429688" y="512.9999389648438" width="106.0255355834961" height="18.0002384185791" fill="black" rx="2"/><rect x="560.2418823242188" y="639.9998779296875" width="51.51636505126953" height="20.000272750854492" fill="black" rx="2"/><rect x="523.9365234375" y="658.9999389648438" width="124.1269760131836" height="18.0002384185791" fill="black" rx="2"/><rect x="517.4911499023438" y="721.9998779296875" width="137.017822265625" height="20.000272750854492" fill="black" rx="2"/><rect x="510.55352783203125" y="740.9999389648438" width="151.21331787109375" height="18.0002384185791" fill="black" rx="2"/><rect x="520.9833984375" y="756.9999389648438" width="130.35724639892578" height="18.0002384185791" fill="black" rx="2"/><rect x="523.4990234375" y="772.9998779296875" width="125.33665466308594" height="18.0002384185791" fill="black" rx="2"/><rect x="234.41232299804688" y="828.9999389648438" width="211.50526428222656" height="18.0002384185791" fill="black" rx="2"/><rect x="238.7483367919922" y="846.9999389648438" width="202.5057830810547" height="18.0002384185791" fill="black" rx="2"/></mask></defs>

  <!-- ── Central board ──────────────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="220" y="310" width="240" height="290" rx="12" stroke-width="1" style="fill:rgb(241, 239, 232);stroke:rgb(95, 94, 90);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="338" text-anchor="middle" dominant-baseline="central" style="fill:rgb(68, 68, 65);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">WeAct Mini</text>
    <text x="340" y="356" text-anchor="middle" dominant-baseline="central" style="fill:rgb(95, 94, 90);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">STM32H743VIT6</text>
  </g>

  <!-- Left pin labels -->
  <text x="228" y="388" text-anchor="start" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:start;dominant-baseline:central">PA5/6/7 SPI1</text>
  <text x="228" y="410" text-anchor="start" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:start;dominant-baseline:central">PB12 CS</text>
  <text x="228" y="432" text-anchor="start" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:start;dominant-baseline:central">PB8/9 I2C1</text>
  <text x="228" y="454" text-anchor="start" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:start;dominant-baseline:central">PA9/10 UART1</text>
  <text x="228" y="476" text-anchor="start" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:start;dominant-baseline:central">PA3 UART2 RX</text>
  <text x="228" y="498" text-anchor="start" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:start;dominant-baseline:central">PB10/11 UART3</text>
  <text x="228" y="520" text-anchor="start" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:start;dominant-baseline:central">PE4 BTN</text>
  <text x="228" y="542" text-anchor="start" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:start;dominant-baseline:central">PD14 BUZZ</text>

  <!-- Right pin labels -->
  <text x="452" y="388" text-anchor="end" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:end;dominant-baseline:central">PE9/11/13/14</text>
  <text x="452" y="406" text-anchor="end" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:end;dominant-baseline:central">TIM1 PWM</text>
  <text x="452" y="432" text-anchor="end" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:end;dominant-baseline:central">PA11/12 USB</text>
  <text x="452" y="454" text-anchor="end" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:end;dominant-baseline:central">3.3V / GND</text>

  <!-- ── ICM42688 IMU (top-left) ─────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="20" y="20" width="148" height="64" rx="8" stroke-width="0.5" style="fill:rgb(238, 237, 254);stroke:rgb(83, 74, 183);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="94" y="42" text-anchor="middle" dominant-baseline="central" style="fill:rgb(60, 52, 137);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">ICM42688 IMU</text>
    <text x="94" y="60" text-anchor="middle" dominant-baseline="central" style="fill:rgb(83, 74, 183);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">SPI1 · VCC GND SCK MOSI MISO CS</text>
    <text x="94" y="76" text-anchor="middle" dominant-baseline="central" style="fill:rgb(83, 74, 183);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">PA5/6/7 · CS=PB12</text>
  </g>
  <path d="M168 48 L205 48 L205 383 L220 388" fill="none" stroke="#7F77DD" stroke-width="1" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-eu23uq)" style="fill:none;stroke:rgb(127, 119, 221);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <path d="M168 60 L210 60 L210 405 L220 410" fill="none" stroke="#7F77DD" stroke-width="1" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-eu23uq)" style="fill:none;stroke:rgb(127, 119, 221);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- ── BMP388 (top-center) ─────────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="266" y="20" width="148" height="54" rx="8" stroke-width="0.5" style="fill:rgb(225, 245, 238);stroke:rgb(15, 110, 86);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="42" text-anchor="middle" dominant-baseline="central" style="fill:rgb(8, 80, 65);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">BMP388 Baro</text>
    <text x="340" y="60" text-anchor="middle" dominant-baseline="central" style="fill:rgb(15, 110, 86);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">I2C1 · 0x77 · VCC GND SCL SDA</text>
  </g>
  <path d="M340 74 L340 190 L212 190 L212 427 L220 432" fill="none" stroke="#1D9E75" stroke-width="1" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-eu23uq)" style="fill:none;stroke:rgb(29, 158, 117);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- ── BMM150 (top-right) ──────────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="512" y="20" width="148" height="54" rx="8" stroke-width="0.5" style="fill:rgb(225, 245, 238);stroke:rgb(15, 110, 86);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="586" y="42" text-anchor="middle" dominant-baseline="central" style="fill:rgb(8, 80, 65);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">BMM150 Compass</text>
    <text x="586" y="60" text-anchor="middle" dominant-baseline="central" style="fill:rgb(15, 110, 86);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">I2C1 · 0x10 · VCC GND SCL SDA</text>
  </g>
  <path d="M512 47 L216 47 L216 427 L220 432" fill="none" stroke="#1D9E75" stroke-width="1" stroke-dasharray="4 3" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-eu23uq)" style="fill:none;stroke:rgb(29, 158, 117);color:rgb(0, 0, 0);stroke-width:1px;stroke-dasharray:4px, 3px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <text x="214" y="200" text-anchor="middle" dominant-baseline="central" transform="rotate(-90,214,200)" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">I2C1 shared</text>

  <!-- ── GPS (left) ─────────────────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="20" y="200" width="148" height="54" rx="8" stroke-width="0.5" style="fill:rgb(230, 241, 251);stroke:rgb(24, 95, 165);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="94" y="222" text-anchor="middle" dominant-baseline="central" style="fill:rgb(12, 68, 124);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">GPS Module</text>
    <text x="94" y="240" text-anchor="middle" dominant-baseline="central" style="fill:rgb(24, 95, 165);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">UART1 · TX→PA10 · RX→PA9</text>
  </g>
  <path d="M168 227 L208 227 L208 449 L220 454" fill="none" stroke="#378ADD" stroke-width="1" marker-end="url(#arrow)" style="fill:none;stroke:rgb(55, 138, 221);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- ── iBUS Receiver (left) ───────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="20" y="380" width="148" height="74" rx="8" stroke-width="0.5" style="fill:rgb(230, 241, 251);stroke:rgb(24, 95, 165);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="94" y="402" text-anchor="middle" dominant-baseline="central" style="fill:rgb(12, 68, 124);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">iBUS Receiver</text>
    <text x="94" y="420" text-anchor="middle" dominant-baseline="central" style="fill:rgb(24, 95, 165);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">UART2 RX · PA3 only</text>
    <text x="94" y="438" text-anchor="middle" dominant-baseline="central" style="fill:rgb(24, 95, 165);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">VCC GND</text>
    <text x="94" y="452" text-anchor="middle" dominant-baseline="central" style="fill:rgb(24, 95, 165);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">signal → PA3</text>
  </g>
  <path d="M168 444 L206 444 L206 471 L220 476" fill="none" stroke="#185FA5" stroke-width="2" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-eu23uq)" style="fill:none;stroke:rgb(24, 95, 165);color:rgb(0, 0, 0);stroke-width:2px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <rect x="166" y="432" width="36" height="14" rx="3" fill="var(--color-background-info)" stroke="none" style="fill:rgb(214, 228, 246);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <text x="184" y="441" text-anchor="middle" dominant-baseline="central" style="fill:var(--color-text-info);fill:rgb(50, 102, 173);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">iBUS</text>

  <!-- ── Jetson Orin Nano (left-lower) ──────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="20" y="530" width="148" height="100" rx="8" stroke-width="0.5" style="fill:rgb(234, 243, 222);stroke:rgb(59, 109, 17);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="94" y="556" text-anchor="middle" dominant-baseline="central" style="fill:rgb(39, 80, 10);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Jetson Orin Nano</text>
    <text x="94" y="574" text-anchor="middle" dominant-baseline="central" style="fill:rgb(59, 109, 17);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">40-pin header UART</text>
    <text x="94" y="592" text-anchor="middle" dominant-baseline="central" style="fill:rgb(59, 109, 17);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Pin 8 TX → PB11 RX</text>
    <text x="94" y="610" text-anchor="middle" dominant-baseline="central" style="fill:rgb(59, 109, 17);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Pin 10 RX → PB10 TX</text>
    <text x="94" y="624" text-anchor="middle" dominant-baseline="central" style="fill:rgb(59, 109, 17);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Pin 6 GND → GND</text>
  </g>
  <!-- TX cross-connection: Jetson TX → FC RX -->
  <path d="M168 592 L206 592 L206 493 L220 498" fill="none" stroke="#3B6D11" stroke-width="1.5" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-eu23uq)" style="fill:none;stroke:rgb(59, 109, 17);color:rgb(0, 0, 0);stroke-width:1.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <!-- RX cross-connection: Jetson RX ← FC TX -->
  <path d="M168 610 L202 610 L202 503 L220 498" fill="none" stroke="#3B6D11" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-eu23uq)" style="fill:none;stroke:rgb(59, 109, 17);color:rgb(0, 0, 0);stroke-width:1.5px;stroke-dasharray:5px, 3px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <!-- GND note -->
  <text x="180" y="628" text-anchor="middle" dominant-baseline="central" style="fill:var(--color-text-secondary);fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">shared GND only</text>
  <!-- UART3 label -->
  <rect x="174" y="578" width="40" height="14" rx="3" fill="var(--color-background-success)" stroke="none" style="fill:rgb(233, 241, 220);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <text x="194" y="587" text-anchor="middle" dominant-baseline="central" style="fill:var(--color-text-success);fill:rgb(38, 91, 25);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">UART3</text>

  <!-- ── Arming Button ──────────────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="20" y="660" width="148" height="50" rx="8" stroke-width="0.5" style="fill:rgb(250, 238, 218);stroke:rgb(133, 79, 11);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="94" y="680" text-anchor="middle" dominant-baseline="central" style="fill:rgb(99, 56, 6);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Arming Button</text>
    <text x="94" y="698" text-anchor="middle" dominant-baseline="central" style="fill:rgb(133, 79, 11);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">PE4 → BTN → GND</text>
  </g>
  <path d="M168 682 L208 682 L208 515 L220 520" fill="none" stroke="#BA7517" stroke-width="1" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-eu23uq)" style="fill:none;stroke:rgb(186, 117, 23);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- ── Buzzer ──────────────────────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="266" y="680" width="148" height="50" rx="8" stroke-width="0.5" style="fill:rgb(250, 238, 218);stroke:rgb(133, 79, 11);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="340" y="700" text-anchor="middle" dominant-baseline="central" style="fill:rgb(99, 56, 6);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Buzzer (active)</text>
    <text x="340" y="718" text-anchor="middle" dominant-baseline="central" style="fill:rgb(133, 79, 11);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">PD14 TIM4_CH3 → GND</text>
  </g>
  <path d="M340 680 L340 608 L220 542" fill="none" stroke="#BA7517" stroke-width="1" marker-end="url(#arrow)" mask="url(#imagine-text-gaps-eu23uq)" style="fill:none;stroke:rgb(186, 117, 23);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- ── Motors (right) ─────────────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="512" y="290" width="148" height="310" rx="8" stroke-width="0.5" style="fill:rgb(250, 236, 231);stroke:rgb(153, 60, 29);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="586" y="318" text-anchor="middle" dominant-baseline="central" style="fill:rgb(113, 43, 19);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">ESC / Motors</text>
    <text x="586" y="336" text-anchor="middle" dominant-baseline="central" style="fill:rgb(153, 60, 29);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">PWM · TIM1</text>
    <text x="586" y="368" text-anchor="middle" dominant-baseline="central" style="fill:rgb(153, 60, 29);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">M1 Front-Right</text>
    <text x="586" y="384" text-anchor="middle" dominant-baseline="central" style="fill:rgb(153, 60, 29);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">PE9 · TIM1_CH1</text>
    <text x="586" y="414" text-anchor="middle" dominant-baseline="central" style="fill:rgb(153, 60, 29);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">M2 Rear-Left</text>
    <text x="586" y="430" text-anchor="middle" dominant-baseline="central" style="fill:rgb(153, 60, 29);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">PE11 · TIM1_CH2</text>
    <text x="586" y="460" text-anchor="middle" dominant-baseline="central" style="fill:rgb(153, 60, 29);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">M3 Front-Left</text>
    <text x="586" y="476" text-anchor="middle" dominant-baseline="central" style="fill:rgb(153, 60, 29);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">PE13 · TIM1_CH3</text>
    <text x="586" y="506" text-anchor="middle" dominant-baseline="central" style="fill:rgb(153, 60, 29);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">M4 Rear-Right</text>
    <text x="586" y="522" text-anchor="middle" dominant-baseline="central" style="fill:rgb(153, 60, 29);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">PE14 · TIM1_CH4</text>
  </g>
  <path d="M460 388 L512 388" fill="none" stroke="#D85A30" stroke-width="1" marker-end="url(#arrow)" style="fill:none;stroke:rgb(216, 90, 48);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <path d="M460 425 L512 425" fill="none" stroke="#D85A30" stroke-width="1" marker-end="url(#arrow)" style="fill:none;stroke:rgb(216, 90, 48);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <path d="M460 462 L512 462" fill="none" stroke="#D85A30" stroke-width="1" marker-end="url(#arrow)" style="fill:none;stroke:rgb(216, 90, 48);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <path d="M460 498 L512 498" fill="none" stroke="#D85A30" stroke-width="1" marker-end="url(#arrow)" style="fill:none;stroke:rgb(216, 90, 48);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- ── USB ────────────────────────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="512" y="630" width="148" height="50" rx="8" stroke-width="0.5" style="fill:rgb(241, 239, 232);stroke:rgb(95, 94, 90);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="586" y="650" text-anchor="middle" dominant-baseline="central" style="fill:rgb(68, 68, 65);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">USB-C</text>
    <text x="586" y="668" text-anchor="middle" dominant-baseline="central" style="fill:rgb(95, 94, 90);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">PA11/12 · GCS / flash</text>
  </g>
  <path d="M460 432 L484 432 L484 653 L512 653" fill="none" stroke="#888780" stroke-width="1" marker-end="url(#arrow)" style="fill:none;stroke:rgb(136, 135, 128);color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>

  <!-- ── QGC params ─────────────────────────────────────────────────────── -->
  <g style="fill:rgb(0, 0, 0);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto">
    <rect x="512" y="710" width="148" height="86" rx="8" stroke-width="0.5" style="fill:rgb(234, 243, 222);stroke:rgb(59, 109, 17);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
    <text x="586" y="732" text-anchor="middle" dominant-baseline="central" style="fill:rgb(39, 80, 10);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:14px;font-weight:500;text-anchor:middle;dominant-baseline:central">Jetson QGC Params</text>
    <text x="586" y="750" text-anchor="middle" dominant-baseline="central" style="fill:rgb(59, 109, 17);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">SERIAL3_PROTOCOL = 2</text>
    <text x="586" y="766" text-anchor="middle" dominant-baseline="central" style="fill:rgb(59, 109, 17);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">SERIAL3_BAUD = 115</text>
    <text x="586" y="782" text-anchor="middle" dominant-baseline="central" style="fill:rgb(59, 109, 17);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">/dev/ttyTHS0 or THS1</text>
  </g>

  <!-- ── Power note ─────────────────────────────────────────────────────── -->
  <rect x="220" y="820" width="240" height="44" rx="8" stroke-width="0.5" style="fill:rgb(245, 244, 237);stroke:rgba(31, 30, 29, 0.3);color:rgb(0, 0, 0);stroke-width:0.5px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:16px;font-weight:400;text-anchor:start;dominant-baseline:auto"/>
  <text x="340" y="838" text-anchor="middle" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Sensors: 3.3V · Motors: separate LiPo</text>
  <text x="340" y="856" text-anchor="middle" dominant-baseline="central" style="fill:rgb(61, 61, 58);stroke:none;color:rgb(0, 0, 0);stroke-width:1px;stroke-linecap:butt;stroke-linejoin:miter;opacity:1;font-family:&quot;Anthropic Sans&quot;, -apple-system, BlinkMacSystemFont, &quot;Segoe UI&quot;, sans-serif;font-size:12px;font-weight:400;text-anchor:middle;dominant-baseline:central">Jetson: own PSU · shared GND only</text>

</svg>ding weact_h743_wiring_diagram_jetson (1).svg…]()


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
