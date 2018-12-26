# ir-blaster

[<img src="https://spielhuus.github.io/ir-blaster/perfboard.png" alt="perfboard" width="350" height="275">](https://spielhuus.github.io/ir-blaster/perfboard.png)
[<img src="https://spielhuus.github.io/ir-blaster/photo.jpg" alt="photo" width="500" height="275">](https://spielhuus.github.io/ir-blaster/photo.jpg)

### Components

|Amount        | Type           | Description  |
|:------------- |:-------------|:-----|
| 1 | [ESP-12E](https://www.sparkfun.com/datasheets/Sensors/Temperature/DHT22.pdf)|esp8266 board|
| 1 | D1|Infrared LED|
|1 | R2|Resistor 1k Ω|
| 1 | R2|Resistor 100 Ω|
|1 | Q2|Transistor 2n2222|
|1 |[TSOP4138](https://spielhuus.github.io/ir-blaster/tsop45.pdf)|IR Receiver Modules for Remote Control Systems|

## Installation

The Arduino IDE has to be properly installed. 

Add the Libraries to you IDE:

- Install Arduino core for ESP8266 WiFi chip
  - Under Preferences > Additional Boards Manager URLs: http://arduino.esp8266.com/stable/package_esp8266com_index.json
  
    Multiple managers can be separated with a comma.

- Open Boards Manager from Tools > Board menu and install esp8266 platform (and don't forget to select your ESP8266 board from Tools > Board menu after installation).

- Install TinyLoRa-BME280 v1.1
  - Download the ZIP archive from [TinyLoRa-BME280 v1.1](https://github.com/ClemensRiederer/TinyLoRa-BME280_v1.1)
  - Install the ZIP archive in the Arduino Library Manager

- Open Examples > TinyLoRa-BME280_v1.1-master > ATtiny_LoRa_BME280
  - Edit NwkSkey, AppSkey, DevAddr
  - Change the Spreading Factor in ATtinyLoRa.cpp
  - The Sleep time can be set with the SLEEP_TOTAL var.

- Burn the sketch to the Chip using an Arduino UNO [2]
  - Burn a bootloader first to set the fuses correctly
  
  
Edit the sketch and configure your MQTT server. Then configure Arduino-IDE for esp8266 and upload the sketch.

```
#ifdef MQTT_ENABLE
// Address of your MQTT server.
#define MQTT_SERVER "<MQTT SERVER IP>"
#define MQTT_PORT <MQTT SERVER PORT>
// Set if your MQTT server requires a Username & Password to connect.
const char* mqtt_user = "";
const char* mqtt_password = "";
#define MQTT_RECONNECT_TIME 5000  // Delay(ms) between reconnect tries.
```

### hass.io integration

Create an MQTT switch in your <pre>configuration.yaml</pre>.

Switch for Samsung TV where the on/off action is dependent of the tv state.

```
switch:
  - platform: template
    switches:
      tv:
        value_template: "{{ is_state('sensor.tv', 'on') }}"
        turn_on:
          service: switch.turn_on
          data_template:
            entity_id: >
              {% if is_state('sensor.tv', 'off') %}
                switch.tv_mqtt_switch
              {% else %}
                switch.null
              {% endif %}

        turn_off:
          service: switch.turn_off
          data_template:
            entity_id: >
              {% if is_state('sensor.tv', 'on') %}
                switch.tv_mqtt_switch
              {% else %}
                switch.null
              {% endif %}

  - platform: mqtt
    name: "TV mqtt Switch"
    icon: mdi:television
    #state_topic: ""
    command_topic: "ir_server/send"
    #availability_topic: "home/bedroom/switch1/available"
    payload_on: "7,E0E040BF"
    payload_off: "7,E0E040BF"
    state_on: "on"
    state_off: "off"
    optimistic: false
    qos: 0
    retain: false
```

### Links

* [ESP8266-HTTP-IR-Blaster](https://github.com/mdhiggins/ESP8266-HTTP-IR-Blaster)
* [IRremote ESP8266 Library](https://github.com/markszabo/IRremoteESP8266)

### License

[Boost Software License](http://www.boost.org/LICENSE_1_0.txt) - Version 1.0 - August 17th, 2003
