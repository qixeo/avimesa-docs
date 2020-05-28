---
title: Avimesa 1000
date: 2020-05-28
slug: avimesa-1000

---
![avimesa-1000][1]

## Introduction

This project contains a user guide for the Avimesa 1000. Designed to connect with industrial 4-20 mA sensors, the Avimesa 1000 is an intelligent I/O device which provides connectivity to Avimesa Messages. The system has a highly power-efficient and capable Cortex M4F processor that manages communication, measurement scheduling, sensor interface power control, and filtering of sensor data. The firmware is power-optimized, allowing for the use of battery power where needed.

<a id="toc"></a>

## Table of Contents

*   [1\. Overview][2]
*   [2\. Quick Start][3]
*   [3\. Power Supply / Sensor Power][4] 
    *   [3\.1 Ratings][5]
    *   [3\.2 Installation][6]
    *   [3\.3 Sensor Power Information][7]
*   [4\. Antenna Information][8] 
    *   [4\.1 Certified Antennas][9]
    *   [4\.2 Installation][10]
*   [5\. Device Modes][11] 
    *   [5\.1 LED Test Mode][12]
    *   [5\.2 Run Mode][13]
    *   [5\.4 Service Mode][14]
    *   [5\.5 Firmware Update Mode][15]
*   [6\. LED Status Codes][16] 
    *   [6\.1 LED Test Mode][17]
    *   [6\.2 Run Mode][18]
    *   [6\.4 Service Mode][19]
    *   [6\.5 Firmware Update Mode][20]
*   [7\. Sensor Connection][21] 
    *   [7\.1 Pinout][22]
    *   [7\.2 Shielded Cables][23]
*   [8\. Device Configuration][24] 
    *   [8\.1 Measurement Schedule / Sleep Time][25]
    *   [8\.2 Channel Settings][26] 
        *   [8\.2.1 Channel Enabled][27]
        *   [8\.2.2 Channel Schedule][28]
        *   [8\.2.3 4-20 mA Channel Settling Time][29]
        *   [8\.2.4 GPIO Channel Settings][30]

<a id="1.-overview"></a>

## 1\. Overview

In general, the Avimesa 1000 features:

1.  Seven (7) analog channels used to (primarily) interface with 4-20 mA sensors
2.  Up to nine (9) GPIO channels (0-3.3VDC), configurable as input/output/latching/etc. exposed on a 40-pin header 
3.  Power supply / sensor power monitoring
4.  Avimesa Messages Synchronization capability
5.  Device configuration for measurement schedule and sensor options

Figure 1 calls out the main components that are used in this guide:

    A. Power Switch  
    B. NFC Header (for future use)  
    C. Antenna Connector (type U.FL)  
    D. Programming LED  
    E. LEDs 1-3 (left to right)  
    F. Reset Button  
    G. Power Connector  
    H. Analog Channels 1-7 (left to right)  
    I. Avimesa 40-pin Header (GPIO channels 9-17)  
    J. Shield Solder Point

![ug-callouts][31]

*Figure 1*

[Top][32]

<a id="2.-quick-start"></a>

## 2\. Quick Start

The following assumes that the Avimesa Gateway is configured and running. Please see [here][33] for help on setting up the Avimesa Gateway.

1.  Attach the antenna to the `Antenna Connector` (Figure 1-C)
2.  Attach power to the `Power Connector` (Figure 1-G)
3.  Attach a 4-20 mA sensors to `Analog Channels 1-7` (Figure 1-H)
4.  Slide the `Power Switch` to the 'ON' position (Figure 1-A)

At this point, the Avimesa 1000 will proceed to take measurements and communicate with Avimesa Messages. The Avimesa Toolkit can be used to view device data and setup the Device Driver Script and device configuration.

If more details are required, please refer to the sections that follow.

[Top][32]

<a id="3.-power-supply-sensor-power"></a>

## 3\. Power Supply / Sensor Power

<a id="3.1-power-supply-sensor-power-rating"></a>

### 3\.1 Rating

Table 1 lists the power supply ratings. A battery can be used or a DC power supply.

| Condition   | Voltage   |
| ----------- | --------- |
| Recommended | 12-24 VDC |
| Maximum     | 27\.5 VDC |
| Minimum     | 10\.5 VDC |

*Table 1*

**It should be noted that the sensors are powered directly from the input voltage (minus a diode drop ~0.7 VDC)**.

This feature provides flexibility in 4-2 mA sensor selection.

<a id="3.2-power-supply-sensor-power-installation"></a>

### 3\.2 Installation

Attach the power supply connector to the `Power Connector` (Figure 1-G).

An example of the connector required and included in the kit is **PN: TODO**.

<a id="3.3-power-supply-sensor-power-info"></a>

### 3\.3 Sensor Power Information

The sensors, when powered programmatically, will be supplied by the same voltage as the input voltage, minus a diode drop of roughly ~0.7 VDC. The digital section of the board is supplied via a 3.3 VDC source for reference.

[Top][32]

<a id="4.-antenna-information"></a>

## 4\. Antenna Information

<a id="4.1-antenna-information-certified"></a>

### 4\.1 Certified Antennas

The Avimesa 1000 is certified with five (5) antennas in the following regions using the Rigado BGM-301 module:

*   USA (FCC certification)
*   Canada (IC certification)
*   Europe (CE report)
*   Japan (MIC certification)
*   Australia & New Zealand (RCM report)

The following antennas are all approved for use with the Avimesa 1000:

*   Pulse W1030 2 dBi 1/4 Wave Dipole – Whip Length: 108.3mm
*   Taoglas FXP73.07.0100A 2.5dBi 1/4 Wave Dipole – Flex 7mm x 47mm x 0.1mm
*   Pulse W1027 3.2 dBi 1/4 Wave Dipole – Whip Length: 136.8mm
*   Kinsun 6670113050-145 2.0 dBi 1/4 Wave Dipole – PCB 12mm x 65mm x 0.46mm
*   Kinsun 6610103081 5.0 dBi 1/2 Wave Dipole – Whip Length: 196.6mm

<a id="4.1-antenna-information-installation"></a>

### 4\.1 Installation

Simply click the antenna to the `Antenna Connector` (Figure 1-C)

[Top][32]

<a id="5.-device-modes"></a>

## 5\. Device Modes

In general, the device runs through a sequence of acquiring data from the attached 4-20 mA sensors, synchronizing with Avimesa Messages, and sleeping. This is called `Run Mode`.

A `Service Mode` is used for factory setup and is not typically used by end users.

A `Firmware Update Mode` is used when updating the firmware.

<a id="5.1-device-modes-led-test"></a>

### 5\.1 LED Test Mode

Upon boot up or soft reset, the device will run through an `LED Test Mode`. This mode is provided to ensure the LEDs are operational, and to provide the user a window of time to put the device into service mode if needed.

<a id="5.2-device-modes-run-mode"></a>

### 5\.2 Run Mode

After boot up, the normal state of the device is `Run Mode`. In the mode, the device will perform measurements and synchronize with Avimesa Messages based upon it's configuration. The configuration is setup via the Avimesa Toolkit and is described [here][24].

<a id="5.3-device-modes-service-mode"></a>

### 5\.3 Service Mode

When the device isn't configured it will boot up into `Service Mode`. The device can also be manually forced into 'service mode' through the following sequence.

1.  Soft reset the device by momentarily pressing the `Reset` button (Figure 1-F)
2.  Immediately after the device resets (before the device transitions to `Run Mode`), press the `Reset` button (Figure 1-F)

<a id="5.4-device-modes-fw-update-mode"></a>

### 5\.4 Firmware Update Mode

When the device is in `Firmware Update Mode`, no other functionality is available. The device will reset automatically after the firmware update completes and resume normal operation.

If the firmware update is interrupted, the device will revert the currently running firmware.

[Top][32]

<a id="6.-led-status-codes"></a>

## 6\. LED Status Codes

<a id="6.1-led-status-codes-led-test"></a>

### 6\.1 LED Test Mode

In this mode, the device will cycle through each of the LEDs (1-3) and LED colors (red/blue).

![ug-callouts][34]

*Figure 2*

<a id="6.2-led-status-codes-run-mode"></a>

### 6\.2 Run Mode

When in this mode, in general the LEDs represent the following:

*   LED1 - Gateway Connection
*   LED2 - Measurement in Progress
*   LED3 - Device Cloud Connection Status
*   PROG LED - Program LED

When sleeping, the device's power switch will be on, and all LEDs will be off.

These states are represented by LEDs 1-3 using the following:

![ug-callouts][35]

*Figure 3*

The same information in text form:

| State         | LED1 Red | LED1 Blue      | LED2 Red | LED2 Blue | LED3 Red | LED3 Blue | PROG LED Green      |
| ------------- | -------- | -------------- | -------- | --------- | -------- | --------- | ------------------- |
| Sleeping      | Off      | Off            | Off      | Off       | Off      | Off       | Off                 |
| Disconnected  | Off      | **Fast Blink** | Off      | Off       | Off      | Off       | Off                 |
| Connected     | Off      | **On**         | Off      | Off       | Off      | Off       | Off                 |
| Measuring     | Off      | Off            | Off      | **On**    | Off      | Off       | Off                 |
| Synchronizing | Off      | Off            | Off      | Off       | Off      | **On**    | Off                 |
| Updating      | Off      | Off            | Off      | Off       | Off      | **On**    | **Possible Blinks** |

*Table 2*

<a id="6.3-led-status-codes-service-mode"></a>

### 6\.3 Service Mode

When in this mode, the state are represented by LEDs 1-3 using the following:

![ug-callouts][36]

*Figure 4*

The same information in text form:

| State        | LED1 Red | LED1 Blue | LED2 Red | LED2 Blue | LED3 Red | LED3 Blue | PROG LED Green      |
| ------------ | -------- | --------- | -------- | --------- | -------- | --------- | ------------------- |
| Disconnected | **On**   | Off       | **On**   | Off       | **On**   | Off       | Off                 |
| Connected    | Off      | **On**    | Off      | **On**    | Off      | **On**    | Off                 |
| Updating     | Off      | **On**    | Off      | **On**    | Off      | **On**    | **Possible Blinks** |
| Flash Error  | **On**   | **On**    | **On**   | **On**    | **On**   | **On**    | Off                 |

*Table 3*

<a id="6.4-led-status-codes-fw-update-mode"></a>

### 6\.4 Firmware Update Mode

When in this mode, the state are represented by LEDs 1-3 using the following:

![ug-callouts][37]

*Figure 5*

[Top][32]

<a id="7.-sensor-conn"></a>

## 7\. Sensor Connection

<a id="7.1-sensor-conn-pinout"></a>

### 7\.1 Pinout

`Analog Channels 1-7` (Figure 1-H) have a screw terminal connection with the following pinout:

| 1                 | 2            | 3             |
| ----------------- | ------------ | ------------- |
| Shield (optional) | Sensor Power | Signal Return |

*Table 4*

If looking at `Analog Channels 1-7` (Figure 1-H), pin 1 is on the left side.

<a id="7.2-sensor-conn-shield"></a>

### 7\.2 Shielded Cables

Although noise resistant by design, if using a shielded cable, you can solder an earth ground to the `Shield Solder Point` (Figure 1-J).

[Top][32]

<a id="8.-device-config"></a>

## 8\. Device Configuration

The device is configured using the functionality exposed by the Avimesa API. An example application, the Avimesa Toolkit (<https://toolkit.avimesa.com>), is available to be used for this and is located at <https://github.com/Avimesa/toolkit-nodejs>

The JSON object used for device configuration can be referenced [here][38]

<a id="8.1-device-config-meas-sched"></a>

### 8\.1 Measurement Schedule / Sleep Time

<a id="8.2-device-config-channel-settings"></a>

### 8\.2 Channel Settings

The Avimesa 1000 features:

1.  Seven (7) analog channels used (primarily) interface with 4-20 mA sensors. Each channel features sensor power, and is configurable for power savings
2.  Up to nine (9) GPIO channels, configurable as input/output/latching/etc.
3.  Power supply monitoring (enabled by default and un-configurable)

Please reference the documentation [here][39] for more details.

<a id="8.2.1-device-config-channel-settings-enable"></a>

### 8\.2.1 Channel Enabled

Sets whether or not a channel is enabled or not. Please reference the documentation [here][39] for more details.

<a id="8.2.2-device-config-channel-settings-sched"></a>

### 8\.2.2 Channel Schedule

For future use, set to 1 for now. Please reference the documentation [here][39] for more details.

<a id="8.2.3-device-config-channel-settings-settling-time"></a>

### 8\.2.3 4-20 mA Channel Settling Time

Specifies the time duration in seconds that the system should settle a 4-20 mA sensor before acquiring data. Please reference the documentation [here][39] for more details.

<a id="8.2.4-device-config-channel-settings-gpio"></a>

### 8\.2.4 GPIO Channel Settings

Specifies the whether a GPIO channel is an input/output, whether it is latching, etc. Please reference the documentation [here][39] for more details.

[Top][32]

 [1]: https://raw.githubusercontent.com/Avimesa/user-guide-avimesa-1000/master/images/avimesa-1000-perspective.png
 [2]: #1.-overview
 [3]: #2.-quick-start
 [4]: #3.-power-supply-sensor-power
 [5]: #3.1-power-supply-sensor-power-ratings
 [6]: #3.2-power-supply-sensor-power-installation
 [7]: #3.3-power-supply-sensor-power-info
 [8]: #4.-antenna-information
 [9]: #4.1-antenna-information-certified
 [10]: #4.2-antenna-information-installation
 [11]: #5.-device-modes
 [12]: #5.1-device-modes-led-test
 [13]: #5.2-device-modes-run-mode
 [14]: #5.3-device-modes-service-mode
 [15]: #5.4-device-modes-fw-update-mode
 [16]: #6.-led-status-codes
 [17]: #6.1-led-status-codes-led-test
 [18]: #6.2-led-status-codes-run-mode
 [19]: #6.4-led-status-codes-service-mode
 [20]: #6.5-led-status-codes-fw-update-mode
 [21]: #7.-sensor-conn
 [22]: #7.1-sensor-conn-pinout
 [23]: #7.2-sensor-conn-shield
 [24]: #8.-device-config
 [25]: #8.1-device-config-meas-sched
 [26]: #8.2-device-config-channel-settings
 [27]: #8.2.1-device-config-channel-settings-enable
 [28]: #8.2.2-device-config-channel-settings-sched
 [29]: #8.2.3-device-config-channel-settings-settling-time
 [30]: #8.2.4-device-config-channel-settings-gpio
 [31]: https://raw.githubusercontent.com/Avimesa/user-guide-avimesa-1000/master/images/ug-callouts.png
 [32]: #toc
 [33]: /avimesa-gateway-for-raspberry-pi-user-guide/
 [34]: https://raw.githubusercontent.com/Avimesa/user-guide-avimesa-1000/master/images/ug-led-states-led-test.png
 [35]: https://raw.githubusercontent.com/Avimesa/user-guide-avimesa-1000/master/images/ug-led-states-run-mode.png
 [36]: https://raw.githubusercontent.com/Avimesa/user-guide-avimesa-1000/master/images/ug-led-states-service-mode.png
 [37]: https://raw.githubusercontent.com/Avimesa/user-guide-avimesa-1000/master/images/ug-led-states-fw-update-mode.png
 [38]: https://github.com/Avimesa/device-driver-scripts#04-the-dialtone-object
 [39]: https://github.com/Avimesa/device-driver-scripts#04-the-dialtone-object-dev-chans-ch-cfg