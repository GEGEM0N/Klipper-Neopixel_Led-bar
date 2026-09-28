# 🌌 Klipper NeoPixel LED Bar

A comprehensive Klipper configuration for multi-functional NeoPixel/WS2812 LED strip control with advanced visual feedback for 3D printers.

<pre>
                ███████╗███████╗░█████╗░████████╗██╗░░░██╗██████╗░███████╗░██████╗
                ██╔════╝██╔════╝██╔══██╗╚══██╔══╝██║░░░██║██╔══██╗██╔════╝██╔════╝
                █████╗░░█████╗░░███████║░░░██║░░░██║░░░██║██████╔╝█████╗░░╚█████╗░
                ██╔══╝░░██╔══╝░░██╔══██║░░░██║░░░██║░░░██║██╔══██╗██╔══╝░░░╚═══██╗
                ██║░░░░░███████╗██║░░██║░░░██║░░░╚██████╔╝██║░░██║███████╗██████╔╝
                ╚═╝░░░░░╚══════╝╚═╝░░╚═╝░░░╚═╝░░░░╚═════╝░╚═╝░░╚═╝╚══════╝╚═════╝░
</pre>

![Klipper](https://img.shields.io/badge/Klipper-Firmware-blue)
![License](https://img.shields.io/badge/License-MIT-green)

Turn your printer's LED strip into a smart status indicator. No more guessing if the bed is still hot, how much of the print is left, or if a print was cancelled!

---

### 🎨 Advanced LED Effects
* **Breathing Effect:** Gentle blue pulsating glow for idle/standby states.
* **Red Strobe:** Attention-grabbing flash effect (perfect for print cancellation alerts).

### 📊 Print Progress Visualization
* **Smart Progress Bar:** Dynamic LED fill that accurately reflects print completion percentage.
* **Automatic Switching:** Disables temperature monitoring during active prints to avoid interference with the progress display.
* **Visual Feedback:** Color-coded segments show progress at a glance.

### 🌡️ Smart Temperature Monitoring
Bed Temperature Indication – Color-coded LED strip showing bed temperature status when the printer is idle:
* 🔴 **>100°C** – Red (High temperature / Danger)
* 🟠 **85–100°C** – Orange-Red (Warming up / Hot)
* 🟡 **50–85°C** – Yellow-Green (Medium / Warm)
* 🟢 **<50°C** – Green (Cool / Safe to touch)
* *Note: Automatically pauses during active printing to show the progress bar.*

### 🎮 Macros & Automation
* **Start/Stop Progress:** Manual control for print progress display.
* **Temp Monitor Toggle:** Enable/disable temperature indication.
* **Effect Control:** One-command activation for breathing and strobe effects.
* **Auto-start:** Temperature monitoring begins automatically on Klipper startup.

---

## ⚙️ Prerequisites

* **Klipper Firmware** installed on your printer.
* **Addressable LED Strip** (WS2812, WS2812B, SK6812, NeoPixel, etc.) properly wired to your MCU.
* **[klipper-led_effect](https://github.com/julianschill/klipper-led_effect)** plugin by [Julian Schill](https://github.com/julianschill) installed.

---

## 🚀 Installation

### 1. Install the `led_effect` Plugin
Connect to your Raspberry Pi / Klipper host via SSH and run:

```bash
cd ~
git clone https://github.com/julianschill/klipper-led_effect.git
cd klipper-led_effect
./install-led_effect.sh
```
*(Follow the plugin's repository instructions if additional setup or KIAUH integration is required).*

### 2. Download the Configuration
Download `LED_SET.cfg` from this repository and upload it to your Klipper configuration folder (usually `~/printer_data/config/`).

### 3. Include the Config in `printer.cfg`
Open your `printer.cfg` and add the following line at the top or bottom:

```ini
[include LED_SET.cfg]
```

### 4. Update Hardware Settings
Open `LED_SET.cfg` and verify the hardware setup matches your machine:

```ini
[neopixel status_led]
pin: mcu2:PC9          # Update to your actual MCU pin
chain_count: 32        # Update to the exact number of LEDs on your strip
color_order: GRB       # Update if your LEDs show wrong colors (e.g., RGB, BRG)
```

### 5. Restart Klipper
Click **FIRMWARE RESTART** in Mainsail/Fluidd or run the following via SSH:
```bash
sudo systemctl restart klipper
```

---

## 📋 Available Macros

| Macro | Description |
| :--- | :--- |
| `START_BLUE_BREATH` | Activates the blue breathing effect. |
| `STOP_BLUE_BREATH` | Stops the breathing effect and restores temp monitor. |
| `START_RED_STROBE` | Triggers rapid red flashing (auto-stops after duration). |
| `START_PRINT_PROGRESS` | Enables the progress bar (call this in `START_PRINT`). |
| `STOP_PRINT_PROGRESS` | Disables the progress bar (call this in `END_PRINT`). |
| `END_PRINT_PROGRESS` | Alias for `STOP_PRINT_PROGRESS`. |
| `START_TEMP_MONITOR` | Enables temperature indication (Auto-runs on boot). |
| `STOP_TEMP_MONITOR` | Disables temperature indication. |
| `NEOPIXEL_DISPLAY` | Generic display control macro used internally. |

---

## 🧩 Integration with your Macros

To get the most out of this setup, integrate the macros into your existing `START_PRINT`, `END_PRINT`, and `CANCEL_PRINT` macros in `printer.cfg`.

**Example:**
```ini
[gcode_macro START_PRINT]
gcode:
    # ... your heating and homing logic ...
    START_PRINT_PROGRESS  # <--- Add this line right before printing starts

[gcode_macro END_PRINT]
gcode:
    # ... your cooldown and presentation logic ...
    END_PRINT_PROGRESS    # <--- Add this line when print finishes

[gcode_macro CANCEL_PRINT]
rename_existing: BASE_CANCEL_PRINT
gcode:
    # ... your cancel logic ...
    BASE_CANCEL_PRINT
    START_RED_STROBE DURATION=10  # <--- Flash red for 10 seconds on cancel
```

---

## 🎛️ Customization

### Adjusting Colors
Modify RGB values in the `[led_effect ...]` sections or temperature thresholds.
*Example: Change breathing color to purple:*
```ini
layers:
    breathing 3 0 top (0.3, 0.0, 0.5) # Purple breath
```

### Temperature Thresholds
Open the `UPDATE_BED_TEMP_LED` macro and change the thresholds to your liking:
```jinja2
{% if temp > 110 %} # Change threshold
    SET_LED LED=status_led RED=1.0 GREEN=0.0 BLUE=0.0
```

### Progress Bar Direction
The current configuration uses reverse indexing (LED 1 fills from the end of the strip). To reverse it back to standard (LED 1 starts filling from the beginning), modify the `NEOPIXEL_DISPLAY` macro:
```jinja2
# Change this:
{% set reversed_num = total - i %}
# To this:
{% set reversed_num = i + 1 %}
```

### Effect Speed
* **Frame rate:** `frame_rate: 50` (blue breath), `frame_rate: 24` (red strobe).
* **Strobe parameters:** `strobe 2 1.5` (frequency 2Hz, duration 1.5s per layer step).

---

## 🛠️ Troubleshooting

| Issue | Solution |
| :--- | :--- |
| **LEDs not lighting up** | Check pin configuration, wiring, and ensure the LED strip has adequate 5V power. |
| **Wrong colors (e.g., Red is Green)** | Verify `color_order` in `[neopixel]`. Try `RGB`, `BRG`, or `GRBW`. |
| **Effects not working** | Ensure the `led_effect` plugin is installed, active, and Klipper was firmware-restarted. |
| **Progress bar reversed** | Adjust `reversed_num` calculation in the `NEOPIXEL_DISPLAY` macro. |
| **Temperature monitor not updating** | Check `_TEMP_MONITOR_STATE.enable` variable. It disables itself during `printing` and `paused` states intentionally. |
| **Klipper throws "Unknown command"** | Make sure you included the `.cfg` file in `printer.cfg` and restarted Klipper. |

---

## 📝 Notes
* The progress bar uses two display templates (`led_print_percent_glow` and `led_print_percent_progress`) for flexible visualization.
* Temperature monitoring is intentionally disabled during printing to avoid conflicting with the progress display.
* The Strobe effect automatically stops after the defined duration when a print is cancelled.
* All macros are designed to be non-blocking and work seamlessly with Klipper's event system.

---

## 📜 License
**MIT** – Free to use, modify, and distribute. See `LICENSE` file for details.
