
# Danfoss Ally Integration for Home Assistant

This repository contains a blueprint and a script to integrate Danfoss Ally devices directly into Home Assistant, bypassing the need for the official Danfoss Ally gateway. The provided files enable easy configuration and control of Danfoss Ally devices within the Home Assistant environment.

## Files in This Repository

- **`blueprint/danfoss-ally.yaml`**: The Home Assistant blueprint for configuring and automating Danfoss Ally devices.
- **`script/danfoss-ally-room-assistant.yaml`**: A Home Assistant script for direct control of Danfoss Ally devices.

## Installation

### Step 1: Clone the Repository
Clone or download this repository to your Home Assistant installation:

```bash
git clone https://github.com/eskholm/danfoss-ally.git
```

### Step 2: Add the Blueprint
1. Open Home Assistant.
2. Navigate to `Settings` > `Blueprints`.
3. Click on the "Import Blueprint" button.
4. Use the file `blueprint.yaml` from this repository to add the Danfoss Ally blueprint.

### Step 3: Add the Script
1. Copy the file `danfoss_ally_script.yaml` to your Home Assistant `scripts/` folder.
2. Go to `Settings` > `Scripts` in Home Assistant and verify that the script is available.

### Step 4: Configure Your Devices
1. Pair your Danfoss Ally devices with your Zigbee network.
2. Use the blueprint and script to control and automate your Danfoss Ally devices.

## Usage

Once the files are set up, you can automate and control your Danfoss Ally devices within Home Assistant. Use the blueprint to create automations and the script for manual control of the devices.

## License
This project is licensed under the GPLv3 License - see the [LICENSE](LICENSE) file for details.
