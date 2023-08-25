.. _mower-hardware: 

========================
Hardware Recommendations
========================

This page lists proven hardware options and combinations for use on Rover mowers.


Autopilots
==========

Orange Cube or Matek H743 is widely used on mowers.

Many of the mowers out there use the Orange Cubes and they have all the processing power and memory that the mower requires. The Matek H743 series have VERY similar processor and memory specs as the Cube Orange and Orange+. What you give up with the Matek boards is heated/vibration damped IMUs (not super important features for the mower application) and the convenient form factor of the Cubes. You actually gain a UART or two. A good compromise might be the ZealotH743, which is packaged into a very nice enclosure at a lower price point than the Cubes.
Some users have used other autopilots for the mower application as well, but the Orange Cube and the Matkek H743 allow advanced control abilities scripts as well as room to store 700 waypoints for complex mowing plans.  Other autopilots may not have the same capabilities.

GPS Hardware
============

2 SimpleRTK2B GPS boards or other  ZED-F9P GPS boards such as Sparkfun SMA- F9Ps are needed for the mower.  One board is the GPS for the mower determining the mower's position and the other board takes position readings with respect to known locations to calculate the direction the mower is pointed (aka Moving Base or GPS for Yaw).  It provides a heading reference that is FAR more reliable and accurate than a magnetometer (compass).  The Ardusimple 3B Heading boards will also work fine. It’s just hard to justify the cost over two Zed-F9P based modules.
DO NOT BUY THE SIMPLERTK2B+HEADING KIT FROM ARDUSIMPLE FOR USE WITH ARDUPILOT!
It can be made to work, but it will take extra effort to install it and keep it configured properly dur software updates on Ardupilot.  The Ardupilot automatic configuration process works with the other ZED-F9P GPS boards.
It is recommended that survey grade dual band GPS antennas are used on both GPS boards.  They have a signal gain of about 5db compared to 2db for the small “Black Puck” GNSS Dual Band antenna that comes with the RTK starter kits. The small low gain antennas will work and a lot of people use them, but the higher gain antennas will help when cloud cover or other factors reduce the signal.  



Radio Control
=============

Selecting a radio transmitter is difficult because a large number of them will work just fine and everyone has personal preferences and a budget they want to stay within.  It is recommended that you get a radio and receiver that has at least 10 channels.
Here are a few people are using: RadioMaster TX16S, FLYSKY FS-I6X with a 10 Channel receiver

Telemetry
=========

mRo SiK Telemetry Radio
Holybro Sik telemetry radio
RFD900 telemetry radio
Along with many others

The telemetry link from your Windows base computer to the mower is essential.  All communication from your base station computer to the mower Autopilot goes through this link.  It is not recommended to use WIFI instead of a radio for telemetry on a mower application.  All of these radios operate in the Frequency Range:  902 - 928 MHz (USA) / 915 - 928 MHz (Australia).  Both the MRo and the Holybro radios transmit 100mw and the RFD 900 radio transmits 1w and is a little more of a long-range option.



Servos
======

The drive servos are selected based on the amount of force that is needed to move the hydraulic control valve on the mower platform that is selected.  The servo mentioned here is for the very common HydroGear transaxle used on many mowers.  If you have a mower platform that is different you will have to determine what servo will work by researching what others have used or measuring the torque required.  There is a resistance spring on each of the hydraulic control valves that needs to be removed to reduce the torque required by the servo.  The resistance springs are only needed when driving with the manual drive handles.  The manual drive handles are disconnected when the servos are connected.  Manual drive handle operation is not compatible with the electronic drive system.  The new manual operation is now to flip a switch on the RC controller and drive with the joysticks.
 It is a multi-dimensional puzzle to figure out where to mount the drive servos. They need to be in a serviceable location, protected from the other moving parts and heat sources, be very firmly mounted, as well as being in the right place to get the movement needed.  This will likely require building custom mounting hardware and linkages. The AGFRC 100KG servo is a bigger more robust solution, having more torque and operating at a higher voltage.  The HiTec D845WP is a 45KG servo operating at a lower voltage (5-8v), also requiring a separate lower voltage power supply.  Both of these servos are Waterproof - Metal Gear servos for use in harsh environments.






Typical Electronic Interconnect Diagram
========================================

.. image:: ../images/mower_diagram.png
    :target: ../_images/mower_diagram.png





Typical Mower Electroinics enclosure
=====================================

.. image:: ../images/mover-Webre_electronics.png
    :target: ../_images/mover-Webre_electronics.png





Harware for other modifications to the mower platform
=================================================

Other hardware is needed to automate the engine throttle, mower deck blade control, and carburetor choke.  Every builder usually implements some safety shutdown switches to stop operation quickly.  A good approach is to put enough safe guards in place so you, “the builder”, feel comfortable doing the task at hand.  Automated equipment can be dangerous if you are not in control even when it is running on its own. Most people have an emergency ignition switch on the mower and a remote-controlled ignition switch on the gas engine mowers.  Most of these added on controls use the RC transmitter to control PWM relays mounted on the mower.  Some builders use completely separate radios for the safety shutdown system   The PWM relays control the additional smaller servos or the switching to turn systems on or off. This part of the build gets very customized and every interface to these mower platforms is different.  Some common parts often used are as follows:

Picture of PWM Relay Board  mower-CZH_Labs SPDT_8channel_PWM_relay_Model_D-228.png


Picture of Servo Cam-over on throttle linkage  mower- servo_cam-over_on_throttle.png


Additional Hardware??
=====================


Information to eventually be moved to mower-gps> and <mower-tuning> pages

RTK GPS Configuration

All First Time Setup and First Drive and Tuning steps in Rover need to be followed before any attempt is made to install Peripheral Hardware like the GPS systems

GPS/Compass (landing page)
https://ardupilot.org/rover/docs/common-positioning-landing-page.html

RTK GPS Correction (Fixed Baseline)
https://ardupilot.org/rover/docs/common-rtk-correction.html#common-rtk-correction

GPS for Yaw (aka Moving Baseline)
https://ardupilot.org/rover/docs/common-gps-for-yaw.html

Steps for configuring (assuming Fixed Baseline and GPS-For-Yaw)
1.	Follow the Rover setup information above
2.	Determine where the RTCM3 correction data is coming from (base station or outside source such as NTRIP)
3.	Determine whether the correction data is going straight to the mower via separate radio or coming from your base station through Mavlink using your telemetry radio
4.	Get the mower operating on GPS1 receiving  RTCM3 correction data.  Look for the  Mission Planner data screen to report GPS rtk Fixed
5.	Follow procedures and install and set up the other GPS unit for GPS-For-Yaw.  With both GPS units installed and receiving RTCM signals the  Mission Planner data screen to report GPS1 rtk Fixed and GPS2 GPS1 rtk Fixed
 
Picture of Mission Planner data screen

Configuration tips:
Make sure you have the correct configuration parameters installed.  Check and recheck them.  Getting the configuration parameters for each GPS unit that define position of the GPS antennas in the body frame are critical.  Remember there has to be a minimum of 30cm (1 meter is better) between the antennas and the farther apart they are, the better.   You will get a more accurate heading reference when they are spaced further apart.
GPS_POS1_X/Y/Z
GPS_POS2_X/Y/Z 
 In general, all the configuration parameters are also very critical and cause problems.  The center of rotation should be selected as the axis on the mower you want it to rotate around.  However, users seem to have an easier time tuning the mower to turn properly if it is defined to be centered between the rear tires on a zero-turn mower.

How get RTCM correction data
========================
A RTK base station will be required to operate an RTK mower unless you can connect to an NTRIP RTK correction service (most users end up building a base station),  If you build a base station you will need an additional ZED-F9P GPS board for the base station.  Often people use a Raspberry Pi and solar panel+SLA battery to power/connect it. The antenna needs to be mounted in a fixed location. A lot of users mount them on top of a building.  A fixed reference is key to getting repetitive mowing jobs.  In those setups the GPS F9P module is set to “Survey-In”  location of that antenna. Self survey-in takes a long time (over night) to reach high accuracy (1-2 cm).  U-Center software is used to configure the F9P module in order to change operating modes.  Once the GPS location of the antenna is determined, you change the operating mode of the F9P module to “Fixed Mode” and enter in the known accurate GPS location of that antenna.  All of your mowing will be done in reference to that location.  The mowing patterns would move around if the accuracy of the reference moved.
Mobile base stations on tri-pods can be used but require usage locations to be surveyed in and coordinates to be entered each time they are setup.  This can lead to problems. If you move around mowing various properties where it is inconvenient to set up a fixed base, a user might consider an NTRIP subscription.  This would also require internet access.
RTK base stations are usually built in one of two ways:
1.	To sent the RTCM correction signal to your ground station computer and inject using Mission Planner via Mavilink
2.	To send the RTCM correction center straight to the mower via a separate dedicated radio
There advantages and disadvantages to each option, but most users elect to send the correction data trough the ground station computer and have it sent over the main telemetry radio.  In this option it does require a companion computer like a raspberry pi to be attached to the F9P GPS board in the RTK base station.
In the second option where the RTCM correction goes straight to the mower no companion computer is needed but a small radio + antenna is plugged into the Arduino header on the F9P board.  In this case normally the Ardusimple RTK2B board is used with the Ardusimple LR radio.  However, other radios could easily be attached to the GPS board in a slightly different way.  In this option the mower gets the correction signal and can keep working, even if the ground station computer is turned off and control is switched to another computer out in the field.

Usually NTRIP subscriptions are more costly than the cost of another F9P module and antenna to create your own fixed base.
NTRIP sites that can be accessed freely via RTK2Go 6. Many of them are privately operated, sometimes unreliable, and sometimes incompatible with our use case(s), but they are free.




MowerTuning Tips
