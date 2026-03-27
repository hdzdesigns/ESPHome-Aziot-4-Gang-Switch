# Aziot 4-Gang Smart Switch (ESPHome / CB3S)

This repository contains a fully local, cloud-free ESPHome configuration for the **Aziot 4-Gang Smart Switch**. By flashing this firmware, you remove all Tuya cloud dependencies and integrate the switch directly into Home Assistant via the ESPHome dashboard.

---

## Hardware Specifications
* **Microcontroller:** Beken CB3S (BK7231N)
* **Relays:** 4x High-voltage relays
* **Inputs:** 4x Tactile physical buttons
* **Indicator:** 1x Status LED (Blue)

### Pin Mapping Table
| Component | Pin | Function | Notes |
| :--- | :--- | :--- | :--- |
| **Relay 1** | `P14` | Output | Default OFF |
| **Relay 2** | `P6` | Output | Default OFF |
| **Relay 3** | `P9` | Output | Default OFF |
| **Relay 4** | `P8` | Output | Default OFF |
| **Button 1** | `P24` | Input | Pullup, Inverted |
| **Button 2** | `P10` | Input | **Shared with UART RX** |
| **Button 3** | `P7` | Input | Pullup, Inverted |
| **Button 4** | `P26` | Input | Pullup, Inverted |
| **Status LED** | `P11` | Output | Inverted, Blue |

---

## Requirements

### Software
1.  **Home Assistant** with the ESPHome Add-on installed.
2.  **ltchiptool:** A GUI/CLI tool for flashing LibreTiny/Beken chips.
    * Website: [https://docs.libretiny.eu/docs/flashing/tools/ltchiptool/](https://docs.libretiny.eu/docs/flashing/tools/ltchiptool/)
3.  **LibreTiny:** The underlying framework used by ESPHome to support Beken chips.

### Hardware
1.  **USB-to-TTL Adapter:** CP2102 or CH340 based.
    * **Warning:** Must be set to **3.3V logic**. 5V will destroy the CB3S module.
2.  **Soldering Equipment:** Fine-tip iron and thin jumper wires.
3.  **Opening Tools:** Plastic pry tool and small Philips screwdriver.

---

## Installation Guide

### 1. Disassembly
* Disconnect the switch from all AC mains power. **Do not attempt to flash while connected to 110V/220V.**
* Remove the front faceplate using a pry tool.
* Unscrew the logic board from the power supply base.

### 2. Wiring the Serial Connection
Solder four wires from your USB-to-TTL adapter to the pads on the side of the CB3S module:
* **VCC** -> `3V3`
* **GND** -> `GND`
* **TX** on adapter -> `RX1` (`P10`) on CB3S
* **RX** on adapter -> `TX1` (`P11`) on CB3S

### 3. ESPHome Configuration
* Create a new device in your ESPHome dashboard.
* Paste the provided `fourgangswitch.yaml` into the editor.
* Update `secrets.yaml` with your Wi-Fi SSID, Password, and API encryption keys.
* Click **Install** > **Manual Download** > **Modern format (.uf2)**.
* Save the compiled file to your computer.

### 4. Flashing Procedure
1.  Open `ltchiptool` and select your COM port and the downloaded `.uf2` file.
2.  Click **Start Flashing**.
3.  **Triggering Download Mode:** The Beken chip requires a reboot to enter flash mode. While the software says "Connecting", briefly ground the **`CEN`** pin or quickly power-cycle the `3V3` wire.
4.  Once the progress bar reaches 100%, desolder the wires and reassemble the switch.

---

## Technical Workarounds

### UART Conflict Mitigation
By default, the Beken chip uses `P10` and `P11` for logging. Since these are used for a button and the LED, the logger must be redirected to prevent crashes:
```yaml
logger:
  baud_rate: 0
  hardware_uart: UART2
```

### Disabling Auto-Download-Reboot (ADR)
To prevent the system from listening for flash commands on the button pin (`P10`), the following build flag is required to strip the ADR service from the firmware:
```yaml
esphome:
  platformio_options:
    build_flags:
      - -DLT_AUTO_DOWNLOAD_REBOOT=0
```

---

## Known Quirks

### Sticky Button 2 (P10)
Button 2 is wired to the hardware `RX` pin. Due to proprietary, closed-source Wi-Fi radio drivers in the Beken SDK, the radio occasionally hijacks this pin to listen for RF calibration commands.
* **Symptom:** Button 2 may be unresponsive for 5–30 seconds after the switch has been idle.
* **Workaround:** Pressing the button multiple times usually "wakes" the logic.
* **Permanent Fix:** If you are comfortable with trace-cutting, move the physical wire from `P10` to the unused **`P23`** pad on the CB3S and update the YAML accordingly.

---

## Credits
* **LibreTiny Project:** For making Beken chips usable with ESPHome.
* **ltchiptool:** For the cross-platform flashing utility.
