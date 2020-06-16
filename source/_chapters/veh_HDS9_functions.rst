.. include:: <isonum.txt>
.. include:: ../_static/figures.txt
.. include:: ../_static/veh-fcn/figures.txt

|veh_010|

Vehicle functions
-----------------

In HDS the vehicle functions group provide a set of controls that interact with the ECM by means of inputs and / or actuator that are not mounted on the base engine itself. E.g. the starter motor, an auxiliary of the engine, may be required to be activated by the ECM. Other example is the Air Conditioning Compressor activation (often a clutch) that may be required to be activated by the ECM. More and more these set of function on a vehicle are placed in the range of controls of a :term:`VCM`.

Vehicle functions digital inputs
################################

Many vehicle function are activated and / or managed by means of special digital inputs. The digital inputs are activated applying a low level or high level voltage to some dedicated pins of the ECM.

.. note:: Refer to wiring harness schematic for the pin assignment.

The conditioning of digital inputs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All the digital inputs follows the conditioning process as described in present chapter:

    0. **HW selectable Pull-up / Pull-down** some digital input provide a selector for pull-up / pull-down electrical signal conditioning. The general syntax of calibration symbol is :guilabel:`bsPULLN_XXXXXXX_ID`. E.g. for Air conditioning request button the selector is :guilabel:`bsPULL10_AirCondReq_ID`. Calibrating to 1 the pull-up is inserted, calibrating to 0 the pull-down is inserted.

    1. **low level digital status** that can be 0 or 1 depending from the voltage applied to the specific pin - the symbol is a ``bsXXXXXXXXXX`` (e.g. ``bsKeyEngStart`` for request of starter activation).

    2. **signal debouncing** take care of avoid intermittent unwanted sequence of activation / deactivation due to bouncing of an electrical contact. Is a digital filtering with a constant in calibration that represents the number of counted sample at same level of the input to allow the commutation from one status to the other. The calibration can be set from 0 to 255 so realizing a debouncing for 4 ms task till more than 1 second.

    .. warning::

        The debouncing introduce a delay in the input proportional to the calibration of the number of sample. E.g. setting to 50 the debouncing filter constant for a digital input means to accept a delay of 50 * 4 = 200 ms from the instant when the button is pressed till the first reaction of the ECM.

    E.g. the debouncing filter constant for ``bsKeyEngStart`` is :guilabel:`zsKEY_START_DEB_STEP` . The output symbol from the debouncing filter is `zsKeyStartDeb` not present in the ECM database of symbols.

    3. **selectable inversion (Digital Negation DN)** to make the input flexible in the way that for any kind of logical electrical setup connected to the input pin is possible to have a resulting status active or inactive to be propagated to the functions that needs the input, a selectable logical negation can be applied. The calibration selector is :guilabel:`zsDN_XXXXXXXXX` (for our example :guilabel:`zsDN_KEY_ENG_START` ) . The output is the final symbol of the digital input status is ``zsKeyEngStart``. The truth table of this feature, using always the ``bsKeyEngStart`` , is the following:

    .. _truth_table_digital_inversion:
    .. list-table:: Digital inversion Truth Table
       :widths: 30 30 30
       :header-rows: 1

       * - ``bsKeyEngStart``
         - :guilabel:`zsDN_KEY_ENG_START`
         - ``zsKeyEngStart``
       * - F (0) 
         - F (0)
         - F (0)
       * - T (1)
         - F (0)
         - T (1)
       * - F (0)
         - T (1)
         - T (1)
       * - T (1)
         - T (1)
         - F (0)

    4. **fixing the output** The value of the output of the digital input conditioning can be fixed to a constant value independent from the input status. For doing this operation is present the calibration fix :guilabel:`zfXXXXXXX` ( :guilabel:`zfKEY_ENG_START`, divided into :guilabel:`zfKEY_ENG_START_Enable` to activate or deactivate the fixed value set in :guilabel:`zfKEY_ENG_START_Value` in our recurrent example).

    5. **the digital input general ENABLE** This calibration flag propagate to the strategies that may be activated by the digital input the real availability or not of the input. In other words, if the application configuration provide the managing of the digital input it must be defined as *available* (value 1), whether if nothing is connected to the pin and we don't want to have ECM reactions for any kind of status can be detected, it must be defined as *NOT available* (value 0). The general calibration symbol for this configuration calibration is :guilabel:`zsAV_XXXXXXXXXX`  ( :guilabel:`zsAV_KEY_ENG_START` in our example).

Following list is the complete set of digital inputs used for vehicle functions:

.. _digital_inputs_list:
.. list-table:: List of digital inputs
   :widths: 20 20 20 20 20 20 20
   :header-rows: 1

   * - Input
     - Debouncing constant
     - DN selector
     - FIX
     - Output
     - DI Available flag
     - Pull-up selector
   * - ``bsAirCondReq``
     - :guilabel:`zsAIR_COND_DEB_STEP`
     - :guilabel:`zsDN_AIR_CND_REQ`
     - :guilabel:`zfAIR_CND_REQ`
     - ``zsAirCondReq``
     - :guilabel:`zsAV_AIR_CND_REQ`
     - :guilabel:`bsPULL10_AirCondReq_ID`
   * - ``bsStartAccDecCbt``
     - :guilabel:`zsSTART_CBT_DEB_STEP`
     - :guilabel:`zsDN_START_CBT`
     - :guilabel:`zfSTART_CBT`
     - ``zsStartAccDecCbt``
     - :guilabel:`zsAV_START_CBT`
     - :guilabel:`bsPULL8_StartAuxReq_ID`
   * - ``bsStopCbt``
     - :guilabel:`zsSTOP_CBT_DEB_STEP`
     - :guilabel:`zsDN_STOP_CBT`
     - :guilabel:`zfSTOP_CBT`
     - ``zsStopCbt``
     - :guilabel:`zsAV_STOP_CBT`
     - :guilabel:`bsPULL9_StopAuxReq_ID`
   * - ``bsKey``
     - :guilabel:`zsKEY_DEB_STEP`
     - :guilabel:`zsDN_KEY`
     - :guilabel:`zfKEY`
     - ``zsKey``
     - :guilabel:`zsAV_KEY`
     - :guilabel:`N.A.`
   * - ``bsIdleAcc``
     - :guilabel:`zsIDLE_DEB_STEP`
     - :guilabel:`zsDN_IDLE_ACC`
     - :guilabel:`zfIDLE_ACC`
     - ``zsIdleAcc``
     - :guilabel:`zsAV_IDLE_ACC`
     - :guilabel:`bsPULL0_AccIdleSw_ID`
   * - ``bsDoorsOpen``
     - :guilabel:`zsDOORS_OP_DEB_STEP`
     - :guilabel:`zsDN_DOORS_OPEN`
     - :guilabel:`zfDOORS_OPEN`
     - ``zsDoorsOpen``
     - :guilabel:`zsAV_DOORS_OPEN`
     - :guilabel:`bsPULL5_EngBrk_DoorSw_ID`
   * - ``bsVehSpeedLim``
     - :guilabel:`zsSPEED_LIM_DEB_STEP`
     - :guilabel:`zsDN_VEH_SPEED_LIM`
     - :guilabel:`zfVEH_SPEED_LIM`
     - ``zsVehSpeedLim``
     - :guilabel:`zsAV_VEH_SPEED_LIM`
     - :guilabel:`N.A.`
   * - ``bsClutch``
     - :guilabel:`zsCLUTCH_DEB_STEP`
     - :guilabel:`zsDN_CLUTCH`
     - :guilabel:`zfCLUTCH`
     - ``zsClutch``
     - :guilabel:`zsAV_CLUTCH`
     - :guilabel:`bsPULL4_ClutchPedSw_ID`
   * - ``bsResume``
     - :guilabel:`zsRESUME_DEB_STEP`
     - :guilabel:`zsDN_RESUME`
     - :guilabel:`zfRESUME_ON`
     - ``zsResumeOn``
     - :guilabel:`zsAV_RESUME`
     - :guilabel:`bsPULL11_DigIn1_MflvOn_ID`
   * - ``bsKeyEngStart``
     - :guilabel:`zsKEY_START_DEB_STEP`
     - :guilabel:`zsDN_KEY_ENG_START`
     - :guilabel:`zfKEY_ENG_START`
     - ``zsKeyEngStart``
     - :guilabel:`zsAV_KEY_ENG_START`
     - :guilabel:`NA`







   * - ``bsBKLRequest``
     - :guilabel:`zsBKL_REQ_DEB_STEP`
     - :guilabel:`zsDN_BKL_REQUEST`
     - :guilabel:`zfBKL_REQUEST`
     - ``zsBKLRequest``
     - :guilabel:`zsAV_BKL_REQUEST`
     - :guilabel:`bsPULL5_EngBrk_DoorSw_ID`




.. note:: Filtering the analog inputs.

    One important piece of the analog signal conditioning is the filtering. The filtering is implemented in two stages:

    1. HW :term:`RF` filter operating directly on the electric signal, these low pass filters have cutoff frequencies @ 72 Hz.

    2. Digital filter: a low pass first order filter which constant operates as the Input New Ratio Factor on the Updated Output Value. The general filter simplified function is

        .. math::
            y_n =  x * k_f + (1 - k_f) * y_{n-1}
            :label: a function

    The filter coefficients of single analog inputs are in calibration. Refer to dedicated session for indication of the calibration *symbol* of each analog input filter coefficient.

    .. warning::

        The analog inputs have its own sample time that can be 4 ms or 100 ms. The filter coefficients with same value may give different RT (Rising Time) depending on the input sample time. Refer to dedicated session for indication of the sample time of each analog input.

.. _a_list:
.. list-table:: List of any
   :widths: 50 50 50 50
   :header-rows: 1

   * - Sensor
     - Input
     - Map name
     - Output
   * - Manifold air temperature
     - zsTAirVolt
     - zvTEMPER_AIR
     - zsTAir
   * - Rail temperature
     - zsTRailVolt
     - zvTEMPER_RAIL
     - zsTRail


Engine starter - The engine crank management
############################################

Control a relay the activation of a relay by means of a low side digital output. The strategy runs @ 4 ms task.

Inputs

    *  ``ssEngState``
    *  ``zsTh2o``
    *  ``zsPotenzAcc``
    *  ``zsKey``
    *  ``zsKeyEngStart``
    *  ``zsStartAccDecCbt``
    *  ``zsStopCbt``
    *  ``bsRPM``
    *  ``csVehicleSpeed``
    *  ``vsGearCoupled``
    *  ``vsParkBrk``
    *  ``hsAvvEnable``
    *  ``vsTeApplEn``


.. note::

    For electrical connection from sensor to the ECU refer to the harness drawing, a document specific for each application.

Below examples of the HDS conversion map. Both cases, CANape and INCA maps are shown.


Air Temperature sensor
^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Air sensor
    :widths: 50 50 50 50 50
    :header-rows: 1

    * - Input
      - Map name
      - Output
      - Sample time
      - LP filter coef.
    * - ``zsTAirVolt``
      - :guilabel:`zvTEMPER_AIR`
      - ``zsTAir``
      - 100 ms
      - :guilabel:`zsKFHW_TAIR`

The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in the air temperature sensor datasheet. Pull-up resistor in HDS ECU is usually 5 kΩ.

