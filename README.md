# P2P Bot Kit hardware: the ESP32 build (Rust)

A bot box for about a third of the price of a Raspberry Pi build. It is a
microcontroller board with both Ethernet and Wi-Fi, running the Rust port
of the [Saturn P2P bot kit](https://github.com/Infinite-Star-Studios/p2pbotkit)
(branch `Rust`, folder `rust/`) instead of the TypeScript kit. There is no
circuit board to design or print, and no operating system on it.

**Recommendation: a Waveshare ESP32-S3-ETH board. The parts cost about
$30–50, or about $20–30 if you already have a USB-C charger and cables.**

Prices are US prices checked on 17 November 2026, so confirm them when you
order. The Raspberry Pi build is on this repository's `RaspberryPi` branch.

> **Status: the hardware is ready to buy, but the firmware is not written
> yet.**
>
> - **What exists:** the Rust port's first stage. Its core reproduces every
>   byte the kit signs on Phantasma and Base, and it builds without an
>   operating system for a bare-metal RISC-V target.
> - **What is still missing:** the checks, the chain readers, the board
>   client, the store's journal and the trading flows. These are stages 1–6
>   of the port's roadmap, and stage 7 is this board's firmware (see
>   `rust/README.md` on the kit's `Rust` branch).
> - **Not yet tried:** a build for this board's Xtensa chip.
>
> Until the firmware exists, trade with the TypeScript kit on the Raspberry
> Pi build. Buy one of these boards now only for firmware development.

## Why this board

| What the bot needs | What this board has |
|---|---|
| Ethernet and Wi-Fi | A W5500 chip (10/100 Ethernet, RJ45) and 2.4 GHz Wi-Fi on the ESP32-S3 |
| Several HTTPS connections at once (the board's long-polls, plus the Phantasma and Base RPCs) | 8 MB of PSRAM next to 512 KB of SRAM, enough for the TLS buffers (tens of KB each) |
| A crash-safe store: every step written and read back before anything is signed | 16 MB of flash, with room for a power-loss-safe file system (LittleFS) for the store |
| Keys protected while stored | Flash encryption (AES-XTS) with its key in the chip's eFuses, usable only by the hardware, plus Secure Boot V2 (RSA-3072) |
| Truly random swap secrets and control tokens | A hardware random number generator |
| A clock within 300 s of Base, or the bot refuses to sign | Network time (SNTP) at start-up. There is no clock battery, but the bot can't trade offline anyway |
| Running for days | It draws a watt or two over USB-C |

The chip is an ESP32-S3R8, a dual-core 240 MHz processor. The board also
has USB-C (native USB, for flashing and power), a microSD slot and a
Pico-style 40-pin header. It comes in four versions: board only, with a
camera kit, with PoE, and with PoE and a camera kit. The bot needs no
camera.

## Shopping list

| # | Part | Why | Price |
|---|---|---|---|
| 1 | **Waveshare ESP32-S3-ETH** (the board-only version) | All of the above, on one board | ~$17–25 |
| 2 | **USB-C power supply**, 5 V, 1 A or more (a good phone charger) | Power. The board needs little | ~$5–8 |
| 3 | **USB-C cable that carries data** | Flashing the firmware, and power | ~$3–5 |
| 4 | **Small enclosure**: an ABS project box with cut-outs for the RJ45 and USB-C, or a 3D-printed case | Keeps dust and fingers off it | ~$3–5 |
| 5 | **Ethernet patch cable** | Wired is the main connection and Wi-Fi is the backup | ~$3–5 |
| | **Total** | | **≈ $30–50** |

**Optional:**

- **PoE.** The PoE version of the board runs on the Ethernet cable alone, so
  there is no separate power supply. It needs a PoE switch, or an 802.3af
  injector (~$10–20).
- **Power backup.** It needs so little power that you can plug its charger
  into your router's UPS, or run it from a USB power bank with pass-through
  charging.
- **No microSD card.** Keep the store in the internal flash, which flash
  encryption covers. A microSD card would not be covered.

Compared with the Raspberry Pi build (about $90–100 for the parts), this
saves about $40–70 per box. The catch is the firmware still to be written,
above.

## Assembly

1. If you bought the PoE version, screw the PoE module onto the board.
2. Put the board in its case, with the RJ45 and USB-C ports reachable.
3. Plug in Ethernet, then USB-C power, or the Ethernet cable alone with PoE.

To flash firmware once it exists, use the Rust ESP32 toolchain on a
computer:

```sh
cargo install espup && espup install
cargo install espflash ldproxy
```

Then `espflash flash --monitor <firmware>` over the USB-C cable.

## Setup for safety

The same rules as the Raspberry Pi build apply. A bot's keys are hot
keys: keep only what it trades on its addresses, and sweep profits to a
wallet the bot has no key for. On this board, these steps replace disk
encryption and the firewall:

1. **Make the keys on a computer, and back them up first.**
   `p2pbot new-keys --dir keys/mainnet --name market` (from the kit's
   `Rust` branch) writes the same key files the TypeScript kit uses. Back
   them up offline, then load them onto the board once over USB.
2. **Production firmware runs with flash encryption in release mode, and
   Secure Boot V2.**
   - Development mode does not stop someone with the board from reading
     the flash.
   - In release mode, the board's flash, keys included, can't be read or
     replaced without its eFuse key.
3. **Burning eFuses is permanent, so plan it first.**
   - Plan which eFuse key blocks get read protection before burning
     anything.
   - Keep the secure-boot signing key offline. If you lose it, the boards
     take no more updates. If it leaks, anyone can sign firmware for them.
4. **Nothing on the board listens on the network.** The firmware only makes
   outbound HTTPS connections, to the board's API and to the pinned
   Phantasma and Base RPCs. There is no web page, no remote shell and no
   open port to attack.
5. **Keep secrets in encrypted storage only.** Wi-Fi credentials and keys go
   in encrypted NVS or flash, never on the microSD card.
6. **Expect it to reboot by itself, on purpose.** Use Ethernet first and
   Wi-Fi as backup, and a watchdog that restarts the board if it hangs. The
   kit's store is built to resume every trade after a restart.

## A Linux board instead

The Rust port's roadmap reaches Linux before the ESP32, so a small Linux
board can run the Rust bot sooner. An example is the Luckfox Pico Ultra W
(~$20–30): 256 MB of RAM, 8 GB of built-in storage, Ethernet and
Wi-Fi 6. It costs about the same as this build, but it is a Linux computer
to look after, as the Raspberry Pi build is.

## Sources

- [Waveshare ESP32-S3-ETH documentation](https://docs.waveshare.com/ESP32-S3-ETH) and [product page](https://www.waveshare.com/esp32-s3-eth.htm)
- [CNX Software on the ESP32-S3-ETH](https://www.cnx-software.com/2024/10/28/waveshare-esp32-s3-eth-board-provides-ethernet-and-camera-connectors-supports-raspberry-pi-pico-hats/) (variants and store prices) · [Tinytronics listing](https://www.tinytronics.nl/en/development-boards/microcontroller-boards/with-wi-fi/waveshare-esp32-s3-eth-ethernet-development-board-with-wi-fi-and-bluetooth) · [Electrokit listing](https://www.electrokit.com/en/development-board-esp32-s3-ethernet)
- [ESP-IDF flash encryption for the ESP32-S3](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/security/flash-encryption.html) · [Espressif on flash encryption and secure boot](https://developer.espressif.com/blog/flash-encryption-and-secure-boot/)
- [Espressif's esp-hal 1.0 (Rust)](https://developer.espressif.com/blog/2025/10/esp-hal-1/)
- [Luckfox Pico Ultra (CNX Software)](https://www.cnx-software.com/2024/06/21/luckfox-pico-ultra-micro-development-board-features-mipi-csi-wifi-6-ethernet-poe-gpio-headers/)
