
Buy

1. Aquara Zigbee Water Leak Sensor
https://www.aliexpress.com/item/1005009154171954.html?spm=a2g0o.order_list.order_list_main.23.1fdd1802Vtx2TT

2. Universal Car Seat Occpuied Sensor Alarm Pad
https://www.aliexpress.com/item/1005012629128239.html?spm=a2g0o.order_list.order_list_main.17.1fdd1802Vtx2TT


A. I suggest putting a fresh good quality CR2320 battery into the Leak Sensor as the ones they come with are very cheap

B.
Cut the plug off the end of the Car Seat sensor lead
Strip the two wires
Unscrew one terminal on the Water Leak Sensor, wind the red wire around it and tighten it. Optionally solder it 
Do the same with the other terminal and the white wire

C 
Add the Water Sensor to your Zigbee network

D
In Zigbee2MQTT rename it 'Bed Sensor Main Bedroom', and tick 'rename entities'

E
Go to Settings/Devices/Entities
Find Bed Sensor Main Bedroom Moisture (bed_sensor_main_bedroom_water_leak) and click its settings
Change
- 'Name' to Occupancy
- 'Icon' to mdi:bed
- 'Show as' to Occupancy
- 'Entity ID' to bed_sensor_main_bedroom_occupancy

F
See automation.yaml for an example automation

