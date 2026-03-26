# Mastering Xiaozhi AI Chatbot with Xmini-C3/V3 Development Board

*Modified August 19, 2025*

---

## Preface

Thanks to the Xiaozhi AI open-source project by "Xia Ge" (Shrimp Bro), which now supports the Xmini-C3 and JLCPCB Zhanshi C3. This document mainly covers firmware flashing and usage for the Xmini-C3. More C3 resources are pending.

---

## Xmini-C3 Introduction

The Xmini-C3 development board was designed and open-sourced by Xia Ge, the lead creator of Xiaozhi AI. It uses the Espressif ESP32-C3 as the main control chip, with onboard 16M NOR FLASH, ES8311 audio codec and NS4150 power amplifier IC for audio, and a ceramic antenna. It can smoothly run the latest version of Xiaozhi AI.

**Features:**

1. Supports BOOT button wake-up (does not support offline voice wake-up)
2. Supports new version automatic OTA upgrade (16M FLASH)
3. Supports switching between multi-turn and single-turn conversation modes
4. Supports 0.96-inch SSD1306 OLED display (monochrome/dual-color), supports emoji and subtitle display
5. Supports 3.7V lithium battery power supply and charging (battery level detection not yet supported)
6. Supports 3W–5W speaker interface
7. Extension interface is 6-Pin, with two power lines and two IO interfaces, supports external IO devices (in development)

---

## Xmini-C3 Configuration Reference

1. MCU: ESP32-C3 QFN-32
2. Flash: NOR FLASH 16MB
3. Audio + Amplifier: ES8311 + NS4150B
4. Power IC: LGS4056 (single-cell lithium) / V3 version changed to TP4057
5. Extension interface: U0RXD, U0TXD, VSys+GND (VSys voltage subject to actual test), 3V3+GND
6. Screen: Default 128x64 resolution SSD1306-driven 0.96-inch OLED
7. Battery: Default 602030 300mAh lithium battery (battery level detection not yet supported), standard 1.25mm connector
8. Speaker: Default 3514 ultra-thin cavity speaker (8Ω 2W), replaceable with ~3W cavity speaker, standard 1.25mm connector

---

## I. Xmini-C3 WiFi Network Configuration

### 1. Device Restart

Confirm that the Xmini-C3 has the correct firmware flashed. Power on the Xmini-C3 by toggling the power switch upward (if no battery, connect USB first). Wait a few seconds for the board to enter network configuration mode — the RGB LED on top will blink blue at a steady rate.

> **Note:** If flashing via VSCode IDE or IDF command line, the board will restart automatically — no manual restart required.

### 2. Xmini-C3 Network Configuration

**1) Connect phone or PC to Xmini-C3 hotspot**

Open your phone's WiFi settings and find the hotspot starting with **Xiaozhi-** — that is the board's hotspot. After connecting, wait a few seconds and the WiFi selection page will appear automatically. Do not close it manually beforehand.

**2) Select WiFi and enter password**

From the 2.4G networks scanned by the Xiaozhi hotspot, select your home or office WiFi network (no need to manually type the SSID — it will be filled in automatically after selecting). Enter the WiFi password and tap **Connect** (if the password is already saved on your phone, it will submit automatically).

> **Note:** If the WiFi configuration page does not open automatically (possible compatibility issue), ensure your phone stays connected to the **Xiaozhi-** hotspot, then open a browser and go to **192.168.4.1** to access the WiFi configuration page directly. The steps are the same as above.

**3) Network connected, device restarts**

If the WiFi password is correct, the Xmini-C3 will connect to the network and restart within 3 seconds. After restarting, the device will automatically connect to the configured WiFi network.

### 3. Re-configuring Network on Xmini-C3

**1) Auto-reconnect failure re-configuration**

Starting from firmware version V0.9.9, Xiaozhi AI supports saving multiple WiFi networks. When switching between saved WiFi environments, the device will switch automatically.

If the device fails to connect to the last-used WiFi (after 5 retries), it will scan for other saved networks. If no saved network is found, the device enters re-configuration mode and plays a voice prompt: *"Please configure the network"* after about 10 seconds.

**2) Manual re-configuration / new WiFi setup**

After the Xiaozhi AI device restarts and before it successfully connects (blue LED blinking), press the BOOT button to enter re-configuration mode.

In the re-configuration page, previously configured WiFi networks are shown. If a network is no longer needed, click the **X** next to its name to remove it.

For detailed configuration steps, refer to Section 2 above.

---

## II. Xmini-C3 Device Activation

### 1. Configuration Panel

Xiaozhi AI backend configuration panel: [https://xiaozhi.me/](https://xiaozhi.me/)

On first boot, the Xmini-C3 enters standby mode. Press the BOOT button to wake it and start a conversation — say anything (e.g., "please configure network") as conversation content (C3 does not support offline voice recognition). If the device is not yet registered, Xiaozhi AI will speak a 6-digit verification code and prompt you to log in to the control panel to add the device.

Visit [https://xiaozhi.me/](https://xiaozhi.me/) from your phone or computer. Register an account if you haven't already.

### 2. Create an AI Agent

After logging in to xiaozhi.me, you will see the welcome screen. Click **Console** to enter.

Click **+ New Agent** to create an AI agent and fill in the agent name.

### 3. Add Device

Following the voice prompt from the device, go to: **Console → Agent → Add Device** (first time). Click **[Add Device]**, enter the 6-digit verification code spoken by the Xiaozhi AI device, then click **OK** to register. The device count will update — click to enter and you will see the newly registered Xmini-C3 device.

You can now use the newly added Xmini-C3 device. Press the button in the upper right to start a conversation.

### 4. Additional Device Settings

**1) Device Notes**

- **OTA Upgrade:** The Xmini-C3 supports automatic OTA upgrades when using the official Xmini-C3 firmware. Make sure the OTA upgrade option is enabled in the device list.
- **Device Note:** To distinguish between multiple devices, you can add a note to each device. Click **[Note]**, fill in the name in the popup, and click **Confirm** to save.

**2) Configure Agent Role**

On the agent configuration page, you can set the AI role's nickname and role description (prompt), or select from system default role templates such as "Taiwan Girlfriend", "English Teacher", etc. Available settings:

- **Assistant Nickname:** The AI's name — the AI will know what it's called (unrelated to wake word).
- **Voice:** Default is "Wanwan Xiaohe" (Chinese). You can choose from 20+ voices and preview audio samples.
- **Role Description:** A text description of the AI's character, personality, and special notes.
- **Memory:** Currently short-term memory only (1000 characters), auto-summarized by AI. Content is overwritten when the limit is reached. Long-term memory upgrade pending.
- **Voice Model:** Default is Qwen (QWen) real-time (recommended). You can also switch to Deepseek V3, DouBao 1.5 Pro, or QWen Max 2.5.

---

## III. Xmini-C3 Firmware Flashing

### 1. Hardware Connection

The Xmini-C3 ships as a blank board with no firmware pre-flashed (may have a test program depending on the specific unit).

For first-time flashing: while connecting the Xmini-C3 via USB to your PC, **hold the BOOT button** while turning on the power switch. This allows the PC to recognize the board's USB interface. Wait a moment — a "Device is ready" notification will appear in the system tray (this guide uses Windows 10 as reference).

### 2. Firmware Preparation

Download the latest Xmini-C3 firmware or source code from the Xiaozhi AI open-source project page (GitHub may require a VPN in some regions).

> **Note:** For the Xmini-C3, download `Xmini-C3.zip`; for the Xmini-C3-V3, download `Xmini-C3-V3.zip`. These are different firmwares due to changes including battery detection pins.

**1) Firmware and Source Download Links**

- Xiaozhi AI firmware releases: [https://github.com/78/xiaozhi-esp32/releases](https://github.com/78/xiaozhi-esp32/releases)
- Latest firmware sync: [v2.2.4 Xiaozhi AI Latest Firmware & Source](https://ccnphfhqs21z.feishu.cn/wiki/W14Kw1s1uieoKjkP8N0c1VVvn8d)
- Source main repo: [https://github.com/78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32)
- If GitHub is inaccessible: [https://xiaozhi.me/download](https://xiaozhi.me/download)

**2) Firmware Version Selection**

- **Xmini-C3 board:** Download the latest `vx.x.x_xmini-c3.zip` file (vx.x.x is the version number), which contains the bin file for flashing.
- **Xmini-C3-V3 board:** Download the latest `vx.x.x_xmini-c3-v3.zip` file, which contains the V3-specific bin file.

> **Note:** Version numbers may differ after updates — the images in this guide are for reference only. Always use the latest version.

**3) Extract Firmware Locally**

Extract the downloaded zip file to a local folder on your PC. It is recommended to avoid Chinese characters in the folder path, as this may cause errors when loading files in the flash tool.

### 3. Flash Firmware

**1) Flash Tool / Method**

Download the ESP32 Flash Download Tool (no installation required) for PC flashing, or use the web browser online flash method (pending testing).

- Flash Download Tool: [https://www.espressif.com.cn/zh-hans/support/download/other-tools](https://www.espressif.com.cn/zh-hans/support/download/other-tools)
- Online flash (pending): [https://espressif.github.io/esp-launchpad/](https://espressif.github.io/esp-launchpad/)

**2) Using FLASH DOWNLOAD TOOL**

**Step 1: Select chip type ESP32-C3**

In the ChipType dropdown, select **ESP32-C3**.

**Step 2: Flash the bin firmware**

1. Click the **...** button on the file selection row to browse and load the extracted bin file (avoid Chinese characters in the path).
2. Make sure the checkbox next to the firmware file is checked — unchecked files will not be flashed.
3. Set the flash start address to **0** or **0x0**. If the bin file row turns green, the file loaded correctly.

**Step 3: Select COM port**

Select the correct COM port for your board (e.g., COM17). If unsure which port is the board, unplug and replug it — the COM port that disappears and reappears is the board's port.

**Step 4: Start flashing**

Click **START** — a progress bar will appear at the bottom indicating flashing is in progress.

If an error occurs, check the previous steps, fix the issue, and retry.

**Step 5: Flashing complete**

The interface will show **FINISH** when done.

After flashing, close the Flash Download Tool, restart the device, and reconfigure the network as needed.

---

## IV. How to Compile the C3 Project (For Developers)

### 1. Prerequisites

- Download the Xiaozhi AI terminal project source code and extract it to a local folder.
- Set up the ESP-IDF 5.3 development environment. Reference: [Windows ESP-IDF 5.5.3 Setup and Xiaozhi Compilation Guide](https://icnynnzcwou8.feishu.cn/wiki/JEYDwTTALi5s2zkGlFGcDiRknXf)
- Alternatively, refer to the official Espressif documentation or search online for IDF setup tutorials.

### 2. Modifying Xiaozhi AI Source Configuration

**Method: VSCode or Cursor IDE**

Ensure your IDF development environment is IDF v5.4+ (v5.3.1 and above also works; this guide uses v5.3.1 as reference).

**1) SDK Configuration for Xmini-C3**

1. Click the **SDK Configuration Editor** (gear icon) in the IDE status bar and wait for it to load.
2. Find the **Xiaozhi Assistant** navigation menu and click to enter the project configuration.
3. In the **Xiaozhi Assistant** configuration page, under **Board Type**, select **虾哥 Mini C3** (Xia Ge Mini C3). Click **Save** — the change takes effect after recompilation.
4. Navigate to **ESP System Settings** and confirm that the CPU frequency for ESP32-C3 is set to **160M**. Click **Save** if you change it.

> **Note:** Since the Xmini-C3 has 16M flash (same as ESP32-S3 WROOM-1 N16R8), no flash size change is needed here. If using another ESP32-C3 board, adjust the flash size and partition table accordingly.

**2) Compilation Settings for Xmini-C3**

To compile for Xmini-C3, update the following in the project:

1. **Set target chip to ESP32-C3:** Change the target to `esp32c3`, then select **ESP32-C3 chip (via builtin USB-JTAG)**.

2. **Flash method: select UART** (other methods have not been fully tested — discuss in the group if issues arise).

3. **Set COM port:** Select the COM port corresponding to your Xmini-C3 board (e.g., COM17).

4. **Compile:** If needed, modify `config.h` or other source files under `main/boards/xmini-c3/`. Click the **Build** icon (wrench) in the IDE status bar to start compilation.

5. **Compilation successful:** After a successful build, the bin file is generated. Click the **Flash** icon (lightning bolt or flame icon) to flash to the Xmini-C3 board.

6. **Flashing complete:** Click the flash icon in the VSCode status bar and wait for the process to complete. A **Flash Done** message confirms successful flashing.
