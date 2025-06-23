.. _mower-gps: 

===============
Mower GPS Setup
===============

All First Time Setup and First Drive and Tuning steps in Rover need to be followed before any attempt is made to install Peripheral Hardware like the GPS systems

RTK GPS section of the GPS/Compass (landing page)
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
5.	Follow procedures to install and set up the other GPS unit for GPS-For-Yaw.  With both GPS units installed and receiving RTCM signals. The Mission Planner data screen should show GPS1 rtk Fixed and GPS2 rtk Fixed.

(image missing)
This is what you should see

Configuration tips
==================

Make sure you have the correct configuration parameters installed.  Check and recheck them.  Getting the configuration parameters for each GPS unit that define position of the GPS antennas in the body frame are critical.  Remember there has to be a minimum of 30cm (1 meter is better) between the antennas and the farther apart they are, the better.   You will get a more accurate heading reference when they are spaced further apart.
GPS_POS1_X/Y/Z
GPS_POS2_X/Y/Z 
The center of rotation should be selected as the axis on the mower you want it to rotate around.  However, users seem to have an easier time tuning the mower to turn properly if it is defined to be centered between the rear tires on a zero-turn mower.

Troubleshooting Problems
========================

This section needs work with some screenshots.  This might be the most important part of this section.

How get RTCM correction data
============================

A RTK base station will be required to operate an RTK mower unless you can connect to an NTRIP RTK correction service (most users end up building a base station,  If you build a base station you will need an additional ZED-F9P GPS board for the base station.  Often people use a Raspberry Pi and solar panel+SLA battery to power/connect it. The antenna needs to be mounted in a fixed location with a clear view of the sky. A lot of users mount them on top of a building.  A fixed reference is key to getting repetitive mowing jobs.  In a base station setup (unless you know the GPS coordinates of the antenna) the GPS F9P module is set to “Survey-In”  the location of the antenna. Self survey-in takes a long time (over night) to reach high accuracy (1-2 cm).  Download U-Center software to configure the F9P module and to change operating modes.  Once the GPS location of the antenna is determined, record the GPS position of the antenna, and change the operating mode of the F9P module to “Fixed Mode” and enter in the known accurate GPS location of that antenna.  All of your mowing will be done in reference to that location.  The mowing patterns can move around if the reference is moved and not setup correctly.
Mobile base stations on tri-pods can be used but require usage locations to be self surveyed-in each time the GPS antenna is moved and this can take a while.  This can lead to problems and mowing patters that can move around slightly.  Generally, if people move around mowing various properties where it is inconvenient to set up a base station, they would use NTRIP.  This would also require internet access.
RTK base stations are usually built in one of two ways:

1.	Send the RTCM correction signal to your ground station computer and inject using Mission Planner via Mavilink, which then sends it over to the mower using the telemetry radio.
2.	Send the RTCM correction center straight to the mower via a separate dedicated radio

There are advantages and disadvantages to each option, but most users elect to send the correction data through the ground station computer and have it sent over the main telemetry radio.  In this option it does require a companion computer like a raspberry pi to be attached to the F9P GPS board in the RTK base station.  In this case Mavlink passes the RTCM3 signal to GPS1, and then it will pass its own correction data on to GPS2 for heading calculation.

In the second option where the RTCM correction goes straight to the mower no companion computer is needed in the base station but two small Sik radios + antennas with matching ID codes are needed. One located in the base station transmits the RTCM3 signal via the radio to the moving-base F9P (GPS2) in the mower and then the RTCM3 signal is sent over to GPS1 using a wire connected to UART2 on both mower F9P boards.  The Ardusimple LR radio that plugs into the Arduino header on the GPS boards can make this very easy, but many radios can be made to work.  When configured this way the mower can keep working, even if the ground station computer is turned off and control is switched to another computer out in the field.

Usually NTRIP subscriptions are more costly than the cost of another F9P module and antenna to create your own fixed base especially if you need correction data often.
NTRIP sites that can be accessed freely via RTK2Go 6. Many of them are privately operated, sometimes unreliable, and sometimes incompatible with our use case(s), but they are free.

