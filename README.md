# uHandPy (DiroSign)

## About the uHandPi

The uHandPi is an intelligent robotic hand by Hiwonder, powered by a **Raspberry Pi 4B**.
It has **6 degrees of freedom** total:
- 5 in the fingers via micro anti-blocking servos
- a pan-tilt base that rotates up to 180° horizontally

An onboard HD camera enables computer vision tasks through OpenCV, making it suited for AI image recognition and gesture-based applications.

📄 [uHandPi documentation](https://wiki.hiwonder.com/projects/uHandPi/en/latest/docs/1.getting_ready.html#)

---

## Hardware Specifications

> Verified directly from the device.

| Spec | Value |
|------|-------|
| **Board** | Raspberry Pi 4 Model B Rev 1.5 |
| **CPU** | ARM Cortex-A72 (ARMv7), 4-core |
| **RAM** | 4 GB LPDDR4 (~3.7 GB usable) |
| **Storage** | microSD **128 GB** (upgraded from original 8 GB card) |
| **Swap** | zram (zstd-compressed), resized to **4 GB** |
| **OS** | Raspberry Pi OS, Debian **Trixie** (upgraded from Raspbian Buster) |
| **Camera** | USB · 640×480 @ 30 fps · YUYV format |
| **WiFi** | Cypress CYW43455 · dual-band 802.11ac (2.4 + 5 GHz) |
| **Bluetooth** | 5.0 |
| **Ethernet** | Gigabit (1 Gbps) |
| **WiFi mode** | Open hotspot (`uHandPi`, no password) via a NetworkManager profile, switchable from a home-WiFi client connection kept in parallel; button-triggered switch also available |

### ⚠️ Power Supply — Known Issue

The stock Hiwonder power adapter is **12V/1A**, which is **marginal/insufficient** for the Pi 4B and 6 servos together. Testing confirmed undervoltage events (`vcgencmd get_throttled`, kernel log: `Undervoltage detected!`) specifically on **fast, simultaneous multi-servo moves** (durations below ~800ms). Single-servo moves and slower simultaneous moves (≥800ms) have run reliably across repeated tests.

**Fix:** replace with a **12V/6A** adapter (same 5.5×2.1mm barrel connector).

Until the new adapter is installed, keep multi-servo move durations at **≥800ms**.

## Pre-installed / Configured Software

| Category | Libraries / Tools |
|----------|-------------------|
| **Vision** | OpenCV, dlib, Pillow |
| **ML / AI** | Keras, TensorFlow, YOLOv3 *(from original Hiwonder image — not used in the new pipeline)* |
| **Servo control** | `pigpio` (compiled from source, v79), `gpiozero`, `lgpio` |
| **Streaming** | mjpg-streamer |
| **Other** | numpy, Flask, sqlite3, requests |

> Note: `pigpiod` is no longer packaged on Debian Trixie and was compiled manually from source. A custom systemd service (`/etc/systemd/system/pigpiod.service`) was created since the old service file no longer ships with the OS.

---

## Software Migration Notes

The original Hiwonder OS image (Raspbian Buster, 2022) was replaced with a fresh Raspberry Pi OS (Trixie) install to get a supported, up-to-date system. This required rewriting the servo control code, since the original stack (`pigpio` Python bindings + raw GPIO) is no longer directly compatible out of the box. The migration was attempted twice (the first attempt was reverted after undervoltage crashes and an open-loop position-tracking bug were being debugged at the same time as the OS change); the second attempt reproduced the same working setup more quickly using the scripts below.

**Files rewritten (English comments, `gpiozero`-based):**

| Original file | New file | Notes |
|----------------|----------|-------|
| `LeServo.py` | `LeServo_gpiozero.py` | Same public API (position in microseconds). Uses `gpiozero.Servo` with the `PiGPIOFactory` backend (hardware PWM via compiled `pigpiod`) instead of raw `pigpio` calls. |
| `LeArm.py` | `LeArm_gpiozero.py` | Same public API (`setServo`, `setServo_CMP`, `initLeArm`, etc.). Servo pin mapping and safety limits (900–2200µs fingers, 500–2500µs pan-tilt) preserved from the original. |
| `hw_button_scan.py` | `hiwonder-toolbox-gpiozero/hw_button_scan.py` | Rewritten using `gpiozero.Button` with built-in `hold_time` (3s). Button 1 (pin 25) switches the WiFi connection to the `uHandPi-hotspot` NetworkManager profile (Hiwonder's own `hw_wifi.py`/`hw_wifi.service` was never carried over to this OS, so this replaces it); Button 2 (pin 22) shuts down the Pi. |
| — | `demo_test.py` | New fun test script: finger wave, fist grip, pan-tilt look-around, and a "wave hello" combo. Includes a live (non-historical) undervoltage check between steps, so it stops early instead of continuing to stress the system if power is genuinely marginal at that moment. |

The original Hiwonder files are kept untouched alongside the new ones for reference.

**Note on manual repositioning:** these are open-loop analog servos with no position feedback. Avoid manually repositioning fingers/pan-tilt while a script isn't actively holding position — the software only tracks its last commanded position, so a mismatch can cause a large, fast, high-current correction on the next command.

**Validated so far:**
- ✅ Single servo movement (pan-tilt, servo 6) — smooth interpolation, hardware PWM (no `PWMSoftwareFallback` warning)
- ✅ Full `demo_test.py` run (finger wave, fist grip, look-around, wave hello) — all 6 servos, no crash
- ✅ Physical buttons (WiFi switch / shutdown) — pin mapping confirmed, hold-to-trigger working
- ✅ Open hotspot (`uHandPi`, no password) created via NetworkManager, switchable without losing the home-WiFi fallback
- ✅ Internet sharing to the hotspot tested via USB phone tethering (`usb0`) — worked automatically through NetworkManager's `ipv4.method=shared`, confirming a USB WiFi dongle (`wlan1`) will work the same way once purchased
- ⏸️ Fast (<800ms) simultaneous multi-servo moves — still blocked until the 12V/5A adapter is installed

---

## Project Goal

We aim to build a pipeline where the robot can perform lip reading using the camera and respond with American Sign Language (ASL) using the hand:

1. **Lip reading Recognition** — A vision model reads lips from the camera.
2. **Language Understanding** — An LLM takes the recognized signs as input and generates a response.
3. **Hand Control** — A third model translates that response into robot hand movements.

### Design Considerations

- **Complicated signs encoding** — a lot of ASL signs require more sophisticated gestures

> Training will happen on a separate PC (Windows + WSL); only optimized inference (TFLite / ONNX, int8 quantized) will run on the Pi.

---

## Setup Checklist

- [x] Clone original 8GB SD card to 128GB card
- [x] Migrate OS to Raspberry Pi OS (Trixie)
- [x] Compile `pigpio` from source + set up systemd service
- [x] Rewrite servo control code (`gpiozero` + hardware PWM backend)
- [x] Rewrite button scan script (`gpiozero`)
- [x] Extend swap (zram) to 4 GB
- [x] Set up open hotspot (`uHandPi`) alongside home WiFi via NetworkManager
- [x] Validate full 6-servo simultaneous movement (≥800ms durations)
- [x] Validate possibility for USB tethering
- [ ] Pay for / receive 12V/6A power adapter
- [ ] Pay for / receive USB WiFi dongle (for hotspot internet sharing)
- [ ] Validate fast (<800ms) simultaneous multi-servo movement once the new adapter is installed
- [ ] Build high-level `Hand` API for the ASL pipeline (e.g. `make_letter()`, `close_fist()`)
- [ ] Build lip recognition model (vision, trained off-device)
- [ ] Connect the full pipeline

---

## Resources

- 🤖 [uHandPi docs](https://wiki.hiwonder.com/projects/uHandPi/en/latest/docs/1.getting_ready.html#)
- 🤝 [ASL letters demonstration](https://alphabet.lingvano.com/glossary/)
- 🤝 [Kaggle ASL Dataset](https://www.kaggle.com/datasets/ayuraj/asl-dataset/data?select=asl_dataset)
- 👄 [MIRACL-VC1 Lip Reading Dataset (Kaggle)](https://www.kaggle.com/datasets/apoorvwatsky/miraclvc1/data)
- 👄 [Lip Reading Datasets (Oxford VGG)](https://www.robots.ox.ac.uk/~vgg/data/lip_reading/)
