
# Before Starting
[Install Dell IPMI Tool](https://www.dell.com/support/home/en-us/drivers/driversdetails?driverid=m63f3)

Open PowerShell and CD to the tool directory:
```PowerShell
cd "C:\Program Files (x86)\Dell\SysMgt\bmc"
```


# Set Manual Fan Speed
Enable Manual Fan Control:
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> raw 0x30 0x30 0x01 0x00
```
- The argument `-I` specifies the interface type to use `lanplus` which enables IPMI v2.0 with RMCP+ (Remote Management Control Protocol Plus), which provides support for encryption and authentication over the network, making it more secure than the older lan interface (IPMI v1.5), which doesn’t support encryption.
- If you were to use `-I lan` instead, the communication would be unencrypted, which is generally not recommended for remote management in production environments.


Set All Fans (0xff) to 16% (0x10)
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> raw 0x30 0x30 0x02 0xff 0x10
```
- **This will lock the fan speed at the specified % value**. The fan will not automatically ramp up with increased temperature.
- Check in iDracs that the thermals are in acceptable range
	- **Normal workload**: A safe temperature range is typically between 40–65°C (104–149°F).
	- **Bad temperature**: A temperature of 80-85°C (176–185°F) or above is generally considered bad.


# View Temperatures 

## Report Temperatures
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> sdr type temperature
```
output
```
Inlet Temp       | 04h | ok  |  7.1 | 21 degrees C
Exhaust Temp     | 01h | ok  |  7.1 | 34 degrees C
Temp             | 0Eh | ok  |  3.1 | 43 degrees C
Temp             | 0Fh | ok  |  3.2 | 38 degrees C
```
- **Inlet Temp** - is the temperature of air entering the chassis
- **Exhaust Temp** - is the temperature of air existing the chassis
- **Temp** - The temperature of CPU 1
- **Temp** - The temperature of CPU 2

## Report Only Temp, Volt & Fan Sensors
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> sdr elist full
```
output
```
Fan1 RPM         | 30h | ok  |  7.1 | 5280 RPM
Fan2 RPM         | 31h | ok  |  7.1 | 5160 RPM
Fan3 RPM         | 32h | ok  |  7.1 | 5160 RPM
Fan4 RPM         | 33h | ok  |  7.1 | 5040 RPM
Fan5 RPM         | 34h | ok  |  7.1 | 5160 RPM
Fan6 RPM         | 35h | ok  |  7.1 | 5280 RPM
Inlet Temp       | 04h | ok  |  7.1 | 21 degrees C
CPU Usage        | FDh | ok  |  7.1 | 0 percent
IO Usage         | F1h | ok  |  7.1 | 0 percent
MEM Usage        | F2h | ok  |  7.1 | 0 percent
SYS Usage        | F3h | ok  |  7.1 | 0 percent
Exhaust Temp     | 01h | ok  |  7.1 | 34 degrees C
Temp             | 0Eh | ok  |  3.1 | 44 degrees C
Temp             | 0Fh | ok  |  3.2 | 38 degrees C
Current 1        | 6Ah | ok  | 10.1 | 1.20 Amps
Current 2        | 6Bh | ok  | 10.2 | 0.80 Amps
Voltage 1        | 6Ch | ok  | 10.1 | 118 Volts
Voltage 2        | 6Dh | ok  | 10.2 | 118 Volts
Pwr Consumption  | 77h | ok  |  7.1 | 224 Watts
```


# Set Automatic Fan Speed
Revert to Automatic Fan Control
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> raw 0x30 0x30 0x01 0x01
```
- min fan speed is 35%


# 3rd party PCIe Response
If a 3rd party PCIe device such as a graphics card or NIC is installed to the server, this will alter the default fan speed. This feature can be disabled.

Disable 3rd Party PCIe Response
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> raw 0x30 0xce 0x00 0x16 0x05 0x00 0x00 0x00 0x05 0x00 0x01 0x00 0x00
```

Enable 3rd Party PCIe Response
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> raw 0x30 0xce 0x00 0x16 0x05 0x00 0x00 0x00 0x05 0x00 0x00 0x00 0x00
```

View 3rd Party PCIe Response State
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> raw 0x30 0xce 0x01 0x16 0x05 0x00 0x00 0x00
```
- Result1= ... 00 00 00 (Enabled)
- Result2= ... 01 00 00 (Disabled)


## Report Power Supply Output
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> sdr type ‘Power Supply’
```
output
```
PS Redundancy    | 74h | ns  |  7.1 | No Reading
Status           | 62h | ok  | 10.1 | Presence detected
Status           | 63h | ok  | 10.2 | Presence detected
```

## Displays Energy Consumption
```PowerShell
.\ipmitool.exe -I lanplus -H <iDracs ip> -U <username> -P <password> delloem powermonitor
```
output
```
Power Tracking Statistics
Statistic      : Cumulative Energy Consumption
Start Time     : Fri Nov  8 19:11:59 2024
Finish Time    : Fri Nov  8 22:25:30 2024
Reading        : 0.8 kWh

Statistic      : System Peak Power
Start Time     : Fri Nov  8 19:11:59 2024
Peak Time      : Fri Nov  8 19:31:38 2024
Peak Reading   : 452 W

Statistic      : System Peak Amperage
Start Time     : Fri Nov  8 19:11:59 2024
Peak Time      : Fri Nov  8 19:31:37 2024
Peak Reading   : 4.3 A
```
