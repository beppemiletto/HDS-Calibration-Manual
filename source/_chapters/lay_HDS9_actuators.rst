.. include:: <isonum.txt>
.. include:: ../_static/figures.txt
.. include:: ../_static/lay-eng/figures.txt

Actuator parameters setting
---------------------------

To manage the operation of the engine in order to satisfy the external requests, the control system processes the input signals and, through internal models, determines the commands for the actuators. The actuators then transform these commands into physical actions that modify the behavior of the engine. The actuators are different for each application. For this reason, at the very beginning of a project is necessary to calibrate the specific parameters of each actuator used for the specific application.

Ignition coils
##############

For a good spark and for coils reliability reasons it’s important to reach the correct maximum primary current value indicated by the manufacturer in the specifications. This value depends on the coil’s physical characteristics, the dwell time and the supply voltage. The vector :guilabel:`jvDW_TIME_OL` permit to set the dwell as a function of battery voltage after the power relay.

|lay_300_C|

|lay_300_I|

