### Manual

If you don't like the automated installation here are manual installation instructions (missing the creation of virtual environment).

1. Install pip if you don't have it:

```bash
sudo apt install python-pip
```

2. Then install this python module needed for the script:

```bash
pip install paho-mqtt==1.6.1
```

3. Install git if you don't have it:

```bash
apt install git
```

4. Clone the repository:

```bash
git clone https://github.com/hjelev/rpi-mqtt-monitor.git
```

5. Rename ```src/config.py.example``` to ```src/config.py```



## Configuration

(only needed for manual installation)
Populate the variables for MQTT host, user, password and main topic in ```src/config.py```.

You can also choose what messages are sent and what is the delay (sleep_time is only used for multiple messages) between them.
If you are sending a grouped message, and you want to delay the execution of the script you need to use the ```random_delay``` variable which is set to 1 by default.
This is the default configuration (check the example file for more info):

```
random_delay = randrange(1)
discovery_messages = True
group_messages = False
sleep_time = 0.5
service_sleep_time = 120
cpu_load = True
cpu_temp = True
used_space = True
voltage = True
sys_clock_speed = True
swap = True
memory = True
uptime = True
uptime_seconds = False
wifi_signal = False
wifi_signal_dbm = False
rpi5_fan_speed = False
display_control = False
shutdown_button = True
restart_button = True
```

If ```discovery_messages``` is set to true, the script will send MQTT Discovery config messages which allows Home Assistant to automatically add the sensors without having to define them in configuration.  Note, this setting is only available when ```group_messages``` is not used.

If ```group_messages``` is set to true the script will send just one message containing all values in CSV format.
The group message looks like this:

```
1.3, 47.1, 12, 1.2, 600, nan, 14.1, 12, 50, -60
```

### External Sensors
External sensors (currently DS18B20 for temperature and SHT21 for temperature and humdity) can be read out.
Therefore the ```ext_sensor```key in the config file must be configured.
A list of sensors with properties ```[name, sensor_type, ID, default_value]``` must be given.
The default_value is returned if the Raspberry fails to read the external sensor. The default value is either a scalar or a list, e.g. temperature and humidity. Examples:
* 1x ds18b20 sensor located in the RPi housing: ```ext_sensors = [["Housing", "ds18b20", "0014531448ff", -300]]```
* If the ID in unkown and if there is only 1 ds18b20 sensor, then use: ```ext_sensors = [["Housing", "ds18b20", 0, -300]]```
* If there are two sensors, one DS18B20 and one SHT21 (via I2C), then use: ```ext_sensors = [["Housing", "ds18b20", "0014531448ff", -300], ["Outside", "sht21", 0, [-300, 0]]]```, where the -300 is the default value for temperature and 0 is the default value for humidity.

default option is ```False```


## Test Raspberry Pi MQTT Monitor

Run Raspberry Pi MQTT Monitor (this will work only if you used the automated installer or created the shortcut manually)

```bash
rpi-mqtt-monitor -d
```

Once you run Raspberry Pi MQTT monitor you should see something like this:

```
:: rpi-mqtt-monitor
   Version: 0.9.1

:: Device Information
   Model Name:  Intel(R) Pentium(R) Silver J5040 CPU @ 2.00GHz
   Manufacturer:  GenuineIntel
   OS: Ubuntu 23.10
   Hostname: ubuntu-pc
   IP Address: 192.168.0.200
   MAC Address: A8-A1-59-82-57-E7
   Update Check Interval: 3600 seconds

:: Measured values
   CPU Load: 48.5 %
   CPU Temp: 71 °C
   Used Space: 12 %
   Voltage: False V
   CPU Clock Speed: False MHz
   Swap: False %
   Memory: 53 %
   Uptime: 0 days
   Wifi Signal: False %
   Wifi Signal dBm: False
   RPI5 Fan Speed: False RPM
   Update: {"installed_ver": "0.9.1", "new_ver": "0.9.1"}
```
## Schedule Raspberry Pi MQTT Monitor execution as a service

If you want to run Raspberry Pi MQTT Monitor as a service you can use the provided service file.
You need to edit the service file and update the path to the script and the user (if you want to use shutdown or restart buttons user needs to be root) that will run it. 
Then copy the service file to ```/etc/systemd/system/``` and enable it:

```bash
sudo cp rpi-mqtt-monitor.service /etc/systemd/system/
sudo systemctl enable rpi-mqtt-monitor.service
```

To test that the service is working you can run:

```bash
sudo service rpi-mqtt-monitor start
sudo service rpi-mqtt-monitor status
```

## Schedule Raspberry Pi MQTT Monitor execution with a cron

Create a cron entry like this (you might need to update the path in the cron entry below, depending on where you installed it):

```
*/2 * * * * cd /home/pi/rpi-mqtt-monitor; /usr/bin/python /home/pi/rpi-mqtt-monitor/rpi-cpu2mqtt.py
```