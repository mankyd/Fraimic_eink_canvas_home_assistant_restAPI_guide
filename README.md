# Fraimic E-Ink Canvas Home Assistant & Rest API Guide

[](https://github.com/custom-components/hacs) [](https://github.com/4seacz/bloomin8_eink_canvas_home_assistant/releases) [](https://www.google.com/search?q=LICENSE)

## At A Glance

Your Fraimic art frame has a built-in REST API — a set of web endpoints that let you control the frame and
check its status from other devices on your local network. You can monitor battery level, trigger a display
refresh, upload artwork, or put the frame to sleep — all from your phone, computer, or smart home system
like Home Assistant.
No internet required for local communication. No account needed. Everything runs on your WiFi network
between your devices and the frame.

### What you can do with the API:
 • Monitor your frame — check battery, WiFi signal, firmware version, and display status
 
 • Control your frame — restart it, put it to sleep, or refresh the display
 
 • Display custom artwork — upload .bin image files directly to the frame
 
 • Integrate with Home Assistant — add Fraimic sensors and controls to your smart home dashboard

The API is designed for Home Assistant users, tinkerers, and anyone who wants to automate their Fraimic
beyond the normal tap-and-speak experience. If you’re just using the frame normally, you don’t need this
guide — everything works through the built-in portal and voice recording.

## Getting Started

### Finding Your Frame’s Address

Every API request starts with your frame’s network address. There are two ways to reach it:

<img width="589" height="107" alt="Screenshot 2026-03-31 at 12 08 26 PM" src="https://github.com/user-attachments/assets/4574fac7-cf5c-4530-8e97-f47f13800057" />

You can find the frame’s IP address on the portal at fraimic.local/info, or in your router’s DHCP client
table.
Important: Your frame must be awake and connected to WiFi. When it’s in deep sleep, it’s completely
unreachable — tap the frame to wake it first.

## Making Requests
The API uses standard HTTP. You can use any tool that speaks HTTP — curl from the command line, a
web browser, Home Assistant’s REST integration, or any programming language.

**• GET** requests retrieve information (battery status, device info)

**• POST** requests trigger actions (restart, sleep, refresh, upload)

No authentication is required. Anyone on your local network can access these endpoints.

## A Quick Test
Open your browser and navigate to:
http://fraimic.local/api/info

You should see a JSON response with your frame’s current status. If that works, you’re ready to use any of
the endpoints below.

<img width="344" height="403" alt="Screenshot 2026-03-31 at 12 10 40 PM" src="https://github.com/user-attachments/assets/d1963209-7bd2-485f-9262-282ffe26d38a" />



# Home Assistant Integration
<img width="444" height="183" alt="Screenshot 2026-03-31 at 12 11 40 PM" src="https://github.com/user-attachments/assets/4aecb211-fa3d-45a8-9d6b-942a9cbf3012" />

This section walks you through adding Fraimic sensors and controls to your Home Assistant dashboard. By
the end, you’ll have battery monitoring, WiFi signal tracking, and one-tap buttons for restart, sleep, refresh,
and image upload.

## Step 1: Add REST Sensors
Open your Home Assistant configuration.yaml file and add the following block. This creates sensors that
poll your frame every 5 minutes:

```
rest:
  - resource: http://fraimic.local/api/info
    scan_interval: 300
    sensor:
    - name: "Fraimic Battery"
      value_template: "{{ value_json.battery.percent }}"
      unit_of_measurement: "%"
      device_class: battery
    - name: "Fraimic WiFi Signal"
      value_template: "{{ value_json.wifi.rssi }}"
      unit_of_measurement: "dBm"
      device_class: signal_strength
    - name: "Fraimic Firmware"
      value_template: "{{ value_json.firmware_version }}"
    - name: "Fraimic Charging"
      value_template: "{{ value_json.battery.charging }}"
    - name: "Fraimic IP"
      value_template: "{{ value_json.wifi.ip }}"
```

## Step 2: Add Shell Commands
These give Home Assistant the ability to send action commands to your frame:

```
shell_command:
  fraimic_restart: >-
    curl -s -X POST http://fraimic.local/api/restart
  fraimic_sleep: >-
    curl -s -X POST http://fraimic.local/api/sleep
  fraimic_refresh: >-
    curl -s -X POST http://fraimic.local/api/refresh
  fraimic_upload: >-
    curl -s -X POST
    -H "Content-Type: application/octet-stream"
    --data-binary @/config/www/fraimic/{{ filename }}
    http://fraimic.local/api/image
```

**Note on the upload command:** Place your .bin image files in /config/www/fraimic/ on your Home
Assistant server. The {{ filename }} variable is passed when you call the command from a script or
automation.

## Step 3: Reload — No Restart Required
After saving configuration.yaml, go to Developer Tools > YAML in Home Assistant and reload:

• REST entities

• Shell Commands

A full Home Assistant restart is not needed. Your new sensors and commands will appear immediately.

## Step 4: Create Scripts for Dashboard Buttons

Go to Settings > Automations & Scenes > Scripts > Add Script, or add directly to your
scripts.yaml:
```
fraimic_restart:
  alias: "Fraimic Restart"
  icon: mdi:restart
  sequence:
  - service: shell_command.fraimic_restart

fraimic_sleep:
  alias: "Fraimic Sleep"
  icon: mdi:sleep
  sequence:
  - service: shell_command.fraimic_sleep

fraimic_refresh_display:
  alias: "Fraimic Refresh Display"
  icon: mdi:monitor
  sequence:
  - service: shell_command.fraimic_refresh
fraimic_display_dinosaur:
  alias: "Display Dinosaur"
  icon: mdi:google-downasaur
  sequence:
  - service: shell_command.fraimic_upload
    data:
      filename: dinosaur.bin
```

## Step 5: Add Dashboard Buttons
Add these button cards to your Fraimic area dashboard for one-tap control:


```
views:
  - cards:
      - show_name: true
        show_icon: true
        type: button
        entity: script.fraimic_restart
      - show_name: true
        show_icon: true
        type: button
        entity: script.fraimic_sleep
      - show_name: true
        show_icon: true
        type: button
        entity: script.fraimic_refresh_display
      - show_name: true
        show_icon: true
        type: button
        entity: script.display_dinosaur

```

## For more info including automation ideas, Troubleshooting and Endpoint Info see our complete [Home Assistant & Rest API Guide](https://drive.google.com/file/d/1fUTzi31iUK2e9K2TmfDHFkUWmTx56qLZ/view?usp=drive_link)



**Find Us:** [Official Website](https://fraimic.com) | [API Docs](https://drive.google.com/file/d/1fUTzi31iUK2e9K2TmfDHFkUWmTx56qLZ/view)| [Business Contact](mailto:support@fraimic.com)



© 2025 FRAIMIC. All rights reserved.
