# uHandPy (DiroSign)

## About the uHandPi

The uHandPi is an intelligent robotic hand by Hiwonder, powered by a **Raspberry Pi 4B**. 
It has **6 degrees of freedom** total : 
- 5 in the fingers via micro anti-blocking servos, 
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
| **Storage** | microSD ~8 GB total · 5.8 GB used · 1.1 GB free ⚠️ | (will probably need to be replaced)
| **SD Read Speed** | ~45.5 MB/s |
| **OS** | Raspbian Buster, kernel 5.10.103 (2022) | (see if we can upgrade)
| **Camera** | USB · 640×480 @ 30 fps · YUYV format |
| **WiFi** | Cypress CYW43455 · dual-band 802.11ac (2.4 + 5 GHz) |
| **Bluetooth** | 5.0 |
| **Ethernet** | Gigabit (1 Gbps) |
| **WiFi default mode** | AP hotspot (`192.168.149.1`) — no internet in this mode |

## Pre-installed Software

| Category | Libraries / Tools |
|----------|-------------------|
| **Vision** | OpenCV, dlib, Pillow |
| **ML / AI** | Keras, TensorFlow, YOLOv3 |
| **Servo control** | pigpio, pyserial, RPi.GPIO |
| **Streaming** | mjpg-streamer |
| **Other** | numpy, Flask, sqlite3, requests |

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

---

## Setup Checklist
 
- [ ] Clone current SD card to 128GB card
- [ ] Expand swap to 4GB
- [ ] Configure WiFi (client mode for SSH access)

---

## Resources

- 🤖 [uHandPi docs](https://wiki.hiwonder.com/projects/uHandPi/en/latest/docs/1.getting_ready.html#)
- 🤝 [ASL letters demonstration](https://alphabet.lingvano.com/glossary/)
- 📦 [Kaggle ASL Dataset](https://www.kaggle.com/datasets/ayuraj/asl-dataset/data?select=asl_dataset)