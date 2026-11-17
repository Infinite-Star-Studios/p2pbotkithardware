# P2P Bot Kit hardware

A small, cheap computer that does one job: run the
[Saturn P2P bot kit](https://github.com/Infinite-Star-Studios/p2pbotkit)'s
market bot around the clock, with both Ethernet and Wi-Fi, assembled from
off-the-shelf parts (no circuit board to design or print).

**Recommendation: a Raspberry Pi 5 with 1 GB of RAM in the official case.
The parts cost about $90–100, plus about $85–100 for a UPS for mainnet if
you don't already have one.**

Prices are US prices from approved resellers, checked on 17 November 2026.
Raspberry Pi raised prices three times in 2026 because of the RAM shortage,
so confirm them when you order.

## Why not a custom chip?

The kit is about 15,000 lines of TypeScript, plus 12,000 lines of tests.
It needs Linux (or macOS), Node.js 22 or later, roughly 250 MB of RAM and
350 MB of disk. That rules out the alternatives:

- A microcontroller such as an ESP32 has less than 1 MB of RAM and cannot
  run it.
- Porting the kit to a microcontroller would mean rewriting and re-auditing
  the code that checks every signature.
- A custom chip costs millions before the first unit is made.

The cheapest "custom" build is a small computer assembled around a chip that
already exists, in this case a Raspberry Pi's.

## What the bot needs

These requirements come from p2pbotkit 0.3.0 (commit `b46ecaa`).

| Need | Source in the kit | What it means for the hardware |
|---|---|---|
| Linux and Node 22 or later, on 64-bit x86, 64-bit ARM or 32-bit ARM; no Windows | [`start.sh`](https://github.com/Infinite-Star-Studios/p2pbotkit/blob/main/start.sh) downloads Node 22 for `x64`, `arm64` and `armv7l` | Any Raspberry Pi-class board works. Use 64-bit Raspberry Pi OS |
| About 175 MB of RAM once loaded and about 260 MB at peak, per bot | Measured ([below](#memory-measured)) | 1 GB is plenty for one mainnet market bot. Devnet runs two bots |
| About 130 MB of packages, plus about 200 MB if it downloads Node | Kit README, "Good to know" | A 32 GB card is far more than enough |
| The clock must be within 300 s of Base's newest block, or the kit refuses to sign | [`docs/CHECKS.md`](https://github.com/Infinite-Star-Studios/p2pbotkit/blob/main/docs/CHECKS.md), check 25 | Network time sync, plus a battery-backed clock (built into the Pi 5) |
| Every state change is fsynced and read back before anything is signed | [`src/store.ts`](https://github.com/Infinite-Star-Studios/p2pbotkit/blob/main/src/store.ts), [`src/signer.ts`](https://github.com/Infinite-Star-Studios/p2pbotkit/blob/main/src/signer.ts) | Steady power and storage that keeps its writes: the official power supply, a good card and a UPS |
| Mainnet trades can take up to about 12 hours. A lock not sent in time is a no-show, and 3 no-shows in 24 hours pause the bot for 24 hours | Kit README, "Stopping and starting again" and "API keys and limits" | Keep it running: a UPS, and a wired network with Wi-Fi as backup |
| Keys are plain JSON files (mode 0600); there is no hardware-wallet support | [`src/keys.ts`](https://github.com/Infinite-Star-Studios/p2pbotkit/blob/main/src/keys.ts) | The box's storage holds the keys: use a dedicated device, encrypt the storage and keep balances small |
| It only makes outbound HTTPS connections, to the board, a Phantasma node and a Base RPC | Kit README, "The safety model" | It needs no open ports, so block all incoming traffic |

### Memory, measured

These numbers are from x86-64 Linux with Node 22.23.4. They have not been
measured on a Pi yet, but 64-bit ARM should be similar.

- **Loading the bot without starting it** (`src/index.ts` and
  `examples/market-bot.ts`): 157 MB resident, plus 17 MB for tsx's esbuild
  helper.
- **The kit's own test suite**
  (`node --import tsx --test --test-concurrency=1 test/*.test.ts`): 257 MB at
  peak in the busiest process. The suite runs the maker, taker and instant
  flows against fake chains. 376 tests passed and 17 were skipped, as the kit
  README says.

## Shopping list

Buy from [Raspberry Pi approved resellers](https://www.raspberrypi.com/resellers/),
such as CanaKit, Adafruit, PiShop.us or Micro Center. Marketplace listings
often charge two or three times the price, and fake SD cards are common
there.

### Base build (required)

| # | Part | Why this one | Price |
|---|---|---|---|
| 1 | **Raspberry Pi 5, 1 GB** | It has Gigabit Ethernet and dual-band Wi-Fi built in, a real-time clock, a PCIe connector for an SSD later, and secure boot. The 1 GB model has kept its $45 price through every 2026 increase | $45 |
| 2 | **Raspberry Pi 27W USB-C Power Supply** | Too little voltage is a common cause of SD card corruption. The official 5.1 V / 5 A supply avoids it | ~$13 |
| 3 | **Raspberry Pi Case for Raspberry Pi 5** | It has a built-in fan for round-the-clock running, and the compact SSD board (below) fits inside it | ~$11 |
| 4 | **microSD card, 32–64 GB, high endurance**: SanDisk High Endurance, Samsung PRO Endurance, or Raspberry Pi's own A2 card | Made for continuous writing | ~$10–20 |
| 5 | **Raspberry Pi RTC Battery** | Keeps the Pi 5's clock running through power cuts | ~$5–7 |
| 6 | **Cat6 Ethernet patch cable** | Wired is the main connection and Wi-Fi is the backup | ~$5 |
| | **Total** | | **≈ $90–100** |

You don't need a monitor, keyboard or mouse, because you set it up over
SSH. You only need a microSD card reader (~$10) if your computer has no SD
slot.

### Recommended for mainnet: a UPS

| Part | Why | Price |
|---|---|---|
| **A small UPS for the Pi and your router or modem**, with a USB data ("management") port, e.g. APC Back-UPS BE600M1 (600 VA). Check the listing for the data port | It keeps the Pi and your internet connection running through power cuts, so trades keep moving. Over USB it can tell the Pi to shut down cleanly (with NUT) when the battery runs low | ~$85–100 |

If you already have a UPS for your router, plug the Pi into it for free.

The compact alternative is a UPS board that mounts on the Pi, such as the
Geekworm X1200 (~$40, plus two 18650 cells). It keeps only the Pi running,
not the router, and it needs a case made for it instead of the official
case.

### Optional: an SSD instead of the microSD card

| Part | Price |
|---|---|
| **Raspberry Pi M.2 HAT+ Compact** (fits inside the official case; 2230-size SSDs only) | ~$15–22 |
| **2230 NVMe SSD**, e.g. Raspberry Pi SSD 256 GB | ~$60–76 (it was $30 at launch; flash memory prices are up) |

An SSD copes with power cuts and wear better than any SD card. The bot
writes very little, though, so a good SD card is fine with a UPS and
backups. Skip the SSD unless you want the extra margin.

### Cheaper, if you must

A **Raspberry Pi 4 with 1 GB** costs $35, and like the Pi 5 1 GB its price
hasn't risen. With its 15 W power supply, a case and a card, the build comes
to about $65–80. It also has Ethernet and Wi-Fi and runs the bot fine. It has
no clock battery, so it gets the time from the network, and no SSD slot.

### What we ruled out

- **Pis with more RAM:** the 2 GB costs about $77.50, the 4 GB about $85 and
  the 8 GB about $130 (October 2026). One bot doesn't use the extra RAM.
- **Raspberry Pi Zero 2 W:** it has 512 MB of RAM against about 260 MB per
  bot process, and no Ethernet port.
- **Orange Pi Zero 3:** it costs about $40 for 1 GB, which is no cheaper than
  a Pi 5. It also gets fewer long-term security updates and has no official
  secure-boot path.
- **Used x86 mini PCs** (e.g. a Lenovo ThinkCentre M720q, which has a TPM 2.0
  chip): about $90–150 used and more refurbished, and they draw several times
  the Pi's idle power. For this job, a Pi 5 with Raspberry Pi's secure boot
  gives the same kind of protection.
- **Microcontrollers (ESP32 and similar):** they can't run Node or the kit
  (see above).

## Assembly (about 15 minutes)

1. Plug the RTC battery into the Pi 5's `BAT` connector and stick it down.
2. Seat the Pi in the case base and plug the case fan into the `FAN`
   connector. If you bought the SSD, fit the M.2 HAT+ Compact and the SSD,
   and connect the PCIe ribbon cable, before you close the case.
3. Use Raspberry Pi Imager to write **Raspberry Pi OS Lite (64-bit)** to the
   card. In its OS customisation settings, set:
   - the hostname;
   - your user;
   - your Wi-Fi network and country;
   - SSH with **public-key authentication only**.
4. Insert the card, plug in Ethernet, then plug in the power.

## Setup for safety

The parts are ordinary. What makes the box safe is how it is set up. These
steps are in order of importance.

1. **Keep little on the bot's addresses.** Whoever can read the key files
   can spend what the addresses hold, so fund the bot only with what it
   should trade. Sweep profits to a wallet the bot has no key for.
2. **Give it one job.** Install nothing else on this Pi: no desktop, no other
   services, no web browsing. Keep it updated with
   `sudo apt update && sudo apt full-upgrade`, and install
   `unattended-upgrades` for security fixes.
3. **Block incoming connections.** The bot only connects out. Allow SSH from
   your own network only, and turn Bluetooth off:

   ```sh
   sudo apt install -y ufw
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp   # your LAN's range
   sudo ufw enable
   sudo systemctl disable --now bluetooth
   ```

4. **Use Ethernet first and Wi-Fi as backup.** Keep both connected.
   NetworkManager, Raspberry Pi OS's default, prefers the wired connection
   and switches to Wi-Fi if the cable or switch fails.
5. **Keep the clock right.**
   - The official RTC battery is rechargeable, so turn on charging: add
     `dtparam=rtc_bbat_vchg=3000000` to `/boot/firmware/config.txt`.
   - Make services wait for network time before they start:
     `sudo systemctl enable systemd-time-wait-sync`.
6. **Encrypt the keys while they are stored.** Always do the first step, then
   choose one of the other two:
   - **The kit's store key (always).** Before the bot's first start, create
     a key file and point `KIT_STORE_KEY_FILE` at it (see "Encrypting the
     store" in the kit README). It encrypts the control tokens and swap
     secrets, but not the key files themselves.
   - **An encrypted volume that you unlock after each boot.** Keep the kit in
     a LUKS container. After a reboot, you SSH in, unlock the container and
     start the bot. This is simple and strong. The cost is a manual step
     after each power-up, which is rare with a UPS.
     ([Commands below.](#encrypted-volume))
   - **Secure boot with automatic unlock (advanced).** Raspberry Pi's
     [secure boot provisioner](https://www.raspberrypi.com/documentation/security/secure-boot-provisioner.html)
     supports the Pi 5. Afterwards the board boots only an OS you signed, and
     its disk is LUKS-encrypted with a key held in the chip's one-time
     programmable memory. It unlocks itself after a power cut but can't be
     read in any other board. **This change is permanent** because it writes
     your key's hash into the chip. The provisioning runs from a second
     64-bit Pi.
7. **Make backups.** Back up `keys/`, `data/`, `market.mainnet.json` and the
   store key, encrypted, to a USB stick you keep offline.

### Install the kit

The commands below make a `p2pbot` user and put the kit in
`/srv/p2pbot/p2pbotkit`, which is the layout the service file expects. If
you are using the encrypted volume, create and mount it at `/srv/p2pbot`
first ([next section](#encrypted-volume)).

```sh
sudo apt install -y git curl
sudo useradd --create-home --shell /bin/bash p2pbot
sudo mkdir -p /srv/p2pbot
sudo chown p2pbot:p2pbot /srv/p2pbot
sudo chmod 700 /srv/p2pbot
sudo -u p2pbot -H sh -c 'cd /srv/p2pbot && git clone https://github.com/Infinite-Star-Studios/p2pbotkit.git'
sudo -u p2pbot -H sh -c 'cd /srv/p2pbot/p2pbotkit && umask 077 && mkdir -p keys && openssl rand -hex 32 > keys/store.key'
sudo -u p2pbot -H sh -c 'cd /srv/p2pbot/p2pbotkit && KIT_STORE_KEY_FILE=$PWD/keys/store.key sh start.sh mainnet'
```

The last command is the kit's normal first start. It does four things:

1. It downloads Node 22 if the system's Node is older.
2. It makes the bot's keys and `market.mainnet.json`.
3. It prints the bot's two addresses.
4. It shows a dry run.

The new config lists no offers, so the first start stops after the dry run.
Then:

1. Add your offers to `market.mainnet.json`, at your own prices.
2. Fund the two addresses. `sh start.sh mainnet setup` shows them again.
3. Check what the bot would do. This sends nothing:

   ```sh
   sudo -u p2pbot -H sh -c 'cd /srv/p2pbot/p2pbotkit && KIT_STORE_KEY_FILE=$PWD/keys/store.key sh start.sh mainnet dry-run'
   ```

4. Install the service (below). It starts trading without asking.

### Encrypted volume

These commands create a 4 GB encrypted file at `/srv/p2pbot.img` and mount
it at `/srv/p2pbot`:

```sh
sudo apt install -y cryptsetup
sudo fallocate -l 4G /srv/p2pbot.img
sudo cryptsetup luksFormat /srv/p2pbot.img
sudo cryptsetup open /srv/p2pbot.img p2pbot
sudo mkfs.ext4 /dev/mapper/p2pbot
sudo mkdir -p /srv/p2pbot
sudo mount /dev/mapper/p2pbot /srv/p2pbot
```

After every reboot, unlock it and start the bot:

```sh
sudo cryptsetup open /srv/p2pbot.img p2pbot && sudo mount /dev/mapper/p2pbot /srv/p2pbot && sudo systemctl start p2pbot-market
```

While the volume is locked, the service skips its start instead of failing.

### Run it as a service

After the first start by hand, install
[`systemd/p2pbot-market.service`](systemd/p2pbot-market.service). With it,
the bot starts at boot and restarts after a crash.

```sh
sudo cp systemd/p2pbot-market.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now p2pbot-market
journalctl -u p2pbot-market -f
```

How the service behaves:

- **Starting:** it runs `start.sh mainnet` with no terminal and with
  `KIT_ALLOW_MAINNET=1`. The kit accepts that in place of the typed yes, and
  then skips the dry run.
- **Stopping:** `systemctl stop` works like pressing Ctrl-C once. The bot
  cancels its offers and finishes its running trades.
- **Timeout:** after `TimeoutStopSec` (15 minutes in the file), systemd kills
  the bot, which works like pressing Ctrl-C twice. The kit's state survives
  this and the next start resumes every trade. A stopped bot sends no locks,
  claims or refunds, though, so don't leave it stopped.

The file passes `systemd-analyze verify`, but it has not been run on a Pi
yet.

## Sources

Prices (November 2026 unless noted):

- Raspberry Pi 5 and 4 prices after the 2026 rises:
  [Tom's Hardware, third price rise of 2026](https://www.tomshardware.com/raspberry-pi/component-shortages-drive-raspberry-pi-prices-up-by-up-to-23-percent-escalating-lpddr4-lpddr5-costs-trigger-the-third-price-hike-of-the-year),
  [Raspberry Pi, "More memory-driven price rises"](https://raspberrypi.com/news/more-memory-driven-price-rises),
  [Phoronix, 16 GB at $385](https://phoronix.com/news/Raspberry-Pi-5-16GB-385)
- Pi 5 1 GB listings: [CanaKit](https://www.canakit.com/raspberry-pi-5-1gb.html), [Adafruit](https://www.adafruit.com/product/6447)
- Case and power supply: [PiShop.us Pi 5 accessories](https://www.pishop.us/product-category/raspberry-pi/raspberry-pi-5/raspberry-pi-5-accessories/), [ThePricer](https://www.thepricer.org/how-much-does-raspberry-pi-cost/)
- M.2 HAT+ Compact: [The Pi Hut](https://thepihut.com/products/raspberry-pi-m-2-hat-compact), [LinuxGizmos](https://linuxgizmos.com/raspberry-pi-m-2-hat-compact-released-with-2230-nvme-support/)
- Raspberry Pi SSD: [Adafruit](https://www.adafruit.com/product/6090), [product brief](https://pip-assets.raspberrypi.com/categories/892-raspberry-pi-5/documents/RP-008357-DS-2-raspberry-pi-ssd-product-brief.pdf)
- SD cards: [PriceSpy](https://pricespy.co.uk/product.php?p=4792302)
- UPS: [APC BE600M1 (Markertek)](https://www.markertek.com/product/spc-be600m1/apc-be600m1-back-ups-es-600va-120v-1-usb-charging-port), [Geekworm X1200 (Central Computers)](https://centralcomputer.com/geekworm-x1200-2-cell-18650-5-1v-5a-ups-hat-for-raspberry-pi-5.html), [Geekworm Pi 5 UPS range](https://geekworm.com/collections/raspberry-pi-5-ups)
- Alternatives: [Orange Pi Zero 3 (SpecPicks)](https://specpicks.com/product/B0CB112MN2), [used ThinkCentre as a home server (dev.to)](https://dev.to/selfhostingsh/used-lenovo-thinkcentre-as-a-home-server-1eh1), [used tiny PCs (Computing For Geeks)](https://computingforgeeks.com/best-used-tiny-pc-homelab/), [Pi and mini PC prices at parity (Tom's Hardware)](https://www.tomshardware.com/raspberry-pi/raspberry-pi-and-mini-pc-home-lab-prices-hit-parity-as-dram-costs-skyrocket-price-hikes-force-hobbyists-to-weigh-up-performance-versus-power-consumption)

Security:

- [Raspberry Pi secure boot provisioner](https://www.raspberrypi.com/documentation/security/secure-boot-provisioner.html)
- [Raspberry Pi security architecture](https://www.raspberrypi.com/documentation/security/security-architecture.html)
