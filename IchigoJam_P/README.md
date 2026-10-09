# IchigoJam P

## Setup
- Install **CMake** and **GCC**  

- Place the following libraries directly under the `IchigoJam_P` directory:  
    - **IchigoJam_BASIC**: Copy the `IchigoJam_BASIC` directory located one level up  
    - **pico-sdk**: https://github.com/raspberrypi/pico-sdk.git (latest release, 2.3.1, commit `079c6f3`) — update submodules ``` git submodule update --init --recursive ```  
    - **pico-extras**: https://github.com/raspberrypi/pico-extras.git (master, commit `52fd7a7`)  
    - **PicoDVI**: https://github.com/mlorenzati/PicoDVI.git (master; verified with commit `9eca921`)  

- Apply the PicoDVI patch (required for stable DVI/NTSC switching with the `SWITCH` command):  
    In the `PicoDVI` directory, apply `patches/picodvi_dvi_stop_race_fix.patch`:  
    ```sh
    cd PicoDVI  
    git apply ../patches/picodvi_dvi_stop_race_fix.patch  
    ```
    - Fixes a race where the DMA IRQ handler hangs waiting for channels that `dvi_stop()` has already aborted (interrupts while stopped are ignored)  

- Apply the TinyUSB patch (required for the USB keyboard):  
    Edit `pico-sdk/lib/tinyusb/src/osal/osal_pico.h` and replace the body of `osal_task_delay()` with a busy loop:  
    ```c
    TU_ATTR_ALWAYS_INLINE static inline void osal_task_delay(uint32_t msec) {
        // sleep_ms(msec); // called from timer interrupt on IchigoJam P
        uint64_t start = time_us_64();
        while (time_us_64() - start < msec * 1000) {
            ;
        }
    }
    ```

- Set the following paths:  
    ```sh
    PICO_SDK_PATH=foo/IchigoJam_P/pico-sdk  
    PICO_EXTRAS_PATH=foo/IchigoJam_P/pico-extras  
    ```

Refer to the following document for detailed instructions:  
[Getting Started with Raspberry Pi Pico](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf)  

## Build
```sh
cd IchigoJam_P  
mkdir build  
cd build  
cmake ..  
make  
```

Write `IchigoJam_P.uf2` file to the Pico device

## 4K version extensions (based on GivetakeJam_P)
- **Program area expanded from 1024 to 4096 bytes** (LIST moved to `#E00`; check free space with `FREE`)
- **Extended array variables**: `[102]`..`[357]` in addition to `[0]`..`[101]` (VAR2 area at `#C00`-#DFF)
    - Usable with `FOR`/`NEXT`, `LET` and `INPUT` as well
- **Storage**: 25 programs in internal flash (changed from 100 due to 4096-byte programs; `FILES` shows 0-24)
- **External EEPROM**: file numbers 100-131 (4096 bytes per slot). Supports 24LC64/24LC256/24FC1025 with a single binary; safer 32-byte page writes
    - `FILES0` auto-detects the EEPROM and shows it if present
- **`WS.LED port,n[,repeat]`** WS2812B LED strip control (port-selectable)
    - port: 1-7 (OUT1-6, 7=LED/GP2), n: number of LEDs, repeat: send count
    - Color data is read from array variables `[0]`.. (3 vars per LED; up to **119 LEDs** with extended arrays)
    - Sends with correct timing in both DVI (252 MHz) and NTSC (157.5 MHz) modes
- **`IR.IN port,[n]`** NEC infrared receive (connect an HX1838-type module to IN1-4)
    - [n+0]..[n+3]=received code, [n+4]=repeat flag, [n+5]=error code, [n+6]=mode (0=standard NEC / 1=NEC extended)
- **`ENV.IN [n]`** environment sensor read (connect AHT20 + BMP280 via I2C)
    - [n+0]=temperature 0.1C, [n+1]=humidity 0.1%RH, [n+2]=pressure 0.1hPa, [n+3]=BMP280 temperature, [n+4]=error, [n+5]=status flags (works with AHT20 alone)

### Notes
- The LIST area moved from `#C00` to `#E00`; existing programs that access RAM directly (e.g. `POKE #C00`) behave differently
- The 4K implementation references [GivetakeJam_P](https://github.com/Canon-boy4/ichigojam-firm) (MIT license, Copyright 2014-2025 the IchigoJam authors)

## Video output
- **DVI (HDMI)** and **NTSC composite** are supported. Switch with the `SWITCH` command (also on F10 key):  
    - `SWITCH 0` ... DVI (HDMI)  
    - `SWITCH 1` ... NTSC composite  
    - `SWITCH` ... toggle  
- The selected mode is saved to flash and restored on the next boot  
- The NTSC screen shows 32x24 characters filling the whole screen  
- NTSC signal is output from **GP3** (pin 5). Connect it to a video input via a resistor/capacitor network:  
    ```
    GP3 -- 75 ohm --- Video Out
                  |
                 === 560 pF
                  |
                 GND
    ```
- NTSC mode runs the CPU at 157.5 MHz, so BASIC executes a bit slower than DVI mode (still faster than the original IchigoJam)  
- In NTSC mode the on-board LED blinks at about 1 Hz (activity indicator)  
    - Fast blinking ... internal error detected  
    - Blinking stopped ... system halted. The watchdog reboots it automatically in about 8 seconds  
- When turning the power off, **wait 5 seconds or more** before turning it back on (due to the RP2040 hardware, a brief power interruption may leave it halted without rebooting. It also recovers automatically in about 8 seconds in that case)  

## Troubleshooting
If something goes wrong, delete the `build` directory and start over  
This is particularly effective if video output issues occur  

If the screen shows "E" followed by a number, an internal state error was detected (1:font pointer 2:VRAM pointer 4:video mode 8:font RAM boundary 16:DMA buffer boundary 32:scanline counter)  

## License
- Copyright 2014-2025 the IchigoJam authors. All rights reserved. MIT license. https://ichigojam.net/license

## Trademark
IchigoJam is a registered trademark of jig.jp co., ltd.  
Raspberry Pi is a registered trademark of the Raspberry Pi Foundation  

## Terms of use
Please download this software only if you agree to the terms of use. Redistribution to third parties requires a license. See https://pcn.club/sp/ijp/ for details.
- Terms of use (IchigoJam royalty-free program license, in Japanese): https://ichigojam.net/ichigojam-license.pdf

NaturalStyle co., ltd.  
Created on 2024-12-25
