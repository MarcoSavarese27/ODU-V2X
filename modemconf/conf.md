# Quectel modem configuration

First of all, check if the modem is correctly connected. Please run
```bash
lsusb
```
You should see something like this
```bash
...
Bus 002 Device 003: ID 2c7c:0801 Quectel Wireless Solutions Co., Ltd. RM520-GL
...
```
Now, check the AT command interface of the modem:
```bash
sudo dmesg | grep -i -e ttyUSB -e qcserial -e quectel
```
The output should contain something like this:
```bash
qcserial 2-1.3:1.0: Quectel RM520N-GL modem converter detected
usb 2-1.3: Qualcomm USB modem converter now attached to ttyUSB0
usb 2-1.3: Qualcomm USB modem converter now attached to ttyUSB1
usb 2-1.3: Qualcomm USB modem converter now attached to ttyUSB2
usb 2-1.3: Qualcomm USB modem converter now attached to ttyUSB3
```
Typically, the AT command interface is `ttyUSB2`, so run
```bash
sudo minicom -D /dev/ttyUSB2
```
and send the following AT commands
```minicom
AT+CGDCONT=1,"IP","YOUR-APN"
AT+QICSGP=1,1,"YOUR-APN","YOUR-USERNAME","YOUR-PASSWORD",2
AT+QIACT=1 
```
They mean:

1. `AT+CGDCONT=1,"IP","YOUR-APN"`: defines the PDP context (the data connection profile) for the SIM:
   * `1` is the Context ID; it's just a "slot number" for the APN profile.
   * `"IP"` is the PDP type.
   * `"YOUR-APN"` is the APN.

2. `AT+QICSGP=1,1,"YOUR-APN","YOUR-USERNAME","YOUR-PASSWORD",2`: sets the authentication and APN credentials for context 1.
    * The first `1` is the Context ID (must be the same as the last command).
    * The second `1` is the ContextType, which says that we want to use a standard cellular data connection (GSM, 3G, 4G). The modem itself determines whether it connects via 3G, 4G, or 5G based on the network availability.
    * `"YOUR-APN"` is the same as before.
    * `"YOUR-USERNAME"` and `"YOUR-PASSWORD"` are the credentials for the APN.
    * The last number defines the AuthenticationMode. `0` means no authentication required; `1` is `PAP`; `2` is `CHAP`; `3` is `PAP/CHAP` (the modem chooses the best option).

3. `AT+QIACT=1`: activates the data context. The number has to match the Context ID we set up until now.

This sequence defines the APN and credentials, and connects the modem to the network.

---
All that is left to do is to get an IP address and connect to the private network.  
If you run
```bash
ip a
```
you should see all your network interfaces. Specifically, you should see an interface with a name similar to `enx3668e7e96***`. Run the command
```bash
sudo dhclient -v enx3668e7e96***
```
(Obviously, replace `enx3668e7e96***` with your actual interface.) Now you should be able to ping the IPs in your private network.

## Troubleshoot
TODO