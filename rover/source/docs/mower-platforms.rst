.. _mower-platforms: 

===============
Mower Platforms
===============

Steps required to build a RTK mower
===============

These steps necessitate considerable labor and typically require about a year to complete, depending on the builder's availability and skill set.

- Determine what type of build you are doing, a conversion, a build from scratch, gas or electric?
- Learn about the process of build a Rover in the Ardupilot Wiki steps https://ardupilot.org/rover/docs/apmrover-setup.html.  Having sufficient knowledge is essential for making informed build decisions and procurements.
- Learn what others have done and start determining what equipment you will need.  For instance here is where you would likely determine what servos you need if you are converting a hydro driven mower.
- Decide how you are getting RTK GPS correction data
- Decide how you are doing telemetry (radio or Wi-Fi)
- Start ordering electronics equipment
- Start making modifications to the mower platform or building your platform like installing the servos, if you are building a hydrostat controlled drive.  At this point a servo test box is useful for basic servo movement.
- Install your electronics enclosure, and make provisions for mounting antennas and other special equipment.
- Breadboard up your electronics up on the benchtop (without the GPS units) to install the firmware in the flight controller, start setting up some of the basic parameters to be able to manipulate servos or other equipment manually with the RC remote.  Some of the failsafes will need to be turned off to allow this.
- Install the electronics into your waterproof electronics enclosure and make sure what ever telemetry system you have selected is working.  You will need to electronically interface with the mower a lot.
- Adjust the installed servos on the mower with no engine running to achieve proper travel and centering using linkages and trim parameters in Mission Planner
- Take all necessary safety precautions and adjust the servos with the rear mower wheels off  the ground to achieve straight tracking with the engine running.  Each wheel should be going exactly the same RPM at low speed.
- Take the mower out for a test drive using the RC transmitter with the manual drive mode selected. 
- Install other control relays and equipment for engine shutdown, engine starting, blade control, any safety lights, or other user specific equipment
- Install the GPS units and get them working (make sure the Satellite antennas are in view of the open sky). 
- Complete all the first drive and tuning steps https://ardupilot.org/rover/docs/rover-first-drive.html
