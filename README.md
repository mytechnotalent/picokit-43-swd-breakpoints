![picokit-43-swd-breakpoints](https://raw.githubusercontent.com/mytechnotalent/picokit-43-swd-breakpoints/main/picokit-43-swd-breakpoints.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-43 SWD BREAKPOINTS

### Debug Probe Breakpoints and SWD Variable Reads
#### Lesson 43 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

<br>

The forty-third Picokit lesson. The node advances a counter once per
step interval and seals it into an authenticated heartbeat. On top of the
normal lesson, the Debug Probe breakpoint lab halts on the counter function and
reads the counter variable over SWD while the core is stopped.

<br>

## What it teaches

<br>

- A paced counter state machine with a one-shot onboard blink.
- Sealing the counter into an authenticated LoRa heartbeat.
- Setting a hardware breakpoint on a known function with the Debug Probe.
- Reading a live variable over SWD while the core is halted.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| Red / Yellow / Green | GP16 / GP18 / GP17 | annunciator status |
| Onboard LED | GP25 | heartbeat, one blink per transmit |
| RYLR998 | GP8 TX / GP9 RX | LoRa heartbeat |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

<br>

The node runs `monitor_step` in a loop. Every 2 seconds it advances
`g_count`, and every 5 seconds it seals `{"n":43,"s":<seq>,"c":<count>}` with
the shared field key and sends it over LoRa.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_43_swd_breakpoints verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
=== PICOKIT-43 SWD BREAKPOINTS // COUNTER + DEBUG LAB ===
BREAKPOINT n=43 count=1 seq=1
BREAKPOINT n=43 count=3 seq=2
RX from 0x0001, N bytes
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=43 rssi=...` per authenticated heartbeat. The terminal
dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Debug lab: breakpoint and read a variable over SWD

The Debug Probe attaches to the RP2350 with CMSIS-DAP and OpenOCD. Build with
the default debug info, start OpenOCD, and attach GDB:

```bash
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg
arm-none-eabi-gdb build/picokit_43_swd_breakpoints.elf
(gdb) target extended-remote localhost:3333
(gdb) monitor reset halt
(gdb) break monitor_count_tick
(gdb) continue
```

Each time the breakpoint fires the core halts before the counter advances.
Read the counter variable over SWD:

```text
(gdb) print g_count
$1 = 7
(gdb) continue
```

The same value is sealed into the heartbeat body
`{"n":43,"s":<seq>,"c":<count>}` and shown on the console as
`BREAKPOINT n=43 count=7 seq=2`.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-44-swd-watchpoints](https://github.com/mytechnotalent/picokit-44-swd-watchpoints)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-43-swd-breakpoints/blob/main/LICENSE)
