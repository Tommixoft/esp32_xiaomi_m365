# esp32_xiaomi_m365
Sample Project for decoding serial bus data on a Xiaomi M365 Scooter on Espressif ESP32 / ESP8266:
- Supports one or two 0.96" 128x64 SSD1306 based OLED screen(s) (SSD1351/128x128 Color OLED support to follow)
- ESP32/Arduino Code
- ESP8266/Arduino works as well, right now no ESP32-specific features are used (might come later)
- OTA for Firmwareupdates
- Telnet Screens for debugging
- Oled Display with different Screens in Drive/Charge/Stop Mode
- use Throttle in stopped mode to Swipe through different screens

# Done
- WiFi Auto Connect to known SSIDs or AP-Mode & Telnet with Timeouts to auto-turnoff Telnet/WiFi
- Firmwareupdates (of ESP Device, not the scooter) over WiFi
- Telnet on Port 36523 with different Screens
- Telnet on Port 36524 with raw byte dump from M365 Bus ("read only")
- M365 Serial Receiver and Packet decoder into Array per Address
- Decoder of BMS Data Array to Variable Values/Telemetrie Screen verified against known apps
- Decoder of ESC Data Array to Variable Values/Telemetrie Screen verified against known apps
- Data Requestor
- Subscription to interesting data fields needed for client-app (e.g. OLED Display)
- Basic screens (nicer layout to follow) for drive/stall/charging
- Added Loop-Timers and Counters for debugging Receiver/Requestor & Oled-Draw/Transfer Durations
- OLED Shows LED/Light Status (to recognize accidental switched on Light @ noon while switching normal/eco mode)
- added handling of 2nd OLED screen, used library has some problems with it -> disabled
- added ESP32 Status Screen
- reformated driving-screen
- status screen for ota-firmwareupdate
- added boilerplate screen in stop mode
- fixed display lock by wlan-search
- added charge subscreen with cell voltages
- Telnet on Port 36525 with packet decode/dump from M365 Bus ("read only")
- compiles on ESP8266, UART/Statemachines/OLED untestet on this plattform
- verified function on ESP8266

# Todos
 - fix - data-requestor timing currently causes ~10% crc errors on m365 bus
 - OLED: Add Popup Messages for Events (e.g. Scooter Error, Temp, BMS CellVoltage variations > treshold, WLAN/BLE On/Off, Client Connected,...)
 - Add Menu/Change functionality for "Config" Screen Items
 - Add Background housekeeper task 
   - Check Scooter Error Register
   - Firmware-Flash Protection
   - Alert User if Cell-Voltages difference is above a treshold
 - custom PCB with 2 OLEDs, ESP32S, VReg --> in production
 - add display-sleeptimer - x seconds after last event (gas/brake/throttle/speed/chargerun-plug/telnet/AP-Client Connection & speed = 0)
 - advanced thief/lock protection
 - fix speed for int16 overflow in m/h -> faster than 31.7 -> overflow

# further Ideas & Visions:
 - add Scooter-Flashing Protection (so no one can flash broken firmware to your scooter while waiting at a red traffic light
 - advanced trip computer (which keeps trip-totals/averages between 2 charge cycles or 2 times with the same available SSID (leaving/coming home)
 - MQTT Logging of Trip-Summary Data
 - Navigation Arrow Display e.g. with Komoot (https://github.com/komoot/BLEConnect)
 
# Telnet Interface
 - Telemetrie Screen shows decoded known values: Batt Voltage, Current, Speed,... 
 - Statistics Screen dumps some internal counters, e.g. Packet Counters, CRC Files, Timing,...
 - ESC/BLE/BMS/"X1" RAW Screens dumps the 512 Byte Array for each device (format "00 00 ...")
 - ESC/BLE/BMS/"X1" Array Screens dumps the 512 Byte Array for each device (in copy & paste format "const bledata[512]={0,0,0...};")

use the letters
 - s,t,e,b,n,x,E,B,N,X to switch between the screens
 - r to reset statistics & m365 data arrays

# Wiring
M365 has a Serial One Wire Bus between BLE Module and ESC which consists of 4 wires, the connection as seen on the BLE Module:
- Ground  (Black, "G")
- One Wire Serial Connection (Yellow, "T", 115200bps, 8n1)
- VBatt (Green, "P", always available)
- 5V (Red, "5", only when scooter is turned on)

ESP32/8266 needs a Vcc of 3.3V, while at the same time the GPIO Pins are 5V save, so you can wire the 5V to a Vreg for 3.3v which feed the ESP, while the Serial Connection can be wired to RX/TX Pins.
It might be a idea to use e.g. 680R or 1k in series to protect the gpio, as well as add a diode from rx in series with a ~100-200R towards TX

# possible Issues and Hints
 - Adafruit_SSD1306 uses 100kHz I2C Clock per default and does not support individual GPIO Pins for Clock and Data. forked & fixed version: https://github.com/smartinick/Adafruit_SSD1306
 - arduino-esp32 core implementation of HardwareSerial and esp32-hal-uart only trigger a uart-rx event/interrupt every 112 bytes which makes it impossible to stay within the timing necersarry for the m365 one-wire-uart. forked & fixed version: https://github.com/smartinick/arduino-esp32
