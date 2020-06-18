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

    1. **input digital status** that can be 0 or 1 depending from the voltage applied to the specific pin - the symbol is a ``bsXXXXXXXXXX`` (e.g. ``bsKeyEngStart`` for request of starter activation).

    2. **signal debouncing** take care of avoiding intermittent unwanted sequence of activation / deactivation due for example to the bouncing of an electrical contact or intermittent finger push on a button. Is a digital filtering with a constant in calibration that represents the number of counted sample at same level of the input to allow the commutation from one status to the other. The calibration can be set from 0 to 255 so realizing a debouncing for 4 ms task till of more than 1 second.

    .. warning::

        The debouncing introduce a delay from the input trigger to the output response proportional to the calibration of the number of sample. E.g. setting to 50 the debouncing filter constant for a digital input means to accept a delay of 50 * 4 = 200 ms from the instant when the button is pressed till the first reaction of the ECM.

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
     - :guilabel:`N.A.`
   * - ``bsBrakePed1``
     - :guilabel:`zsBRK_PD1_DEB_STEP`
     - :guilabel:`zsDN_BRAKE_PED1`
     - :guilabel:`zfBRAKE_PED1`
     - ``zsBrakePed1``
     - :guilabel:`zsAV_BRAKE_PED1`
     - :guilabel:`bsPULL2_BrakePedSw1_ID`
   * - ``bsBrakePed2``
     - :guilabel:`zsBRK_PD2_DEB_STEP`
     - :guilabel:`zsDN_BRAKE_PED2`
     - :guilabel:`zfBRAKE_PED2`
     - ``zsBrakePed2``
     - :guilabel:`zsAV_BRAKE_PED2`
     - :guilabel:`bsPULL3_BrakePedSw2_ID`
   * - ``bsParkBrk``
     - :guilabel:`zsPRK_BRK_DEB_STEP`
     - :guilabel:`zsDN_PARK_BRK`
     - :guilabel:`zfPARK_BRK`
     - ``zsParkBrk``
     - :guilabel:`zsAV_PARK_BRK`
     - :guilabel:`bsPULL7_ParkBrk_LimSw_ID`
   * - ``bsNIdleAcc``
     - :guilabel:`zsIDLE_DEB_STEP`
     - :guilabel:`zsDN_NIDLE_ACC`
     - :guilabel:`zfNIDLE_ACC`
     - ``zsNIdleAcc``
     - :guilabel:`zsAV_NIDLE_ACC`
     - :guilabel:`bsPULL0_AccIdleSw_ID`
   * - ``bsNIdleAcc``
     - :guilabel:`zsIDLE_DEB_STEP`
     - :guilabel:`zsDN_KICK_DOWN`
     - :guilabel:`zfKICK_DOWN`
     - ``zsKickDown``
     - :guilabel:`zsAV_KICK_DOWN`
     - :guilabel:`bsPULL0_AccIdleSw_ID`
   * - ``bsOff``
     - :guilabel:`zsOFF_DEB_STEP`
     - :guilabel:`zsDN_OFF`
     - :guilabel:`zfOFF`
     - ``zsOff``
     - :guilabel:`zsAV_OFF`
     - :guilabel:`bsPULL12_DigIn2_MflvOff_ID`
   * - ``bsSetPlus``
     - :guilabel:`zsPLUS_DEB_STEP`
     - :guilabel:`zsDN_SET_PLUS`
     - :guilabel:`zfSET_PLUS`
     - ``zsSetPlus``
     - :guilabel:`zsAV_SET_PLUS`
     - :guilabel:`bsPULL13_DigIn3_MflvP_ID`
   * - ``bsSetMinus``
     - :guilabel:`zsMINUS_DEB_STEP`
     - :guilabel:`zsDN_SET_MINUS`
     - :guilabel:`zfSET_MINUS`
     - ``zsSetMinus``
     - :guilabel:`zsAV_SET_MINUS`
     - :guilabel:`bsPULL14_DigIn4_MflvM_ID`
   * - ``bsBKLRequest``
     - :guilabel:`zsBKL_REQ_DEB_STEP`
     - :guilabel:`zsDN_BKL_REQUEST`
     - :guilabel:`zfBKL_REQUEST`
     - ``zsBKLRequest``
     - :guilabel:`zsAV_BKL_REQUEST`
     - :guilabel:`N.A.`

.. warning::

    The table above table `digital_inputs_list`_ shows the standard Dgital Input configuration. Different version of HDS SW may have different configuration of Digital Input connection to functions.


Engine starter - The engine crank management
############################################

Controls the activation of a relay by means of a low side digital output. The strategy runs @ 4 ms task. The start of the engine can be done in three different mode.

    1. Standard start - using the provided main starter button or ignition key position.

    2. Cabin tilt start button - when present the engine can be started using a dedicated button placed into the engine compartment. Useful during vehicle / engine service when the truck cabin is tilted or the engine compartment of a bus is open.

    3. Emergency start - if allowed is possible to start the engine with a simple push of the accelerator pedal.

Inputs
^^^^^^

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
    *  ``hsAvvEnable`` - is set to True (1) when (the ``hsLrnEnable`` is False (0) **OR** ``hsLrnTimeOut`` is NOT expired ( ``hsLrnTimeOut`` = ``bsCounter4ms`` >= :guilabel:`hsPON_TIME` )) **OR** the throttle self learning is terminated ( ``hsLrnStep`` = [16 OR 18 OR 20] )
    *  ``vsTeApplEn`` always True for HDS standard SW

.. note::

    For electrical connection of input/output signals/command to the ECU refer to the harness drawing, a document specific for each application.

Output
^^^^^^

    1. ``rsStarter`` - the command for the starter relay that become ``dsSTRel`` after diagnosis module and finally ``ssSTRel`` for the *Engine State Detection*
    2. ``rsEngRun``
    3. ``rsKeyEngStart``
    4. ``rsStartCrank``


Cabin Tilt Crank
^^^^^^^^^^^^^^^^

Main conditions
    The output ``rsStartCbt`` is set to True (1) if ( ``rsParkBrk`` is True **AND** ``rsSpeedNulCond`` is True(1) **AND** ``rsAckStartBut`` **AND** ``zsStopCbt`` is False(0) )

    where:

        ``rsParkBrk`` = ``vsParkBrk`` fixed by :guilabel:`rfPARK_BRK` calibration. The condition of Park Brake inserted can be fixed to True or False for this strategy context.

        ``rsSpeedNulCond`` is True if ``csVehicleSpeed`` **==** 0

        ``rsAckStartBut`` is equal to ``zsStartAccDecCbt`` debounced with a strategy context dedicated debouncing filter with time constant :guilabel:`rsTIME_START_CBT`.

        .. warning::

            This second debouncing filter is in cascade with the one of the input ``zsStartAccDecCbt``. Consider that the induced delays are summed!


Emergency Crank
^^^^^^^^^^^^^^^

Main conditions
    The output ``rsStartEmerg`` is set to True (1) if ( ``rsKeyEngStart`` is False(0) since :guilabel:`rsTIME_START_EM` **AND** ``zsPotenzAcc`` is >= :guilabel:`rsPED_ACC_EM` )

    where:

        :guilabel:`rsTIME_START_EM` is the counter threshold in 4 ms ticks with the starter button or starter key position OFF - resolution 4 ms - range 0,255

        :guilabel:`rsPED_ACC_EM` is the accelerator pedal position threshold in [%] to allow the emergency start

Standard Crank
^^^^^^^^^^^^^^

Main conditions
    The output ``rsStarterStd`` is set to True (1) if ( ``rsGearCoupledNeg`` is True(1) **AND** ``rsKeyEngStart`` is True(1) )

    where:

        ``rsGearCoupledNeg`` is equal to **NOT** ``vsGearCoupled`` fixed by :guilabel:`rfGEAR_CPL_NEG` calibration. The condition of Gear Not Coupled can be fixed to True or False for this strategy context.

        ``rsKeyEngStart`` is True(1) if ( ``zsKeyEngStart`` is True(1) **AND**  ``hsAvvEnable`` is True(1) )

Output Starter Command
^^^^^^^^^^^^^^^^^^^^^^

Main conditions
    The output ``rsStarter`` is set to True (1) if {[( ``rsStarterStd`` is True(1) **OR** ``rsStarterCbt`` is True(1) ) **OR** ``rsStartEmerg`` is True(1) ] **AND** ``zsKey`` is True(1) **AND** ``rsEngRun`` is False(0)}. ``rsStarter`` can be fixed by :guilabel:`rfSTARTER` calibration.

    where:

        ``rsEngRun`` is True(1) if ( ``bsRPM`` **\>** :guilabel:`rvENG_RUN_CRK` rpm threshold f( ``zsTh2o`` ) ) fixed by :guilabel:`rfENG_RUN` calibration. The condition of Engine Run can be fixed to True or False for this strategy context..

        the partial conditions [( ``rsStarterStd`` is True(1) **OR** ``rsStarterCbt`` is True(1) ) **OR** ``rsStartEmerg`` is True(1) ] can be fixed by :guilabel:`rfEN_START` calibration. The condition of Engine Crank Request can be fixed to True or False for this strategy context.


.. tip::

    The crucial calibration is the engine speed threshold when the engine is considered STARTED and the STARTER COMMAND must be terminated even in case the pushing on the button or the start position of the ignition key is still active.

    |veh_020|

    :guilabel:`rvENG_RUN_CRK` must be set testing on final vehicle considering the final inertia of the powertrain. The test bed dynamic may be not fitting the real one of the engine installed into vehicle.

    .. warning::

        Low values in :guilabel:`rvENG_RUN_CRK` may lead to missed start of engine. High values may lead to starter mechanical damages.





