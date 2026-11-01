# DIY Zigbee Bed Occupancy Sensor

Convert an **Aqara Zigbee water leak sensor** and an inexpensive **car seat pressure pad** into a battery-powered bed occupancy sensor for **Zigbee2MQTT** and **Home Assistant**.

When someone lies on the pressure pad, its contacts close. The Aqara sensor reports this as a water leak, which Home Assistant can instead display as **Occupied**. When the pad is released, it reports **Clear**.

> **Note:** This is a DIY convenience sensor, not a safety or medical monitoring device. Test it with your mattress before relying on it for automations.

## Parts and tools

| Item | Link / details |
| --- | --- |
| Aqara Zigbee Water Leak Sensor | [AliExpress](https://www.aliexpress.com/item/1005009154171954.html) |
| Universal Car Seat Occupancy Sensor Pad | [AliExpress](https://www.aliexpress.com/item/1005012629128239.html) |
| CR2032 battery | Fresh, good-quality replacement recommended |
| Wire strippers and small screwdriver | For preparing and connecting the wires |
| Soldering iron and solder | Optional |

## 1. Prepare the Aqara sensor

1. Open the Aqara sensor and replace the supplied battery with a fresh **CR2032** if necessary.
2. Locate the two metal screw terminals on its underside.
3. Confirm the pressure pad acts as a normally open switch: without pressure its contacts should be open; with pressure they should close.

## 2. Connect the pressure pad

1. Cut the connector plug off the car seat pressure pad cable.
2. Strip a short length of insulation from the **red** and **white** wires.
3. Loosen one of the Aqara sensor's screw terminals.
4. Wrap the red wire around the terminal and tighten it securely. Optionally to prevent the wires coming loose, solder the connections or use a hot glue gun to glue them.
5. Repeat with the white wire on the other terminal.
6. Ensure the bare wires cannot touch each other directly and that both connections are secure.

The pad is a simple contact switch, so **wire polarity does not matter**. Avoid overheating the sensor if soldering.

### Expected electrical behaviour

| Pad condition | Contact state | Aqara reading |
| --- | --- | --- |
| No pressure | Open | Dry / no leak |
| Pressure applied | Closed | Wet / leak |

## 3. Pair with Zigbee2MQTT

1. Enable **Permit join** in Zigbee2MQTT.
2. Put the Aqara water leak sensor into pairing mode, following its pairing instructions.
3. Wait for it to appear in Zigbee2MQTT.
4. Test the pressure pad and verify the leak state changes when pressure is applied and released.

## 4. Rename the device

In Zigbee2MQTT, rename the device to:

`Bed Sensor Main Bedroom`

Tick **Rename entities** and save.

## 5. Configure Home Assistant

Go to **Settings → Devices & services → Entities**.

Find **Bed Sensor Main Bedroom Moisture**, initially something like:

`binary_sensor.bed_sensor_main_bedroom_water_leak`

Open its settings and change:

| Setting | Value |
| --- | --- |
| Name | `Occupancy` |
| Icon | `mdi:bed` |
| Show as | `Occupancy` |
| Entity ID | `binary_sensor.bed_sensor_main_bedroom_occupancy` |

Save. The entity should now show **Occupied** when the pad is pressed and **Clear** when released. The original battery entity remains available.

> If **Show as** is not available for your entity, create a template binary sensor with `device_class: occupancy` instead.

## 6. Alexa announcement automation

Add the following to `automations.yaml`. It uses the same `notify.alexa_media` announcement format as other Alexa Media Player automations, targeting only **Living Room Echo Show**.

```yaml
- alias: "Bed Occupancy - Alexa Announcement"
  id: bed_occupancy_alexa_announcement
  description: "Announce main bed occupancy changes on Living Room Echo Show"
  mode: single

  trigger:
    - platform: state
      entity_id: binary_sensor.bed_sensor_main_bedroom_occupancy
      from: "off"
      to: "on"
      for:
        seconds: 2
      id: occupied

    - platform: state
      entity_id: binary_sensor.bed_sensor_main_bedroom_occupancy
      from: "on"
      to: "off"
      for:
        minutes: 2
      id: unoccupied

  action:
    - service: notify.alexa_media
      data:
        target:
          - media_player.living_room_echo_show
        message: >-
          {% if trigger.id == 'occupied' %}
            The bed is now occupied.
          {% else %}
            The bed is now unoccupied.
          {% endif %}
        data:
          type: announce
```

Reload automations in Home Assistant after saving.

### Announcement behaviour

| Event | Requirement | Announcement |
| --- | --- | --- |
| Bed becomes occupied | Continuously occupied for 2 seconds | “The bed is now occupied.” |
| Bed becomes unoccupied | Continuously empty for 2 minutes | “The bed is now unoccupied.” |
| Brief movement or pressure loss | Does not satisfy the delay | None |
| Sensor becomes unavailable | Not a valid `on`/`off` transition | None |

> **Avoid double delays:** If the entity is already a template sensor with `delay_on` or `delay_off`, remove the corresponding `for:` blocks above unless you deliberately want both delays.

## 7. Install and test

1. Position the pressure pad under the mattress where it reliably detects the sleeper's weight.
2. Route the cable to avoid pinching, sharp bends or repeated pulling.
3. Place the Aqara sensor somewhere accessible for battery replacement.
4. Get into and out of bed and watch the occupancy entity in Home Assistant.
5. Adjust the pad position if it does not reliably register pressure.
6. Verify the Alexa announcements occur after the configured delays.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| Always **Clear** | Check pad position, connections and whether the pad closes under pressure. |
| Always **Occupied** | Check for shorted wires or a pad that remains compressed. |
| Rapid state changes | Reposition the pad or increase the automation's unoccupied delay. |
| No Alexa announcement | Confirm `notify.alexa_media` is available and `media_player.living_room_echo_show` is the correct target. |
| Wrong entity name | Check the actual entity ID under Home Assistant **Settings → Devices & services → Entities**. |
| No battery readings | Check the device's battery entity and allow time for its next Zigbee update. |

## Result

A low-cost Zigbee bed occupancy sensor with Home Assistant occupancy states, battery reporting and optional Alexa announcements—without firmware modifications or additional electronics.

