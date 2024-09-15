# ESPHome-Thermostate-Moes-BHT-002


Moes-BHT-002 WIFI thermostats have become very popular among home automation enthusiasts.

![1](https://user-images.githubusercontent.com/11642286/208393574-359c222d-f9fa-4e75-8af6-3d01674839ef.jpg)

The full name is BHT-002-GBLW. This is a wi-fi electric floor heating thermostat from the Tuya ecosystem.
Of course it can be used in the native ecosystem. It can be connected to other systems using various plugins and junctions.
I use Home Assistant and to connect I had the following options:
- Tuya Integration https://www.home-assistant.io/integrations/tuya/
- Local Tuya Integration https://github.com/rospogrigio/localtuya
- Overwire using the project https://github.com/klausahrenberg/WThermostatBeca and connect via MQTT
- There is also STANDARD integration https://esphome.io/components/tuya.html

All these methods work, but they have disadvantages. In some cases you need to be connected to external servers,
in some cases you will not be able to use the full potential of the device, in some cases there will be errors
and confusion. And none of these methods will be 100% under your control. After studying the last project
I came to the conclusion that it would not be a bad idea to write a component for ESPHome.
Especially since the Tywe3S module is responsible for communication in this device, which turned out to be an
analog of ESP-12. The thermostat has a processor that serves the heating and control process and communicates
with the Tuwe3S module via UART. Naturally we could not find a complete description of the protocol, in the
project WThermostatBeca was described only the service exchange, the rest is not. We had to decipher the protocol,
helped by the fact that it is simple and made in Chinese style :)))

The output is firmware, which is native to ESPHome. As always, the main problem is initially pouring it into Tywe3S.
I know that there is a project that allows you to replace the firmware in Tuya devices without soldering anything.
It's easier for me to solder. It's a standard circuit:

![image](https://user-images.githubusercontent.com/11642286/208415302-ad267f5a-6b39-47d1-96ba-4d81af84e8c5.png)

Of course a completely correct scheme, roughly like this:
![image](https://user-images.githubusercontent.com/11642286/208404195-f8384adb-0c07-4520-aa52-4a8dff1be936.png)

But I decided to do without resistors "for a couple of flashes". I suggest you still use resistors.
The front panel is unbolted from the thermostat. External power supply is 3.3 volts.
It is necessary to short-circuit RESET to GND for a short time before filling the firmware.

Device Essences:

![image](https://user-images.githubusercontent.com/11642286/208405764-30eaf520-fde5-4c25-b54b-a31cc241caed.png)

The clock is synchronized automatically if a time source is set.

Configuration example:

```yaml
climate:
  - platform: tuya_termo
    name: devicename
    uart_id: id_uart_bus
    time_id: id_sync_time
    mcu_reset_pin: GPIOxx
    optimistic: true
    visual:
      min_temperature: 5
      max_temperature: 35
      eco_temperature: 20
      overheat_temperature: 45
      deadzone_temperature: 1
    internal_temperature:
      name: Internal Temperature
    external_temperature:
      name: External Temperature
    children_lock:
      name: Child Lock
    shedule:
      selector:
        name: Plan Day Selector
      hours:
        name: Plan Hours
      minutes:
        name: Plan Minutes
      temperature:
        name: Plan Temperatures
    product_id:
      name: Product Identifier
```

**UPD (20.09.23):**

There are now versions with CB3S module, based on BK7231N. At the moment there is a great project LibreTiny ESPHome,
I hope you will find it easily. With its help you can compile the firmware for a new module.
The code has been updated and corrected to work correctly with new versions of thermostats with BK72xx chips.
New thermostats do not publish the temperature of the remote sensor, so for some users it will be in the status "Unknown", just hide it in the configuration.

**I would like to thank Michael Mivlz for his help in debugging the code.**

TUYA modules on the BK72xx chip can be flashed in almost the same way as the ESP8266, except that at the very beginning of the boot you need to put the CEN
pin to ground for about a second. Of course, do not forget that you need LibreTiny ESPHOME.

Connection for flashing:

![image](https://github.com/Brokly/ESPHome-Thermostate-Moes-BHT-002/assets/11642286/922be46b-6c74-49d4-aa16-962a422c1779)

Unzip the chip from the manufacturer's website:

![image](https://github.com/Brokly/ESPHome-Thermostate-Moes-BHT-002/assets/11642286/49673fa9-3ab6-495a-9b32-de093d7249b6)

Foot assignment table:

![image](https://github.com/Brokly/ESPHome-Thermostate-Moes-BHT-002/assets/11642286/0e64e196-e72c-41dc-b0a4-bbcccd5d0c9b)

**UPD (18.12.23):**

Due to the slow response of the thermostats on beckens, a setting has been added
```yaml
- platform: tuya_termo
  оptimistic: true # оr false
```
The default setting is true. If this option is enabled, the data publication always corresponds to the user setting, regardless of whether the thermostat has applied the setting.
In case of a non-applicable setting, it will be updated to the actual state after some time.

**UPD (02.03.24):**

Added setting:
```yaml
- platform: tuya_termo
  mode_restore: true # оr false
```
The default setting is false. The setting enables auto-restore of the thermostat's operating mode after power off. It only restores the operating mode - Mode.
Now the WIFI and server connection status transmitted to the thermostat corresponds to the real situation. Previously the status was fictitious.

**UPD (07.03.24):**

Now when restoring the operating mode after a power failure (mode_restore: true), all operational parameters of the thermostat (operating mode, preset, target temperature) are restored.

Added an output for resetting the thermostat MCU. If there is no response from the thermostat MCU for a long time, an inverse reset pulse is generated at the output (shorted to GND).
In normal state the output is in HI-IMPEDANCE mode. To use the thermostat reset function, hardware modifications are required. It is necessary to run a wire from DC12->DC5 modem control
to the thermostat output leg.

![image](https://github.com/Brokly/ESPHome-Thermostate-Moes-BHT-002/assets/11642286/986822d7-a11e-4a6a-9523-2422d72b2ae4)

My version of the thermostat uses an inverter [LP6498A](http://www.lowpowersemi.com/storage/files/2023-05/7af3ba3edb6fc52b54c51add36301d7e.pdf)

Connection point for dc/dc converter operating mode control
![image](https://github.com/Brokly/ESPHome-Thermostate-Moes-BHT-002/assets/11642286/62a75737-76d4-4040-a4ac-c83248175eaf)![image](https://github.com/Brokly/ESPHome-Thermostate-Moes-BHT-002/assets/11642286/44d7258b-4a69-4528-916b-e9c49c1f42ee)



```yaml
- platform: tuya_termo
  mcu_reset_pin: GPIOXX
```

Changed configuration of schedule controllers:
```yaml
- platform: tuya_termo
  shedule:
    selector:
      name: "Shedule Day Selector"
    hours:
      name: "Shedule Hours"
    minutes:
      name: "Sheduule Minutes"
    temperature:
      name: "Shedule Temperatures"
```

<b>UPD (12.03.24):</b>

Added reboot counter. The counter counts the number of reboots and is reset to zero when flashing via OTA. A reboot is attempted if the thermostat MCU does not respond for a long time.
If three attempts to re-establish communication are unsuccessful, a complete reboot of the communication module is performed.
```yaml
- platform: tuya_termo
  mcu_reload_counter:
    name: MCU Reset Counter
```

Added a setting to enable/disable time synchronization packets. Some thermostats, for the clock to work properly, require every second packets that allow the clock to go, otherwise the clock stands still.
By default, the setting is disabled (false) and no every-second packets are sent. Below is an example of a configuration with every-second packets sent enabled
```yaml
- platform: tuya_termo
  time_id: sync_time_id
  time_sync_packets: true
```

Added settings for additional outputs. Some devices require control via pins. This is the reset input of the reset_pin protocol, upon receiving a change in signal level on this pin, the control module re-initializes,
with all settings being transferred as if the device had just been turned on. Also output status_pin, presumably to this pin should be connected LED device, indicating the mode of operation of the network.
These pins can be added to the configuration:
```yaml
- platform: tuya_termo
  status_pin: Pxx
  reset_pin: Pxx
```
But usually these pins need to be configured as required by the thermostat MCU. If the thermostat requires such a configuration, ERROR and/or WARNING level messages will be logged indicating the required pins.

Approximate complete configuration (not everything in it you need):
```yaml
climate:
  - platform: tuya_termo
    name: ${upper_devicename}
    uart_id: uart_bus # uart bus for controlling the thermostat MCU
    time_id: sync_time # time synchronization source
    time_sync_packets: false # time synchronization packets, for some device versions (false by default)
    optimistic: true
    mode_restore: true # setting to restore operation mode after reboot (default is true)
    mcu_reset_pin: P9  # Thermostat MCU forced reset output pin (only for retrofitted thermostats)
    reset_pin: P14 # input pin for reinitializing the communication protocol with MCU, required for some thermostats
    status_pin: P15 # output pin for network status indication, required for some thermostats, but LED can be installed
    mcu_reload_counter: # counter of forced restarts of the thermostat MCU
      name: ${upper_devicename} MCU Reset Counter
    visual: # settings for proper widget display, in most cases defaults are used
      min_temperature: 5
      max_temperature: 35
      eco_temperature: 20
      overheat_temperature: 45
      deadzone_temperature: 1
    internal_temperature: # room temperature sensor
      name: ${upper_devicename} Internal Temperature
    external_temperature: # external floor temperature sensor
      name: ${upper_devicename} External Temperature
    children_lock:
      name: ${upper_devicename} Child Lock
    shedule: # set of controllers for managing the thermostat's autonomous operation schedule
      selector:
        name: ${upper_devicename} Plan Day Selector
      hours:
        name: ${upper_devicename} Plan Hours
      minutes:
        name: ${upper_devicename} Plan Minutes
      temperature:
        name: ${upper_devicename} Plan Temperatures
    product_id:
      name: ${upper_devicename} Product Identifier
```
