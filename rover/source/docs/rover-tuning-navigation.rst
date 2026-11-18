.. _rover-tuning-navigation:

=================
Tuning Navigation
=================

This page describes how to tune the Navigation control including "S-Curves", the "Position controller" and the optional "Path follower".  The lower level :ref:`speed<rover-tuning-throttle-and-speed>` and :ref:`turn rate<rover-tuning-steering-rate>` controllers should be tuned before attempting to tune this controller.

.. image:: ../images/rover-tuning-navigation1.png
    :target: ../_images/rover-tuning-navigation1.png

S-Curves and the position controller are used in all the autonomous modes including :ref:`Auto<auto-mode>`, :ref:`Guided<guided-mode>`, :ref:`RTL<rtl-mode>` and :ref:`SmartRTL<smartrtl-mode>`.

Basic Parameters
----------------

Set these before tuning the controllers below.

- :ref:`WP_SPEED<WP_SPEED>` defines the target speed (in m/s) that the vehicle will drive at in :ref:`Auto<auto-mode>` and :ref:`Guided<guided-mode>`.  It will also be used in :ref:`RTL<rtl-mode>` and :ref:`SmartRTL<smartrtl-mode>` if :ref:`RTL_SPEED<RTL_SPEED>` is set to zero
- :ref:`WP_RADIUS<WP_RADIUS>` specifies the distance before the waypoint that the vehicle may begin turning towards the next waypoint.  With the :ref:`path follower<rover-tuning-navigation-path-follower>` enabled, corners are instead shaped by :ref:`WP_PF_RADIUS<WP_PF_RADIUS>`, while :ref:`WP_RADIUS<WP_RADIUS>` still affects the planned speed through corners and sets how close the vehicle must come to a waypoint it stops at

S-Curves
--------

S-Curves are used to plan a smooth path that brings the vehicle close to each waypoint while not exceeding speed or acceleration limits.  The resulting path includes continuous position and 2D velocity targets.

Factors affecting the path include:

- The vehicle will cut the corners but will always try to pass within :ref:`WP_RADIUS<WP_RADIUS>` or :ref:`TURN_RADIUS<TURN_RADIUS>` (whichever is larger) of each waypoint.  Skid-steer vehicles ignore :ref:`TURN_RADIUS<TURN_RADIUS>`, so for them only :ref:`WP_RADIUS<WP_RADIUS>` applies
- The vehicle will cut the corners more at higher speeds
- The vehicle will cut the corners more if the maximum accelerations are reduced (e.g. the lowest of :ref:`ATC_ACCEL_MAX<ATC_ACCEL_MAX>`, :ref:`ATC_DECEL_MAX<ATC_DECEL_MAX>` and :ref:`ATC_TURN_MAX_G<ATC_TURN_MAX_G>` * 9.81)
- The vehicle will slow down in the corners if necessary to pass within :ref:`WP_RADIUS<WP_RADIUS>` of the waypoint without exceeding the maximum accelerations (e.g. the lowest of :ref:`ATC_ACCEL_MAX<ATC_ACCEL_MAX>`, :ref:`ATC_DECEL_MAX<ATC_DECEL_MAX>` and :ref:`ATC_TURN_MAX_G<ATC_TURN_MAX_G>` * 9.81)
- Placing waypoints close together may lead to the vehicle traveling slowly

.. _rover-tuning-navigation-position-controller:

Position Controller
-------------------

The position controller is responsible for calculating the desired speed and turn rate so the vehicle follows the path between waypoints created by the S-Curves (see above).  The desired speed and turn rate are then passed to the lower level controllers.

Method for tuning the position controller:

- Connect the ground station to the vehicle using a telemetry radio
- Create a rectangular or back-and-forth mission with long straight segments and upload to the vehicle

  .. image:: ../images/rover-tuning-navigation-mission.png
      :target: ../_images/rover-tuning-navigation-mission.png

- Arm the vehicle (in :ref:`Manual<manual-mode>` or :ref:`Hold<hold-mode>`) and then switch to :ref:`Auto<auto-mode>` mode
- Adjust these parameters to improve the vehicle's tracking along the lines

  - set :ref:`PSC_VEL_I<PSC_VEL_I>` and :ref:`PSC_VEL_D<PSC_VEL_D>` to zero
  - increase :ref:`PSC_VEL_P<PSC_VEL_P>` until the vehicle tracks the corners nicely without oscillating in the straights
  - increase :ref:`PSC_VEL_D<PSC_VEL_D>` enough to make the corners more responsive but without introducing oscillations in the straights.  This should normally be not more than 10% of :ref:`PSC_VEL_P<PSC_VEL_P>`
  - set :ref:`PSC_VEL_I<PSC_VEL_I>` to 20% of :ref:`PSC_VEL_P<PSC_VEL_P>`

- These values normally do not need to be changed

  - :ref:`PSC_POS_P<PSC_POS_P>` should be left at 0.2 (the default).  This converts the position error into a desired velocity.  Higher values will lead to the vehicle trying to drive back to the line more quickly but if raised too high may lead to oscillation
  - :ref:`PSC_VEL_FF<PSC_VEL_FF>` should always be 0
  - :ref:`PSC_VEL_IMAX<PSC_VEL_IMAX>` should always be 1
  - :ref:`PSC_VEL_FLTD<PSC_VEL_FLTD>`, :ref:`PSC_VEL_FLTE<PSC_VEL_FLTE>` should be left at the default of 5 although small vehicle may benefit from higher values

It may also help to monitor the velocity controller PIDs in real-time

- Ensure the mission has long straight segments aligned either North-South or West-East

  .. image:: ../images/rover-tuning-navigation-mission-northsouth.png
      :target: ../_images/rover-tuning-navigation-mission-northsouth.png

- Enable real time PID reporting of the position controller's velocity controller

  - If the mission runs West-East set GCS_PID_MASK = 64 (Velocity North)
  - If the mission runs North-South set GCS_PID_MASK = 128 (Velocity East)

- Display the real-time PID values on the ground station.  If using MP, go to the Data screen, check the "Tuning" checkbox.  Double click on the display area and select, "piddesired", "pidachieved", "pidP", "pidI" and "pidD"

  .. image:: ../images/rover-tuning-navigation-realtime.png
      :target: ../_images/rover-tuning-navigation-realtime.png

The velocity PID controller inputs and outputs are also recorded in the :ref:`onboard log's <common-downloading-and-analyzing-data-logs-in-mission-planner>` PIDN (North) and PIDE (East) messages.
Overall performance can be seen by comparing the Tar ("Target") and Act ("Actual") fields.  The contributions of the PID controller's FF (feed-forward), P, I and D to the output can be seen in the FF, P, I and D fields.

  .. image:: ../images/rover-tuning-navigation-logs.png
      :target: ../_images/rover-tuning-navigation-logs.png

How the Position Controller works
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Compares the vehicle's current location to the desired location and calculating a desired 2D velocity to close the gap
- Compares the vehicle's current velocity to the above velocity (aka position correction velocity) + the S-Curve provided velocity and calculating a desired 2D acceleration to reduce the error
- The vehicle forward-back portion of position correction velocity + S-Curve velocity is the target forward speed
- the lateral component of the 2D acceleration becomes the target turn rate

.. _rover-tuning-navigation-path-follower:

Path Follower
-------------

The path follower is an optional steering method for waypoint navigation (:ref:`Auto<auto-mode>`, :ref:`Guided<guided-mode>`, :ref:`RTL<rtl-mode>` and :ref:`SmartRTL<smartrtl-mode>`) for vehicles that must hold a line closely, both on straight segments and through tight corners, such as mowers and other agricultural vehicles working in rows.  On a skid-steer mower it reduced crosstrack error on straight rows by up to 25% and roughly halved how far the vehicle swung outside corners compared with a well-tuned position controller on the same mission, at a cost of about 3% more time (see :ref:`How the Path Follower works<rover-tuning-navigation-path-follower-how>` below).

.. note::

    The path follower is not included in firmware for autopilots with 1 MB of flash (see :ref:`common-limited_firmware<common-limited_firmware>`).  It can be included in a firmware built with the `Custom Firmware Build Server <https://custom.ardupilot.org>`__.

To enable and tune the path follower:

- Tune the :ref:`turn rate<rover-tuning-steering-rate>` and :ref:`speed<rover-tuning-throttle-and-speed>` controllers and the :ref:`position controller<rover-tuning-navigation-position-controller>` first.  The path follower passes its turn rate and speed to the first two.  It uses :ref:`PSC_POS_P<PSC_POS_P>` to keep the vehicle's speed matched to the planned position along the path, and the position controller still steers in :ref:`some situations<rover-tuning-navigation-path-follower-how>`.
- Set :ref:`WP_PF_ENABLE<WP_PF_ENABLE>` = 1
- Measure the steering delay.  Drive a mission in Auto, then compare the desired turn rate with the actual turn rate in the onboard log (STER.DesTurnRate and STER.TurnRate).  The delay is the time from a change in the desired turn rate to the matching change in the actual turn rate.
- Set :ref:`WP_PF_DELAY<WP_PF_DELAY>` to about 1.5 times the measured delay (the default is 0.25 seconds).  This limits how aggressively the vehicle steers back to the path at higher speeds, which prevents weaving across the line.  If the vehicle weaves slowly across the line on straight segments, increase this value.  If it drifts back to the line too slowly, reduce it.
- Set :ref:`WP_PF_DIST<WP_PF_DIST>` to the distance over which the vehicle should return to the path, typically 3 to 5 m.  This mainly affects slow-moving vehicles, because at higher speeds :ref:`WP_PF_DELAY<WP_PF_DELAY>` usually sets the limit instead.
- Set :ref:`WP_PF_RADIUS<WP_PF_RADIUS>` to the radius of the arc the vehicle should drive at each corner.  Smaller values give tighter corners but lower corner speeds.  Skid-steer vehicles ignore :ref:`TURN_RADIUS<TURN_RADIUS>` during waypoint navigation, so this sets their corner radius.
- If the vehicle starts each corner late or keeps turning after the corner ends, increase :ref:`WP_PF_LEAD<WP_PF_LEAD>`, which starts each change in curvature earlier.  A value close to the measured steering delay is a good starting point (0.2 seconds worked well on a hydrostatic mower).  If the vehicle turns in early and cuts inside the corner, reduce it.
- If the vehicle tracks consistently to one side of straight segments (for example on a side slope), increase :ref:`WP_PF_KI<WP_PF_KI>` (0.1 is a reasonable starting value).
- If the vehicle cannot hold the curve through corners, first check that the :ref:`turn rate<rover-tuning-steering-rate>` controller follows the desired turn rate closely.  Corners can then be slowed by reducing :ref:`ATC_TURN_MAX_G<ATC_TURN_MAX_G>` or :ref:`ATC_STR_RAT_MAX<ATC_STR_RAT_MAX>`, which also affect other modes.

The path follower's performance is recorded in the onboard log's PFOL message (when the NTUN bit of :ref:`LOG_BITMASK<LOG_BITMASK>` is set), which includes the crosstrack error (Y), heading error (HE), path curvature (C), desired turn rate (W), integrator contribution (I) and speed limit (V).

.. _rover-tuning-navigation-path-follower-how:

How the Path Follower works
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Instead of passing position errors through the position controller's velocity PID, the path follower calculates the desired turn rate directly from the vehicle's distance from the path, its heading relative to the path and the path's curvature.  The steering law is the path-following law of `Samson (1995) <https://doi.org/10.1109/9.362899>`__, as applied to farm tractor guidance by `Thuilot et al. (2002) <https://hal.archives-ouvertes.fr/hal-02466250>`__ (`DOI <https://doi.org/10.1023/A:1015678121948>`__).  Like the latter, its gains are set as a distance traveled rather than a time, so at low and moderate speeds the vehicle returns to the path over the same distance whatever its speed.  The gradually tightening and widening curves at each corner (see below) are `Euler spirals <https://en.wikipedia.org/wiki/Euler_spiral>`__, also called clothoids, the transition curves used in road and railway design.  Their curvature changes in proportion to the distance traveled, so at a steady speed the turn rate changes at a steady rate, which keeps it within the vehicle's steering acceleration limit.

Each corner is planned as a smooth curve (straight, gradually tightening curve, arc, gradually widening curve, straight) whose arc radius is set by :ref:`WP_PF_RADIUS<WP_PF_RADIUS>` (but never less than :ref:`TURN_RADIUS<TURN_RADIUS>` on vehicles that cannot pivot).  The vehicle slows down before each corner so it can follow the curve within its turn rate, lateral acceleration and steering acceleration limits (:ref:`ATC_STR_RAT_MAX<ATC_STR_RAT_MAX>`, :ref:`ATC_TURN_MAX_G<ATC_TURN_MAX_G>` and :ref:`ATC_STR_ACC_MAX<ATC_STR_ACC_MAX>`), so corners on very short legs, such as the connector of a tight U-turn, are driven more slowly.  Vehicles that cannot pivot slow further on such corners so their steering can keep up, so leave enough room between waypoints for them to turn at :ref:`TURN_RADIUS<TURN_RADIUS>` with some distance to spare.  If a leg starts with the vehicle pointing well away from the path (for example from rest), the vehicle first turns toward the path and then follows it.

This is where most of its advantage in corners comes from.  The position controller follows a target point whose speed through a corner is set by the acceleration limits, not by how fast the vehicle can turn.  In a tight corner it can ask for more turn rate than the vehicle can deliver, then overshoot the next segment while the turn rate winds back down.  The path follower plans each corner within the vehicle's limits, including when to start unwinding the turn, and slows to a speed at which the vehicle can follow it.  Because the vehicle slows for each corner instead of running wide, missions with many tight corners take a little longer.

The position controller still steers during pivot turns (see :ref:`WP_PIVOT_ANGLE<WP_PIVOT_ANGLE>`), when reversing, when stopping at a waypoint, and during object avoidance using BendyRuler.  Object avoidance using Dijkstra's steers along each avoidance segment with the path follower.  :ref:`Simple object avoidance<rover-object-avoidance>` and fences still slow and stop the vehicle, but the path follower does not steer the vehicle to slide along an obstacle or fence, so the vehicle stops instead.  If simple avoidance backs the vehicle up, the position controller steers.
