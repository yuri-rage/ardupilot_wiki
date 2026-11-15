.. _rover-tuning-navigation:

=================
Tuning Navigation
=================

This page describes how to tune the Navigation control including "S-Curves" and the "Position controller".  The lower level :ref:`speed<rover-tuning-throttle-and-speed>` and :ref:`turn rate<rover-tuning-steering-rate>` controllers should be tuned before attempting to tune this controller.

.. image:: ../images/rover-tuning-navigation1.png
    :target: ../_images/rover-tuning-navigation1.png

S-Curves and the position controller are used all the autonomous modes including :ref:`Auto<auto-mode>`, :ref:`Guided<guided-mode>`, :ref:`RTL<rtl-mode>` and :ref:`SmartRTL<smartrtl-mode>`.

S-Curves
--------

S-Curves are used to plan a smooth path that brings the vehicle close to each waypoint while not exceeding speed or acceleration limits.  The resulting path includes continuous position and 2D velocity targets.

Factors affecting the path include:

- The vehicle will cut the corners but will always try to pass within :ref:`WP_RADIUS<WP_RADIUS>` or :ref:`TURN_RADIUS<TURN_RADIUS>` (whichever is larger) of each waypoint
- The vehicle will cut the corners more at higher speeds
- The vehicle will cut the corners more if the maximum accelerations are reduced (e.g. the lowest of :ref:`ATC_ACCEL_MAX<ATC_ACCEL_MAX>`, :ref:`ATC_DECEL_MAX<ATC_DECEL_MAX>` and :ref:`ATC_TURN_MAX_G<ATC_TURN_MAX_G>` * 9.81)
- The vehicle will slow down in the corners if necessary to pass within :ref:`WP_RADIUS<WP_RADIUS>` of the waypoint without exceeding the maximum accelerations (e.g. the lowest of :ref:`ATC_ACCEL_MAX<ATC_ACCEL_MAX>`, :ref:`ATC_DECEL_MAX<ATC_DECEL_MAX>` and :ref:`ATC_TURN_MAX_G<ATC_TURN_MAX_G>` * 9.81)
- Placing waypoints close together may lead to the vehicle traveling slowly

.. _rover-tuning-navigation-pos-control:

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

Stanley Controller
------------------

The Stanley controller is an optional path-following controller for :ref:`Auto<auto-mode>` mode. It steers to correct two errors at once: the vehicle's heading relative to the path, and its sideways distance from the path (cross-track error). Corners between waypoints are rounded into circular arcs ("fillets"), and the vehicle is steered around each arc in advance (feed-forward) rather than waiting for an error to build. It is based on `Hoffmann et al., 2007 <https://ai.stanford.edu/~gabeh/papers/hoffmann_stanley_control07.pdf>`_.

Set :ref:`ATC_STAN_USE<ATC_STAN_USE>` to 1 to enable it. Make sure ``LOG_BITMASK`` includes Steering, as the tuning steps below rely on the onboard logs.

.. note::
   Stanley only steers during :ref:`Auto<auto-mode>` waypoint navigation. The standard controllers take over during each pivot turn and on reverse legs (set by :ref:`DO_SET_REVERSE<mav_cmd_do_set_reverse>`), then Stanley resumes. Stanley is not used when :ref:`SAIL_ENABLE<SAIL_ENABLE>` is set.

   On Ackermann vehicles, Stanley sets the steering angle directly and the :ref:`steering rate controller <rover-tuning-steering-rate>` is not used. On skid-steer vehicles, Stanley's output is converted to a turn rate, so the steering rate controller must be tuned first.

Tune using a mission with long straight legs and a few 90° corners, driven at the speed you will normally operate (the gains depend on speed). Start somewhere open, at low speed, until the vehicle is behaving well.

Recommended tuning steps:

- **Set Vehicle Geometry**:

  - Ackermann: set :ref:`ATC_WHLBASE_LEN<ATC_WHLBASE_LEN>` to the distance between the front and rear axles (m) and :ref:`ATC_STR_ANG_MAX<ATC_STR_ANG_MAX>` to the maximum steering angle (deg). Both are used directly to calculate steering.
  - Skid-steer: set :ref:`ATC_WHLBASE_LEN<ATC_WHLBASE_LEN>` to the distance between the front and rear wheels, or the length of the track footprint (m). Leave :ref:`ATC_STR_ANG_MAX<ATC_STR_ANG_MAX>` at its default to start. See :ref:`Tuning Skid-Steer Vehicles<rover-tuning-navigation-stanley-skid>` below.

- **Check the Steering Rate Controller (skid-steer only)**: Stanley steers through the :ref:`steering rate controller <rover-tuning-steering-rate>`, and a slow one makes the vehicle weave slowly on straights (swinging across the path every few seconds) even at default Stanley gains. Drive a mission in Auto and compare ``STER.TurnRate`` with ``STER.DesTurnRate``: the actual turn rate should follow the demand within a few tenths of a second. If it lags by much more, address that before tuning the Stanley gains: retune the steering rate controller, and check for deadband or backlash in the drive, which also delay the response (possible on hydrostatic or other mechanically linked drives).

- **Tune the Speed Profile**: Stanley only steers; speed still comes from the S-curve planner. With the :ref:`speed controller<rover-tuning-throttle-and-speed>` tuned:

  - Set :ref:`WP_SPEED<WP_SPEED>` to the desired speed between waypoints, then drive a mission in Auto and compare ``THR.DesSpeed`` with ``THR.Speed``.
  - If speed lags the demand when speeding up or stopping, reduce :ref:`WP_ACCEL<WP_ACCEL>` (0 uses the lower of :ref:`ATC_ACCEL_MAX<ATC_ACCEL_MAX>` and :ref:`ATC_DECEL_MAX<ATC_DECEL_MAX>`).
  - If the vehicle enters corners too fast and runs wide, reduce :ref:`ATC_TURN_MAX_G<ATC_TURN_MAX_G>`; if it slows more than necessary, increase it (see also :ref:`Tuning Corners<rover-tuning-navigation-stanley-corners>`).
  - Leave :ref:`PSC_POS_P<PSC_POS_P>` at its default unless speed surges or hunts on straights (reduce it). The ``PSC_VEL_*`` gains are not used while Stanley is steering.

- **Cross-Track Gain and Damping** (:ref:`ATC_STAN_K<ATC_STAN_K>`, default 1.0, and :ref:`ATC_STAN_KD<ATC_STAN_KD>`, default 0): K sets how firmly the vehicle steers back to the path; KD damps the turn rate so the vehicle settles without weaving.

  - Increase :ref:`ATC_STAN_K<ATC_STAN_K>` in steps of 0.1 to 0.2 until the vehicle begins to weave (``STAN.XTrack`` swings back and forth across zero on the straights) or overshoots the path after corners.
  - Add damping: set :ref:`ATC_STAN_KD<ATC_STAN_KD>` to about 10% of K and increase in steps of 0.05 until the weaving stops.
  - Repeat while higher K still settles, then finish with K slightly below the point where weaving returns. If corrections become sluggish or steering becomes jittery, reduce KD.

- **Low-Speed Softening** (:ref:`ATC_STAN_V0<ATC_STAN_V0>`, default 1.0 m/s): limits how sharply the vehicle corrects at low speed. Most vehicles can leave this at the default. Increase it if steering is jerky or violent when starting to move; decrease it if corrections at low speed are sluggish.
- **Corner Lead (optional)** (:ref:`ATC_STAN_LEAD<ATC_STAN_LEAD>`, default 0 s): starts and finishes each corner earlier to make up for slow steering response, which is common on hydrostatic skid-steer vehicles that turn late and swing wide on corner exit. Set it to the delay between a turn rate request and the vehicle's response, found by comparing ``STER.DesTurnRate`` with ``STER.TurnRate`` in the logs (typically 0.2 to 0.4 s for skid-steer vehicles; if unsure, start at 0.3 s). Reduce it if the vehicle cuts inside corners.
- **Cross-Track Integral (optional)** (:ref:`ATC_STAN_KI<ATC_STAN_KI>`, default 0): removes a steady offset from the path, such as drifting downhill on a side slope or in a steady cross-wind or current. Start at about 10% of :ref:`ATC_STAN_K<ATC_STAN_K>` (typically 0.05 to 0.25), or leave at 0 if there is no steady offset. Correct mechanical steering trim with ``SERVOn_TRIM`` instead. If ``STAN.ITerm`` swings back and forth with each weave rather than settling to a steady value, KI is too high: reduce it.

  The learned correction reverses automatically on return passes (e.g. after a U-turn at the end of a row). It pauses in corners, while the vehicle is still converging on the path, and while stopped, and it fades slowly (over about 30 s) so it cannot build up indefinitely.

.. _rover-tuning-navigation-stanley-skid:

Tuning Skid-Steer Vehicles
^^^^^^^^^^^^^^^^^^^^^^^^^^

Skid-steer vehicles have no steering angle, so Stanley treats :ref:`ATC_WHLBASE_LEN<ATC_WHLBASE_LEN>` and :ref:`ATC_STR_ANG_MAX<ATC_STR_ANG_MAX>` as tuning values:

- **ATC_WHLBASE_LEN**: larger values look further ahead and soften steering; smaller values sharpen steering but look less far ahead.
- **ATC_STR_ANG_MAX**: caps how hard Stanley can turn. Increase it if the vehicle cannot turn sharply enough; decrease it if steering is violent.

Stanley's steering angle (Delta) becomes a turn rate of ``max(speed, 0.5 m/s) * tan(Delta) / ATC_WHLBASE_LEN``, limited by :ref:`ATC_STR_RAT_MAX<ATC_STR_RAT_MAX>`. At speed ``v``, the tightest arc the vehicle can follow is about ``v / ATC_STR_RAT_MAX`` (with ATC_STR_RAT_MAX in radians per second).

.. _rover-tuning-navigation-stanley-corners:

Tuning Corners
^^^^^^^^^^^^^^

Corner problems are usually caused by limits rather than gains, so increasing :ref:`ATC_STAN_K<ATC_STAN_K>` rarely helps. Check the logs:

- **Turns late and swings wide on exit, with steering not at its limits**: increase :ref:`ATC_STAN_LEAD<ATC_STAN_LEAD>`.
- **STAN.Delta stays at ATC_STR_ANG_MAX for much of each corner** (Delta is logged in radians; e.g. 60° is 1.05 rad): the corner is as tight as the vehicle can steer, leaving no room to correct errors. Increase :ref:`TURN_RADIUS<TURN_RADIUS>` above ``ATC_WHLBASE_LEN / tan(ATC_STR_ANG_MAX)`` so corners are planned with steering to spare (corners become rounder). If sharp corners are limited by :ref:`WP_RADIUS<WP_RADIUS>`, increasing it also helps.
- **Skid-steer: STER.DesTurnRate stays at ATC_STR_RAT_MAX**: the vehicle cannot turn fast enough at its corner speed. Reduce :ref:`ATC_TURN_MAX_G<ATC_TURN_MAX_G>` to slow down for corners, or increase :ref:`ATC_STR_RAT_MAX<ATC_STR_RAT_MAX>` if the vehicle can turn faster.

Each corner arc is sized to keep the vehicle within :ref:`ATC_TURN_MAX_G<ATC_TURN_MAX_G>` at its speed entering the corner (radius ``speed² / (ATC_TURN_MAX_G * 9.81)``), made smaller if needed to stay within about :ref:`WP_RADIUS<WP_RADIUS>` of the waypoint, but never smaller than the minimum turn radius: the larger of :ref:`TURN_RADIUS<TURN_RADIUS>` and ``ATC_WHLBASE_LEN / tan(ATC_STR_ANG_MAX)``. :ref:`TURN_RADIUS<TURN_RADIUS>` has no effect if it is smaller than the second value; setting it above that value reserves steering for corrections in corners.

Evaluating Performance via Logs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

With Steering logging enabled, Stanley writes the ``STAN`` message to the :ref:`onboard log <common-downloading-and-analyzing-data-logs-in-mission-planner>`:

- **XTrack**: cross-track error at the vehicle's position (m, positive when left of the path)
- **XTrackFA**: cross-track error projected forward to the front axle (m)
- **HdgErr**: heading error relative to the path (rad)
- **XTerm**, **ITerm**, **DTerm**, **FF**: cross-track, integral, damping and corner feed-forward steering terms (rad). FF starts before the corner when :ref:`ATC_STAN_LEAD<ATC_STAN_LEAD>` is set.
- **Delta**: total steering angle after limits (rad)

Other Parameters
----------------

- :ref:`WP_SPEED<WP_SPEED>` defines the target speed (in m/s) that the vehicle will drive at in :ref:`Auto<auto-mode>` and :ref:`Guided<guided-mode>`.  It will also be used in :ref:`RTL<rtl-mode>` and :ref:`SmartRTL<smartrtl-mode>` if :ref:`RTL_SPEED<RTL_SPEED>` is set to zero
- :ref:`WP_RADIUS<WP_RADIUS>` specifies the distance before the waypoint that the vehicle may begin turning towards the next waypoint
