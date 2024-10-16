# winc1500-Manual-UART-Firmware-Update
This repo serves as a WINC1500 manual UART firmware update procedure and slightly modified Microchip flash script.  This is to consolidate and supplement other (incomplete) documentation from Microchip.

## Modified Source Script
NOTE: The original script and firmware update files can be found [here](https://github.com/Microchip-MPLAB-Harmony/wireless_wifi/tree/master/utilities/wifi/winc)
The script in this repo has been modified to allow the user to power cycle the WINC1500 during the update procedure.  Specifically, at line 133 of winc_flash_tool.cmd added:
```
echo Power cycle WINC and set to bootloader mode.
pause 1
```
[Source](https://forum.microchip.com/s/topic/a5C3l000000MeKbEAK/t385088)

## Firmware Update Procedure
1. Connect 3.3V UART to USB converter directly to WINC1500 TX, RX, GND pins (solder directly to WINC1500 module if needed).
2. Program microcontroller to toggle enable and reset lines to put module in bootloader mode.  Put this code in main.c:
   
```
WDRV_WINC_CHIP_EN_Clear();
WDRV_WINC_RESETN_Clear();
WDRV_WINC_SS_Set();
CORETIMER_DelayMs(1000);
WDRV_WINC_CHIP_EN_Set();
CORETIMER_DelayMs(100);
WDRV_WINC_RESETN_Set();
while (1);
```

Alternatively, manually manipulate CHIP_EN pin and RESET to achieve bootloader mode.  Different sources suggest different methods:

```
1. ATWINC1500 UART interface pins Rx, TX, and GND to be connected to PC. Since ATWINC1500 UART is TTL level output, please take care about the RS232 converter to connect to the PCs COM port. 
2. RESET_N and CHIP_EN pins to be connected to GND to make it low while doing the  ATWINC1500 power up. 
3. Power up the ATWINC1500 module with VCC 3.3 V and GND. 
4. Once power up is done, CHIP_EN pin to be connected VCC 3.3V to make it high.
5. RESET_N pin to be connected to be VCC 3.3 V to make it high.
```
[Source](https://microchip.my.site.com/s/article/How-to-update-the-ATWINC1500-firmware-without-using-any-MCU-or-using-other-vendor-MCUs)

or
```
If there are any difficulties regarding the connections (the chip not being discovered when running the programming tools),
the WINC1500 module can be manually put in the right state by putting the CHIP_EN pin to VCC and the RESET pin to VCC and
keeping them connected to VCC after this while the programming is being executed.

In short, after powering up the module, the CHIP_EN must be pulled high, and then the RESETN must be pulled high.
```
![image](https://github.com/user-attachments/assets/f6691b0f-01be-4dfd-a497-59fde68a8383)

[Source](https://microchip.my.site.com/s/article/WINC1500-Firmware-Update-methods)

3.  Power cycle the WINC1500.
4.  Navigate to the \winc folder, open a terminal and execute
`.\winc_flash_tool.cmd /p COM20 /d WINC1500 /v 19.7.7 /x /e /i aio /w
5.  The script should initially connect to the WINC1500 and read the chip ID and XO offset.
6.  Eventually, the script will pause and display:
```
Power cycle WINC and set to bootloader mode.
Press any key to continue . . .
```
7.  The WINC1500 needs to be reset again for the script to properly continue.  Power cycle the WINC1500 and allow the CHIP_EN pin and RESET pins to toggle to reinitiate bootloader mode.
8.  Press a key to allow the script to continue.
9.  The script should proceed to erase, flash, and verify firmware.
