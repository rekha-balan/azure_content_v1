<properties 
   pageTitle="Turn your StorSimple device on or off | Microsoft Azure"
   description="Explains how to turn on a new StorSimple device, turn on a device that was shut down or lost power, and turn off a running device."
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carolz"
   editor="" />

<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="12/02/2015"
   ms.author="alkohli" />

# Turn your StorSimple device on or off
## Overview
Shutting down a Microsoft Azure StorSimple device is not required as a part of normal system operation. However, you may need to turn on a new device or a  device that had to be shut down. Generally, a shutdown is required in cases in which you must replace failed hardware, physically move a unit, or take a device out of service. This tutorial describes the required procedure for turning on and shutting down your StorSimple device in different scenarios.

The following table lists various scenarios for turning on and shutting down your StorSimple device and provides links to the appropriate procedures.

| Scenario | Reference topics |
|:--- |:--- |
| Turn on a new device |[Turn on a new device](#turn-on-a-new-device.md)<ul><li>[New device with primary enclosure only](#new-device-with-primary-enclosure-only.md)</li><li>[New device with EBOD enclosure](#new-device-with-ebod-enclosure.md)</li></ul> |
| Turn on a device after shutdown |[Turn on a device after shutdown](#turn-on-a-device-after-shutdown.md)<ul><li>[Device with primary enclosure only](#device-with-primary-enclosure-only.md)</li><li>[Device with EBOD enclosure](#device-with-ebod-enclosure.md)</li></ul> |
| Turn on a device after a power loss |[Turn on a device after a power loss](#turn-on-a-device-after-a-power-loss.md)<ul><li>[Device with primary enclosure only](#8100.md)</li><li>[Device with EBOD enclosure](#8600.md)</li></ul> |
| Turn on a device after the primary enclosure and EBOD connection is lost |[Turn on a device after the primary and EBOD enclosure connection is lost](#turn-on-a-device-after-the-primary-and-EBOD-enclosure-connection-is-lost.md) |
| Shut down a running device |[Turn off a running device](#turn-off-a-running-device.md)<ul><li>[Device with primary enclosure only](#8100a.md)</li><li>[Device with EBOD enclosure](#8600a.md)</li></ul> |

## Turn on a new device
The steps for turning on a Microsoft Azure StorSimple device for the first time differ depending on whether the device is an 8100 or an 8600 model. The 8100 has a single primary enclosure, whereas the 8600 is a dual-enclosure device with a primary enclosure and an EBOD enclosure. The detailed steps for both models are covered in the following sections.

* [New device with primary enclosure only](#new-device-with-primary-enclosure-only.md)

* [New device with EBOD enclosure](#new-device-with-ebod-enclosure.md)


### New device with primary enclosure only
The StorSimple 8100 model is a single enclosure device. Your device includes redundant Power and Cooling Modules (PCMs). Both PCMs must be installed and connected to different power sources to ensure high availability.

Perform the following steps to cable your device for power.

<!--author=alkohli last changed: 9/16/15-->

#### To cable for power

1. Make sure that the power switches on each of the Power and Cooling Modules (PCMs) are in the OFF position.

2. Connect the power cords to each of the PCMs in the primary enclosure.

3. Attach the power cords to the rack power distribution units (PDUs) as shown in the following image. Make sure that the two PCMs use separate power sources.

    >[AZURE.IMPORTANT] To ensure high availability for your system, we recommend that you strictly adhere to the power cabling scheme shown in the following diagram. 

    ![Cable your 2U device for power](./media/storsimple-cable-8100-for-power/HCSCableYour2UDeviceforPower.png)

    **Power cabling on an 8100 device**

    |Label|Description|
    |:----|:----------|
    |1|PCM 0|
    |2|Controller 1|
    |3|Controller 0|
    |4|PCM 1|
    |5|PDUs|

4. To turn on the system, flip the power switches on both PCMs to the ON position.


> [!NOTE]
> For complete device setup and cabling instructions, go to [Install your StorSimple 8100 device](storsimple-8100-hardware-installation.md). Make sure that you follow the instructions exactly.
> 
> 
### New device with EBOD enclosure
The StorSimple 8600 model has both a primary enclosure and an EBOD enclosure. This requires the units to be cabled together for Serial Attached SCSI (SAS) connectivity and power.

When setting up this device for the first time, perform the steps for SAS cabling first and then complete the steps for power cabling.

<!--author=alkohli last changed: 9/23/15-->

#### To attach the SAS cables

1. Identify the primary and the EBOD enclosures. The two enclosures can be identified by looking at their respective back planes. See the following image for guidance. 

    ![Back plane of primary and EBOD enclosures](./media/storsimple-sas-cable-8600/HCSBackplaneofprimaryandEBODenclosure.png)

    **Back view of primary and EBOD enclosures**

    |Label|Description|
    |:----|:----------|
    |1|Primary enclosure|
    |2|EBOD enclosure|

2. Locate the serial numbers on the primary and the EBOD enclosures. The serial number sticker is affixed to the back ear of each enclosure. The serial numbers must be identical on both enclosures. [Contact Microsoft Support](storsimple-contact-microsoft-support.md) immediately if the serial numbers do not match. See the following illustration to locate the serial numbers.

    ![Rear view of enclosure showing serial number](./media/storsimple-sas-cable-8600/HCSRearviewofenclosureindicatinglocationofserialnumbersticker.png)

    **Location of serial number sticker**

    |Label|Description|
    |:----|:----------|
    |1|Ear of the enclosure|

3. Use the provided SAS cables to connect the EBOD enclosure to the primary enclosure as follows:

    1. Identify the four SAS ports on the primary enclosure and the EBOD enclosure. The SAS ports are labeled as EBOD on the primary enclosure and correspond to port A on the EBOD enclosure, as shown in the SAS cabling illustration, below.

    2. Use the provided SAS cables to connect the EBOD port to port A.

    3. The EBOD port on controller 0 should be connected to the port A on EBOD controller 0. The EBOD port on controller 1 should be connected to the port A on EBOD controller 1. See the following illustration for guidance. 
																	
     ![SAS cabling for your device](./media/storsimple-sas-cable-8600/HCSSAScablingforyourdevice.png)

     **SAS cabling**

    |Label|Description|
    |:----|:----------|
    |A|Primary enclosure|
    |B|EBOD enclosure|
    |1|Controller 0|
    |2|Controller 1|
    |3|EBOD Controller 0|
    |4|EBOD Controller 1|
    |5, 6|SAS ports on primary enclosure (labeled EBOD)|
    |7, 8|SAS ports on EBOD enclosure (Port A)|


<!--author=alkohli last changed: 9/16/15-->


#### To cable your device for power

>[AZURE.NOTE] Both enclosures on your StorSimple device include redundant PCMs. For each enclosure, the PCMs must be installed and connected to different power sources to ensure high availability.

1. Make sure that the power switches on all the PCMs are in the OFF position.

2. On the primary enclosure, connect the power cords to both PCMs. The power cords are identified in red in the power cabling diagram, below.

3. Make sure that the two PCMs on the primary enclosure use separate power sources.

4. Attach the power cords to the power on the rack distribution units as shown in the power cabling diagram.

5. Repeat steps 2 through 4 for the EBOD enclosure.

6. Turn on the EBOD enclosure by flipping the power switch on each PCM to the ON position.

7. Verify that the EBOD enclosure is turned on by checking that the green LEDs on the back of the EBOD controller are turned ON.

8. Turn on the primary enclosure by flipping each PCM switch to the ON position.

9. Verify that the system is on by ensuring the device controller LEDs have turned ON.

10. Make sure that the connection between the EBOD controller and the device controller is active by verifying that the four LEDs next to the SAS port on the EBOD controller are green.

    >[AZURE.IMPORTANT] To ensure high availability for your system, we recommend that you strictly adhere to the power cabling scheme shown in the following diagram.

    ![Cable your 4U device for power](./media/storsimple-cable-8600-for-power/HCSCableYour4UDeviceforPower.png)

    **Power cabling**

    |Label|Description|
    |:----|:----------|
    |1|Primary enclosure|
    |2|PCM 0|
    |3|PCM 1|
    |4|Controller 0|
    |5|Controller 1|
    |6|EBOD controller 0|
    |7|EBOD controller 1|
    |8|EBOD enclosure|
    |9|PDUs|


> [!NOTE]
> For complete device setup and cabling instructions, go to [Install your StorSimple 8600 device](storsimple-8600-hardware-installation.md). Make sure that you follow the instructions exactly.
> 
> 
## Turn on a device after shutdown
The steps for turning on a Microsoft Azure StorSimple device after it has been shut down are different depending on whether the device is an 8100 or an 8600 model. The 8100 has a single primary enclosure, whereas the 8600 is a dual-enclosure device with a primary enclosure and an EBOD enclosure.

* [Device with primary enclosure only](#device-with-primary-enclosure-only.md)

* [Device with EBOD enclosure](#device-with-ebod-enclosure.md)


### Device with primary enclosure only
After a shutdown, use the following procedure to turn on a StorSimple device with a primary enclosure and no EBOD enclosure.

#### To turn on a device with a primary enclosure only
1. Make sure that the power switches on both Power and Cooling Modules (PCMs) are in the OFF position. If the switches are not in the OFF position, then flip them to the OFF position and wait for the lights to go off.

2. Turn on the device by flipping the power switches on both PCMs to the ON position. The device should turn on.

3. Check the following to verify that the device is fully on:

   1. The OK LEDs on both PCM modules are green.

2. The status LEDs on both controllers are solid green.

3. The blue LED on one of the controllers is blinking, which indicates that the controller is active.

   If any of these conditions are not met, then your device is not healthy. Please [contact Microsoft Support](storsimple-contact-microsoft-support.md).



### Device with EBOD enclosure
After a shutdown, use the following procedure to turn on a StorSimple device with a primary enclosure and an EBOD enclosure. Perform each step in sequence exactly as described. Failure to do so could result in data loss.

#### To turn on a device with a primary and an EBOD enclosure
1. Make sure that the EBOD enclosure is connected to the primary enclosure. For more information, see [Install your StorSimple 8600 device](storsimple-8600-hardware-installation.md).

2. Make sure that the Power and Cooling Modules (PCMs) on both the EBOD and primary enclosures are in the OFF position. If the switches are not in the OFF position, then flip them to the OFF position and wait for the lights to go off.

3. Turn on the EBOD enclosure first by flipping the power switches on both PCMs to the ON position. The PCM LEDs should be green. A green EBOD controller LED on this unit indicates that the EBOD enclosure is on.

4. Turn on the primary enclosure by flipping the power switches on both PCMs to the ON position. The entire system should now be on.

5. Verify that the SAS LEDs are green, which ensures that the connection between the EBOD enclosure and the primary enclosure is good.


## Turn on a device after a power loss
A power outage or interruption can shut down a Microsoft Azure StorSimple device. The power outage can happen on one of the power supplies or both power supplies. The recovery steps are different depending on whether the device is an 8100 or an 8600 model. The 8100 has a single primary enclosure, whereas the 8600 is a dual-enclosure device with a primary enclosure and an EBOD enclosure. This section describes the recovery procedure for each scenario.

* [Device with primary enclosure only](#8100.md)

* [Device with EBOD enclosure](#8600.md)


### Device with primary enclosure only <a name="8100">
The system can continue its normal operation if there is power loss to one of its power supplies. However, to ensure high availability of the device, restore power to the power supply as soon as possible.

If there is a power outage or power interruption on both power supplies, the system will shut down in an orderly and controlled manner. When the power is restored, the system will automatically turn on.

### Device with EBOD enclosure <a name="8600">
#### Power loss on one power supply
The system can continue its normal operation if there is power loss to one of its power supplies on the primary enclosure or the EBOD enclosure. However, to ensure high availability of the device, please restore power to the power supply as soon as possible.

#### Power loss on both power supplies on primary and EBOD enclosures
If there is a power outage or power interruption on both power supplies, the EBOD enclosure will shut down immediately and the primary enclosure will shut down in an orderly and controlled manner. When power is restored, the appliance will start automatically.

If the power is switched off manually, then take the following steps to restore power to the system.

1. Turn on the EBOD enclosure.

2. After the EBOD enclosure is on, turn on the primary enclosure.


### Power loss on both power supplies on EBOD enclosure
When you set up your cables, you must ensure that the EBOD is never connected alone to a separate PDU. If the EBOD and primary enclosure fail at the same time, the system will recover.

If only the EBOD enclosure fails on both power supplies, the system will not automatically recover. Take the following steps to turn on the system and restore it to a healthy state:

1. If the primary enclosure is turned on, switch off both Power and Cooling Modules (PCMs).

2. Wait for a few minutes for the system to shut down.

3. Turn on the EBOD enclosure.

4. After the EBOD enclosure is on, turn on the primary enclosure.


## Turn on a device after the primary and EBOD enclosure connection is lost
If the connection is lost between the standby controller and the corresponding EBOD controller, the device continues to work. If the connection between the system active controller and the corresponding EBOD controller is lost, failover should occur and the device should continue to work as normal.

When both Serial Attached SCSI (SAS) cables are removed or the connection between the EBOD enclosure and the primary enclosure is severed, the device will stop working. At this point, perform the following steps.

### To turn on the device after connection is lost
1. Access the back of the device.

2. If the SAS cable connection between the EBOD enclosure and the primary enclosure is broken, all SAS lane LEDs on the EBOD enclosure will be off.

3. Shut down both Power and Cooling Modules (PCMs) on the EBOD enclosure and the primary enclosure.

4. Wait until all the lights on the back of both the enclosures turn off.

5. Reinsert the SAS cables, and ensure that there is a good connection between the EBOD enclosure and the primary enclosure.

6. Turn on the EBOD enclosure first by flipping both PCM switches to the ON position.

7. Ensure that the EBOD enclosure is on by checking that the green LED is ON.

8. Turn on the primary enclosure.

9. Ensure that the primary enclosure is on by checking that the controller green LED is ON.

10. Verify that the EBOD enclosure connection with the primary enclosure is good by checking that the SAS lane LEDs (four per EBOD controller) are all ON.


> [!IMPORTANT]
> If the SAS cables are defective or the connection between the EBOD enclosure and the primary enclosure is not good, when you turn on the system, it will go into recovery mode. Please [contact Microsoft Support](storsimple-contact-microsoft-support.md) if this happens.
> 
> 
## Turn off a running device
A running Microsoft Azure StorSimple device may need to be shut down if it is being moved, taken out of service, or has a malfunctioning component that needs to be replaced. The steps are different depending on whether the Microsoft Azure StorSimple device is an 8100 or an 8600 model. The 8100 has a single primary enclosure, whereas the 8600 is a dual-enclosure device with a primary enclosure and an EBOD enclosure. This section details the steps to shut down a running device.

* [Device with primary enclosure](#8100a.md)

* [Device with EBOD enclosure](#8600a.md)


### Device with primary enclosure <a name="8100a">
Currently there is no way to shut down a running StorSimple device from the Azure classic portal. The only way to shut it down is by using Windows PowerShell for StorSimple. To shut down the device in an orderly and controlled manner, access Windows PowerShell for StorSimple and follow the steps below.

> [!IMPORTANT]
> Do not shut down a running device by using the power button on the back of the device.
> 
> Before shutting down the device, make sure that all the device components are healthy. In the Azure classic portal, navigate to **Devices** > **Maintenance** > **Hardware Status**, and verify that status of all the components is green. This is true only for a healthy system. If the system is being shut down to replace a malfunctioning component, you will see a failed (red) or degraded (yellow) status for the respective component in the **Hardware Status**.
> 
> 
You can connect to the Windows PowerShell for StorSimple through the device serial console or through Windows PowerShell remoting. After you access Windows PowerShell for StorSimple, perform the following steps to shut down a running device.

#### To shut down a running device
1. Connect to the serial console of the device.

2. In the menu that appears, verify that the controller you are connected to is the **Standby** controller. If it is not the standby controller, disconnect from the controller, and connect to the other controller.

3. In the serial console menu, choose **Option 1** to log on to the standby controller with full access.

4. At the prompt, type: 

    `Stop-HCSController`

    This should shut down the current standby controller.

   > [!IMPORTANT]
> Wait for the controller to shut down completely before you proceed to the next step.
> 
5. To verify that the shutdown has finished, check the back of the device. The controller fault LED should be solid red.

6. Connect to the active controller through the serial console and follow the same steps to shut it down.

7. After both the controllers have shut down completely, the status LEDs on both controllers should be blinking red.

8. If you need to shut down the device completely at this time, flip the power switches on both Power and Cooling Modules (PCMs) to the OFF position.

9. To verify that the device has completely shut down, check that all the lights at the back of the device are off.


### Device with EBOD enclosure <a name="8600a">
> [!IMPORTANT]
> Before shutting down the primary enclosure and the EBOD enclosure, ensure that all the device components are healthy. In the Azure classic portal, navigate to **Devices** > **Maintenance** > **Hardware Status**, and verify that all the components are healthy.
> 
> 
#### To shut down a running device with EBOD enclosure
1. Follow all the steps listed in [Device with primary enclosure only](#8100a.md).

2. After the primary enclosure is shut down, shut down the EBOD by flipping off both Power and Cooling Module (PCM) switches.

3. To verify that the EBOD has shut down, check that all lights on the back of the EBOD enclosure are off.


> [!NOTE]
> The SAS cables that are used to connect the EBOD enclosure to the primary enclosure should not be removed until after the system is shut down.
> 
> 
## Next steps
[Contact Microsoft Support](storsimple-contact-microsoft-support.md) if you encounter problems when turning on or shutting down a StorSimple device.

