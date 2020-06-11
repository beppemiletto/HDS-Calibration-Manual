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

Injectors
#########

From Wikipedia, the free encyclopedia  `Fuel Injection <https://en.wikipedia.org/wiki/Fuel_injection#Fuel_injector>`_.

HDS system is based on :term:`MPI`. More precisely is capable of accurate Sequentially Phased Multipoint Port Injection. It can be used also as :term:`SPI` with the assumption that in any case the numbers of driven injectors must be same as the number of cylinders declared in :guilabel:`asNUMCYL`.

The most common fuel metering devices, notwithstanding with the type of fuel managed, for spark ignited engines are injectors.

Fuel injectors are very quick-acting on off valves capable of being cycled (that is opened and closed) in the order of few milliseconds. Injectors are available in a variety of flow rates and are also divided into low impedance and high impedance injectors depending on their electrical resistance. Peak-and–hold (P&O) drivers can drive both low impedance and high impedance injectors while saturation drivers can drive high impedance injectors only.

.. note::

    HDS system features 8 P&H injector drivers.

.. warning::

    When connecting more than one injector to a driver they must be used the "Series" connection and not the "Parallel". Make sure that the total resistance and inductance of the driven injector assembly considering the available supply voltage allows the correct opening current.

The calibration of the injector is divided into 2 different sections:

1. Injector's electrical model - to control in the proper way the injector opening current waveform

2. Injector's fluid dynamic model - to control the correct quantity of fuel delivered

Injector's electrical model
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Injector's estimated temperature
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The behaviuor of the injector when driven by means of a current is affected by its own temperature. Since it is not possible to provide a specific sensor, HDS features an injector's temperature model based on three normally measured temperatures:

1. ``zsTAir`` meaning the environmental temperature, especially at engine start after the overnight soak

2. ``zsTh2o`` meaning the general engine thermal status

3. ``zsTRail`` meaning the temperature of the media flowing through the injector

.. tip:: ``qsInjTemperature`` is the estimated temperature calculated with following function:

    .. math::
        \rm\small{qsInjTemperature =  zsTAir * qsK\_TAIR + zsTh2o * qsK\_TH2O + zsTRail * qsK\_TRAIL}
        :label: *qsInjTemperature* function

    The three calibration factors are resumed in following table:

    .. table::  **qsInjTemperature** calibration coefficients

        +-------------------+---------------------------+
        | Input Parameter   | Calibration coefficient   |
        +-------------------+---------------------------+
        | ``zsTAir``        | :guilabel:`qsK_TAIR`      |
        +-------------------+---------------------------+
        | ``zsTh2o``        | :guilabel:`qsK_TH2O`      |
        +-------------------+---------------------------+
        | ``zsTRail``       | :guilabel:`qsK_TRail`     |
        +-------------------+---------------------------+

    from the function is intuitive that the coefficients values must be complemented to the unit. So

    .. math::
        \rm\small{(qsK\_TAIR + qsK\_TH2O + qsK\_TRAIL ) = 1}
        :label: *qsInjTemperature* coefficients unitary complements rule

    ``qsInjTemperature`` is used to enter many tables of the injector's electrical model.

The Injector's current control
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The P&H drivers of HDS need operating parameters to be set according to electric and physical parameters:

1. ``zsVBPW`` the voltage available after the power relay supplied to the injector:

2. ``qsInjTemperature`` the estimated injector's temperature.

.. note::

    In below figure a typical P&H command of HDS injector's driving circuit. HDS has injectors current controller with a closed loop on the target current value. In the picture in red the measured current flowing into injector coil and in blu the applied voltage. The current is controlled by means of :term:`PWM` command: they are clearly indicated both the choppered parts of the command (reaching Peak and following Hold targets) as well as the corresponding controller ripples on the target currents.

|lay_400|

.. tip:: The P&H parameters setting

    ``bsInjPeakCurrent`` [mA] is the applied Peak Current that is interpolated in the table  :guilabel:`qtPEAK_CURRENT` with inputs  ``zsVBPW`` and ``qsInjTemperature``. Limits low 1000 [mA] and high 8200 [mA] (the highest value available from HDS HW).

    ``bsInjPeakTime`` [|micro|\s] is the applied Peak time to reach the peak current that is interpolated in the table  :guilabel:`qtT_PEAK` with inputs  ``zsVBPW`` and ``qsInjTemperature``. Limits low 200 [|micro|\s] and high 8200 [|micro|\s].

    ``bsInjHoldCurrent`` [mA] is the applied Hold Current that is interpolated in the table  :guilabel:`qtHOLD_CURRENT` with inputs  ``zsVBPW`` and ``qsInjTemperature``. Limits low 200 [mA] and high 8200 [mA] (the highest value available from HDS HW).

    ``bsInjFastDecayTime`` [|micro|\s] is the applied Peak time to reach the peak current that is copied from scalar  :guilabel:`qsFAST_DECAY_TIME`. Default value 0 [|micro|\s].

    Following the injector's manufacturer specification, by means of a oscilliscope with a current clamp, calibrate the tables and scalars in order to fit in the best way the specified values.

.. warning:: **Impossible currents settings**

    Some calibrated current values cannot be achieved in case of physical Ohm law :math:`I=\frac{V}{R}` violation. E.g. in case is required a current of 8000 [mA] when the supply Voltage is 16000 [mV] (as in an engine crank phase with low batteries in a 24 [V] electrical system) with a injector resistance of 2.2 [Ohm]. In this case the maximum current reachable is  :math:`I=\frac{V}{R} = \frac{16000 [mV]}{2.2 [Ohm]} = 7272 [mA]`. To be sure to achieve the target the calibration should be somehow low than the physical limit, i.e. 7200 [mA].

    Also, is very common reach high resistance when using series of injectors. In this case pay also attention to the total impedance, since the coils of the injectors are a inductance that are summed in case of series connection. In some cases the time to reach the resistive current may become much longer than the one with only one injector.

.. warning:: **Saturated command Injectors**

    Saturated command current injectors must be calibrated using the maximum current allowed by the specification of the manufacturer both in the :guilabel:`qtPEAK_CURRENT` and  :guilabel:`qtHOLD_CURRENT` and setting (leaving) :guilabel:`qsFAST_DECAY_TIME` to 0. The :guilabel:`qtT_PEAK` is a don't care parameter and can be set to a minimal value. In case the manufacturer only specifies the operating voltages (minimum and maximum) and not the current , before setting the target current in the tables, perform current measures applying the specified DC voltages. Then calibrate the table in the measured range. Verify after calibration the real mean current value achieved by the HDS driver.

Injectors Cold De-sticking
~~~~~~~~~~~~~~~~~~~~~~~~~~

Injectors in cold condition, e.g. with temperature below -20 [°C] may have problems to be opened the really first times. HDS features a special strategy to try to fast warm-up the injector assy as well as apply an extra opening force.These actions are actuated applying high current for long time on;y once, in a safe condition, before the engine start.

.. warning::

    For testing the Injectors Cold De-sticking strategy all the calibrations must be present into ECU from key on so a download of the calibrations must be done for having the strategy ready for working at start-up of ECM.

.. tip::

    The strategy is activated by a general enable calibration: :guilabel:`qsCOLD_DESTICKING_EN` must be set to 1 for enable (0 disable).

    When the general enable is on then if two of this three conditions are verified as True:

        1. ``zsTh2o`` <= :guilabel:`qsTH2O_LOW_THR`

        2) ``zsTAir`` <= :guilabel:`qsTAIR_LOW_THR`

        3) ``zsTRail`` <= :guilabel:`qsTRAIL_LOW_THR`

    the strategy is ready to run at key on with engine still not running (not even in crank phase): ``qsColdDestikingCnd`` = 1 AND ``ssEngState`` = *WAIT* are the conditions that is possible to monitor with calibration software.

    If also the battery voltage after the power relay ``zsVBPW`` > :guilabel:`qsVBPW_LOW_THR` calibration threshold (the current must be provided, if not available is only a worst condition for starting engine),

    ``qsColdInjRunning`` goes to 1.

    If all conditions are verified properly,  right after the *key on* a sequence of injection pulses is performed in the numerical series 1 to ``bsNUMCYL`` every :guilabel:`qsDELTA_START` [ms]. The single pulse (the one applied to each injector) has a total duration of :guilabel:`qsCOLD_TINJ` [|micro|\s]  using :guilabel:`qsCOLD_PEAK_CURR`  [mA] for :guilabel:`qsCOLD_PEAK_TIME` [|micro|\s] and in  :guilabel:`qsCOLD_FASTDEC_TIME` [|micro|\s] the current move to  :guilabel:`qsCOLD_HOLD_CURR` [mA].

    :guilabel:`qsCOLD_HOLD_CURR`  can be calibrated equal to :guilabel:`qsCOLD_PEAK_CURR` . :guilabel:`qsCOLD_FASTDEC_TIME`   can be set to 0.

Below in figure an example of strategy activation on a 8 cylinder engine.

|lay_410|

Injector's fluid dynamic model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. sidebar:: Minimum pressure ratio required for choked flow to occur

    .. list-table:: Choked flow for gases
        :widths: 50 50
        :header-rows: 1

        * - Gas
          - Min :math:`\frac {P_u} {P_d}`
        * - Dry air
          - 1.893
        * - Nitrogen
          - 1.895
        * - Oxygen
          - 1.893
        * - Helium
          - 2.049
        * - Hydrogen
          - 1.899
        * -  Methane
          -  1.837
        * - Propane
          - 1.729
        * - Butane
          - 1.708
        * - Ammonia
          - 1.838
        * - Chlorine
          - 1.866
        * - Sulfur dioxide
          - 1.826
        * - Carbon monoxide
          - 1.895
        * - Carbon dioxide
          - 1.83


The injector model in HDS is based on the assumption that is possible to reach, in every working conditions of the engine, the `Choked flow <https://en.wikipedia.org/wiki/Choked_flow>`_ condition. With a good approximation this condition is reached in NG injectors when the P\ :sub:`u` meaning Injector Rail absolute pressure ``zsPrail`` is at least 2 times the P\ :sub:`d` downstream absolute pressure ``zsMap`` (really for methane the factor is 1.837).

Generally speaking the HDS's injector model calculate an injector's opening duration for a single injection, meaning that it occurs every cylinder combustion cycle or every intake phase, having in input the fuel quantity in mass. In other words convert a fuel mass to be delivered during a single stroke of the injector in the opening time needed by the injector itself to deliver this amount of fuel. The result of the calculation is ``qsTjBase``. The basic transfer function for the injector model can be assimilated for a triad of rail pressure values, gas temperature and injectors supply voltage to a straight line with angular coefficient and offset in calibration.

Since it is possible to have an injector scheme with more than one injector per cylinder, the angular coefficient of the straight line, also called injector gain, is divided by the number of injectors present for each cylinder.

.. _General_injector_model_transfer_function:

    .. math::
        \rm\tiny{ qsTjBase = qsQInjCorr * \frac {qsInjgain} {qsN\_INIET\_CYL} + (qsInjODpre + qsInjOVbpw)}
        :label: *General injector model transfer function*

    where

    * :math:`\rm\small qsTjBase [\mu s]` is the result of the calculation visible with ``qsTjBase`` symbol.

    * :math:`\rm\small qsQInjCorr [mg/cycle]` is the quantity of fuel to be injected for the present combustion cycle in the single cylinder visible with ``qsQInjCorr`` symbol.

    * :math:`\rm\small qsInjgain [\mu s / mg]` is the Injector Gain visible with ``qsInjgain`` symbol interpolated every TDC on the table :guilabel:`qtINJGAIN` in function of the rail gas pressure ``zsPRail`` and temperature ``zsTRail``. For a compressible media pressure and temperature are functional of the media density.

    * :math:`\rm\small qsN\_INIET\_CYL [n]` is the number of injectors grouped in a single cylinder that is possible to set with scalar calibration  :guilabel:`qsN_INIET_CYL` symbol that can be set to the values 1 or 2..

    * :math:`\rm\small qsInjODpre [\mu s]` is the Injector Time Offset visible with ``qsInjODpre`` symbol interpolated every TDC on the table :guilabel:`qvINJ_O_DPRE` in function of the rail gas pressure ``zsPRail``.

    * :math:`\rm\small qsInjOVbpw [\mu s]` is the Injector Time Offset visible with ``qsInjOVbpw`` symbol interpolated every TDC on the table :guilabel:`qvINJ_O_VBPW` in function of injector supply voltage ``zsVBPW``.

.. tip::

    The accurate calibration of the injector's fluid dynamic model needs some **dedicated instrumentation**:

    1. a gas flow meter ro measure the real mass flow rate of the gas in kg/hour. During the calibration process take care of identify always the calculated flow rate ``vsFuelMassFlow`` [kg/h] with the measured mass flow value.

    2. a device to control the pressure in the rail ``zsPRail`` at a desired value. It can be one of the followings:

        a. an electronic CNG pressure regulator is the best choice since is able to keep continuously the pressure at a target value not depending from flow rate or tank source pressure.

        b. a mechanic CNG variable pressure regulator, may be necessary to trim continuously the regulation device since the variations of flow rate can affect the output pressure

        c. a standard CNG pressure regulator with MAP compensation connected to a source of variable pressure (like a compressor with a air pressure regulator). Like b. also this solution may need to be set every time the flow rate change.

    3. a variable voltage power supply to control the ``zsVBPW`` in the range 12 to 32 [V] and current limit 30 [A]

    .. warning:: **Injectors specifications limits? No, thank.**

        **Before to approach the calibration of this model it must be clear the operating range to be calibrated.** At a first glance it may be restricted to the injector manufacturer specification range of pressure, temperatures and voltages. But in real usage some extreme conditions may be found (i.e. is very common the rail under-pressure due to empty tank) and is better to have available a setting that enable at least a safe recovery limp home working mode. Mis-fuelling may very easy lead to misfire with dramatic effect on the torque generation of the engine and catalyst damages.

    .. note:: Empiric or physic method? Both!

        The injector's fluid dynamic model is a physical model. But fuel rail is an industrial designed part that includes a lot of simplifications and built with many constraints. The measure ot the gas pressure and temperature is done in the best way but may be the measures are not fitting perfectly the exact working conditions of the injectors. For this reason an experimental session for calibrate the most important parameters is strongly suggested. But the dimension of the tables and the extension of their range may require a huge amount of work. For this other reason an hybrid approach is explained that includes both experimental data gathering and physical calculation.

    **Identifying the straight lines**

    As explained in the :ref:`model transfer function<General_injector_model_transfer_function>` in steady condition of gas pressure and temperature and supplied voltage, the injector's fluid dynamic model is a  straight line.

    |lay_420|

    Each point of :guilabel:`qtINJGAIN` table is a slope f(``zsPRail``, ``zsTRail``). Each point of :guilabel:`qvINJ_O_DPRE` is a part of the offset f(``zsPRail``). Each point of :guilabel:`qvINJ_O_VBPW` is a part of the offset f(``zsVBPW``).

    A. Assuming to start working at a supply nominal voltage (i.e. 27.5 [V] for 24 [V] electrical system) set the value of the offset at this break point to 0 in :guilabel:`qvINJ_O_VBPW`.

    B. Define the nominal value of ``zsPRail`` and set the pressure exactly at this value. Set a central :guilabel:`qvINJ_O_DPRE` table breakpoint at the nominal value and set the table all to 0.

    C. Running engine at intermediate engine speed (e.g. 1250 [rpm]), inducing a high value of ``qsQInjCorr`` (the rightmost point in independent axis - e.g. applying high load to engine and/or forcing if possible the **qfQINJ_ADAPT** fix symbol) set a value in :guilabel:`qtINJGAIN` at ``zsPRail`` and  ``zsTRail`` closest crossing breakpoints so that the measured flow rate of fuel is equal to ``vsFuelMassFlow`` symbol.

    D. Reduce the value of ``qsQInjCorr`` (the leftmost point in independent axis - e.g. applying low load to engine and/or forcing if possible the **qfQINJ_ADAPT** fix symbol). Set a value in :guilabel:`qvINJ_O_DPRE` at ``zsPRail`` breakpoint so that the measured flow rate of fuel is equal to ``vsFuelMassFlow`` symbol.

    E. Checking that ``zsTRail`` is not changed so much (up to 5 degrees + / - the accuracy is still good enough) from C. , repeat the C. procedure to adjust the slope after having modified the offset.

    F. Repeat the procedure D. to adjust the offset.

    G. Repeat E. and F. steps until the adjustment of slope and offset of the first straight line is stable.

    H. Annotate the slope value (the interpolated value in :guilabel:`qtINJGAIN` ) ``qsInjgain`` ,   ``zsPRail`` and  ``zsTRail``. They will be used for the  following physical model fit extrapolations.

    Repeat the procedure from B. to H. for all remaining ``zsPRail`` real working points setting a specific breapoint in  :guilabel:`qtINJGAIN`. Don't care for the moment about the temperature of the rail dependencies. They will be set in a second moment.







