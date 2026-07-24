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
| **Swap** | zram (zstd-compressed), 2 GB — configured by default on the current OS image |
| **OS** | Raspberry Pi OS, Debian **Trixie** (upgraded from Raspbian Buster) |
| **Camera** | USB · 640×480 @ 30 fps · YUYV format |
| **WiFi** | Cypress CYW43455 · dual-band 802.11ac (2.4 + 5 GHz) |
| **Bluetooth** | 5.0 |
| **Ethernet** | Gigabit (1 Gbps) |
| **WiFi mode** | Client mode (connects to home network) — hotspot mode available via physical button reset |

### ⚠️ Power Supply — Known Issue

The stock Hiwonder power adapter is **12V/1A**, which is **insufficient** to run the Pi 4B and 6 servos together. Testing confirmed repeated undervoltage events (`vcgencmd get_throttled` → `0x50000`, kernel log: `Undervoltage detected!`) even when moving a single servo.

**Fix:** replace with a **12V/5A** adapter (same barrel connector). Reference part: [Hiwonder 12V/5A Power Adapter](https://www.hiwonder.com/products/power-supply-adapter-12v-5a).

Until the new adapter is installed, avoid running servo movement tests — repeated brownouts risk corrupting the SD card or damaging hardware over time.

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

The original Hiwonder OS image (Raspbian Buster, 2022) was replaced with a fresh Raspberry Pi OS (Trixie) install to get a supported, up-to-date system. This required rewriting the servo control code, since the original stack (`pigpio` Python bindings + raw GPIO) is no longer directly compatible out of the box.

**Files rewritten (English comments, `gpiozero`-based):**

| Original file | New file | Notes |
|----------------|----------|-------|
| `LeServo.py` | `LeServo_gpiozero.py` | Same public API (position in microseconds). Uses `gpiozero.Servo` with the `PiGPIOFactory` backend (hardware PWM via compiled `pigpiod`) instead of raw `pigpio` calls. |
| `LeArm.py` | `LeArm_gpiozero.py` | Same public API (`setServo`, `setServo_CMP`, `initLeArm`, etc.). Servo pin mapping and safety limits (900–2200µs fingers, 500–2500µs pan-tilt) preserved from the original. |
| `hw_button_scan.py` | `hiwonder-toolbox-gpiozero/hw_button_scan.py` | Rewritten using `gpiozero.Button` with built-in `hold_time` (3s) instead of a manual polling loop. Button 1 (pin 25) resets WiFi to hotspot mode; Button 2 (pin 22) shuts down the Pi. Both confirmed working on the physical buttons. |

The original Hiwonder files are kept untouched alongside the new ones for reference.

**Validated so far:**
- ✅ Single servo movement (pan-tilt, servo 6) — confirmed working, smooth interpolation intact
- ✅ Hardware PWM via compiled `pigpiod` (no more `PWMSoftwareFallback` warning)
- ✅ Physical buttons (WiFi reset / shutdown) — pin mapping confirmed, hold-to-trigger working
- ⏸️ Full 6-servo simultaneous movement — blocked until the 12V/5A adapter is installed

---

## Project Goal

We aim to build a pipeline where the robot hand can understand and respond to American Sign Language (ASL):

1. **ASL Recognition** — A vision model reads ASL signs from the camera.
2. **Language Understanding** — An LLM takes the recognized signs as input and generates a response.
3. **Hand Control** — A third model translates that response into robot hand movements.

### Design Considerations

- **Reading frequency** — e.g. sample a sign every 10 seconds
- **Word spacing** — e.g. 20 seconds without a sign counts as a space
- **End of prompt** — e.g. 30 seconds of inactivity or hands dropping signals end of input

An idea to explore is to make the model learn autonomously, on itself.

> Training will happen on a separate PC (Windows + WSL); only optimized inference (TFLite / ONNX, int8 quantized) will run on the Pi.

---

## Setup Checklist

- [x] Clone original 8GB SD card to 128GB card
- [x] Configure WiFi (client mode for SSH access)
- [x] Migrate OS to Raspberry Pi OS (Trixie)
- [x] Compile `pigpio` from source + set up systemd service
- [x] Rewrite servo control code (`gpiozero` + hardware PWM backend)
- [x] Rewrite button scan script (`gpiozero`)
- [ ] Replace power adapter with 12V/5A
- [ ] Validate full 6-servo simultaneous movement
- [ ] Build high-level `Hand` API for the ASL pipeline (e.g. `make_letter()`, `close_fist()`)
- [ ] Define ASL reading conventions (frequency, spacing, end-of-prompt)
- [ ] Build ASL recognition model (vision, trained off-device)

---

## Resources

- 🤖 [uHandPi docs](https://wiki.hiwonder.com/projects/uHandPi/en/latest/docs/1.getting_ready.html#)
- 🤝 [ASL letters demonstration](https://alphabet.lingvano.com/glossary/)
- 📦 [Kaggle ASL Dataset](https://www.kaggle.com/datasets/ayuraj/asl-dataset/data?select=asl_dataset)
- 🔌 [Replacement 12V/5A Power Adapter](https://www.hiwonder.com/products/power-supply-adapter-12v-5a)
