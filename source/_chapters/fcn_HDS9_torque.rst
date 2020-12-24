.. include:: <isonum.txt>
.. include:: ../_static/figures.txt
.. include:: ../_static/fcn/torque/figures.txt

Torque Management
-----------------

.. sidebar:: The Torque Control
    :subtitle: Managing the torque

    The demand of the performance level of the engine must be severely controlled and accurately managed to avoid not forecasted behaviour of the vehicle.

    |torque_010|

Preface: Composing and managing the performance request
#######################################################

The performance demand that the engine must deliver to meet the needs of the vehicle is the fundamental parameter that governs the operation of the engine control and consequently the engine itself. The HDS engine control architecture is defined as "Torque based" precisely to highlight the fundamental importance of this parameter which constitutes the fundamental input for all the control strategies contained in the ECU software.

In the following chapters we often refer to a "Torque" request standing for performance demand. But aside of the torque request, the performance demand can be also a certain engine speed (e.g. for controlling the :term:`PTO` ) or a vehicle speed (e.g. for Cruise Control). So, always we should take into account a couple of performance controlling parameters:

    #. The **Request** that can be a percentage or a speed, so an amount.

    #. The **Control Mode** that can be for torque, speed or nothing, so just a mode for controlling the above mentioned requested amount.


ONE-BOX or TWO-BOXes
^^^^^^^^^^^^^^^^^^^^

Typically, the driver of a vehicle communicates to the engine the request to increase or decrease the speed as well as the rate of the  change (acceleration) by pressing the accelerator pedal. This is the most striking component of the so-called torque demand. The electrical load on the alternator, the power dissipated by the passenger compartment air conditioner compressor, the air compressor for commercial vehicle brakes are all components that add to the driver's request.

Modern commercial vehicles can be configured in two different ways with regard to the management of torque requests:

    |torque_020|

    1. All requests are communicated by means of analog or digital sensors to the engine control unit (:term:`ECM`) which takes care of composing them in the most appropriate way. In this case we define the architecture :term:`ONE-BOX`. Assuming the motor control point of view, we could say that the torque demand management is endogenous.


    |torque_030|


    2. All requests are communicated by means of analogue or digital sensors to the vehicle control unit (:term:`VCM`) which takes care of composing them in the most appropriate way. In this case we define the architecture :term:`TWO-BOX`. Still assuming the motor control point of view, we could say that the torque demand management is exogenous and will reach the ECM via a digital link in its composed and filtered form.

.. warning::

    The main discriminant for the choice of the architecture :term:`ONE-BOX` or :term:`TWO-BOX` is wwther the accelerator pedal is electrically connected to the ECM (i.e. HDS9 ECU) or not. In case the accelerator pedal is connected to the ECM please follow the calibration procedure of the torque control module according to the relevant choice of setting the control in the ONE-BOX mode. Many other configurator calibrations can set some customisation of the other torque relevant inputs (e.g. the Air Conditioning Command can come from CAN in a system basically ONE-BOX).


The long path of the torque request - the functions of torque management
########################################################################

At the end of the torque request management calculation only a unique value of torque is forwarded to the control strategies to be actuated. This value is the ``esCmuObjLimitTst`` parameter symbol [n*m] that suddenly become a Target Air Flow Quantity.

The above simple statement hides a long, complex and branched calculation chain. The normal representation of this chain as a transfer function cannot be useful. We need to use a representation like a series of interconnected calculation blocks each of which operates with the foreseen priority. The reason of such a complexity of the Torque Management calculation chain is due to many important factors which main are:

    #. **The highest safety relevant calculation** - it is mandatory to have always a perfect control of the requested / produced engine torque. The highest priority is to avoid any unwanted torque generation.

    #. **One output but many inputs** - the total torque request is composed starting from, respect to the ECM, many external and internal demands that combines following priority rules.

    #. **Engine/Vehicle operation modes** may continuously change the combination logics of the single inputs

We will call the single calculation block as a sub-function included in a (function) of the torque management strategy. Following is the cascaded approach representation of the flow of the performance requests. Cascade approach may be not the perfect visualisation of the whole strategy because all the process are concurrent. But it is also possible to visualize the request like something that start in a way from the driver and is modified along the calculation chain till it become an air quantity.

.. topic:: Torque Request - (Torque Control)

    The Torque Request Main Input where the driver / vehicle / machinery submit the Main request of performance in term of percentage of Torque. Is also the sub-function that must be configured according to the ONE-BOX or TWO-BOX architecture. The inputs can be many while the outputs are only a **Request** and a **Control Mode**.

.. topic:: Speed Request - (Speed Control)

    The Speed Request Main Input where the driver / vehicle / machinery submit the Main request of performance in term of engine speed. Is also the sub-function that must be configured according to the ONE-BOX or TWO-BOX architecture. The inputs can be many while the outputs are only a **Request** and a **Control Mode**.

.. topic:: Torque Request - (Speed Control)

    The Speed Request coming from the previous above topic must be converted in a Torque request (remember the HDS is a Torque Based Engine Control Module!). This sub-funtion includes the Speed Control strategy (PID controlling the engine speed).

.. topic:: Torque/Speed (Control Mode)

    Simultaneous requests of Torque or Speed must be arbitrated according to the control modes priority. This sub-function take care of decide what is the request that will be forwarded to the remaining part of the calculation chain.

.. topic:: Low Idle Control - (Torque Saturation)

    The low idle control strategy outputs a value of torque that must be considered as the minimum torque request that cannot be exceeded below.

.. topic:: Internal protection - (Torque Limitation)

    The value of torque is limited on the upper side by a series of torque limitation strategies for thermal, diagnostic and bad fuelling reason.

.. topic:: Speed Limitations - (Torque Limitation)

    There are two more limitations that are operating in series: the Vehicle speed limiter and the HIgh Idle Controller. The second have priority.

.. topic:: Conversion in N*m - (Engine Fit Torque Request)

    This sub-function take the percentage of performance coming from previous sub-functions and converts it to a Real Torque using the Maximum Torque Model of the engine.

.. topic:: The Air Demand - (Torque to Air)

    The sub-function convert the torque target into an air quantity target according to a calibration model.


In the following dedicated chapters all testing points will be provided as well as the calibration parameters. Some complex sub-function that needs a dedicated calibration session will be highlighted and linked to the dedicated chapters.






Torque Request - (Torque Control)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Function = (Torque Control), sub-function = Torque Request

The ONE-BOX approach
~~~~~~~~~~~~~~~~~~~~

.. tip:: **ONE-BOX or TWO-BOX choice**

    The **ONE-BOX** approach can be selected setting :guilabel:`vfTE_TSC1_VE_CTRL2_EN` calibration fix symbol to 1. Viceversa setting it to 0 the **TWO-BOX** mode is choosen.

In **ONE-BOX** mode the torque request may have different source since ONE-BOX approach is much more complex than the TWO-BOX one: it suppose that all the vehicle functions / subsystems that may require a torque demand / control are managed directly by the ECM.

**from CAN messages over the J1939 CAN BUS:**

.. _Table-1:

.. list-table:: Torque Request - Torque Control - CAN Inputs
        :widths: 15, 15,  15, 15, 15
        :header-rows: 1

        * - Request amount
          - CAN Message
          - Recovery Value
          - Fix
          - Assigned Priority
        * - ``csTorqueReqVCM_R``
          - TSC1_VE_CONTROL
          - :guilabel:`csTORQUE_REQ_REC`
          - :guilabel:`cfTORQUE_REQ_CAN`
          - ``vsPri2``
        * - ``csTorqueReqVCM2_R``
          - TSC1_VE_CONTROL2
          - :guilabel:`csTORQUE_REQ_2_REC`
          - :guilabel:`cfTORQUE_REQ_2_CAN`
          - ``vsPri3``
        * - ``csTorqueLimVCM_R``
          - TSC1_VE_LIMIT
          - :guilabel:`csTORQUE_LIM_REC`
          - :guilabel:`cfTORQUE_LIM_CAN`
          - ``vsPri1``


**or from PEDAL:**

.. list-table:: Torque Request - Torque Control - PEDAL Input
        :widths: 15, 15,  15, 15, 15
        :header-rows: 1

        * - Request amount
          - Function
          - Recovery Value
          - Fix
          - Calibrated Priority

        * - ``gsPdlPerfReq``
          - DrivePedal
          - None
          - :guilabel:`gfPDL_RUN_REQ`
          - :guilabel:`vsPDL_REQ_PRI`

.. tip:: **Request priority arbitrage and DOORS OPEN limitation**

    All the requests coming from the different channels above (only one or many concurrent) enter an arbitrage block from which exit the *Winning* one. The value of this torque request is the symbol ``vsTSC1TorqueReq``. The arbitrage block is complex and does not need other calibrations than :guilabel:`vsPDL_REQ_PRI` that set the priority of the accelerator pedal input.

    After the arbitrage block the ``vsTSC1TorqueReq`` value is limited by the :guilabel:`vsLIM_PERF_PDL_DO` calibration symbol that represent the maximum performance request allowed in case of **DOORS OPEN SAFETY ACTIVE**. The status active is monitored using the ``vsDoorsOpen`` symbol. Details of the **DOOR OPEN** safety function are in :ref:`DoorsOpen`


    After this limitation the final value of torque request is ``vsTorqueReqVCM2``.


The TWO-BOXes approach
~~~~~~~~~~~~~~~~~~~~~~

This is the case where a VCM or Body Computer in the vehicle make the arbitrage among different requests. The request taken into account is ``csTorqueReqVCM2_R`` described in table Table-1_ . The TWO BOXes mode is selected setting :guilabel:`vfTE_TSC1_VE_CTRL2_EN` to 0. After this choice node the value of torque request is ``vsTorqueReqVCM2``. In this case the torque limitation for Doors Open is performed by the VCM.

Speed Request - (Speed Control)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Function = (Torque Control), sub-function = Speed Request

The ONE-BOX approach
~~~~~~~~~~~~~~~~~~~~

.. tip:: **ONE-BOX or TWO-BOX choice**

    The **ONE-BOX** approach can be selected setting :guilabel:`vfTE_TSC1_VE_CTRL_EN` calibration fix symbol to 1. Viceversa setting it to 0 the **TWO-BOX** mode is choosen.

In **ONE-BOX** mode the speed request may have different source since ONE-BOX approach is much more complex than the TWO-BOX one: it suppose that all the vehicle functions / subsystems that may require a even mixed torque and speed demand / control are managed directly by the ECM.

**from CAN messages over the J1939 CAN BUS:**

.. _Table-2:

.. list-table:: Speed Request - Torque Control - CAN Inputs
        :widths: 15, 15,  15, 15, 15
        :header-rows: 1

        * - Request amount
          - CAN Message
          - Recovery Value
          - Fix
          - Mode
        * - ``csSpeedReqVCM_R``
          - TSC1_VE_CONTROL
          - :guilabel:`csSPEED_REQ_REC`
          - :guilabel:`cfSPEED_REQ_CAN`
          - ``csVCM_C_TSC_Mode_R``
        * - ``csSpeedReqVCM2_R``
          - TSC1_VE_CONTROL2
          - :guilabel:`csSPEED_REQ_2_REC`
          - :guilabel:`cfSPEED_REQ_2_CAN`
          - ``csVCM_C_TSC_Mode2_R``
        * - ``csSpeedLimVCM_R``
          - TSC1_VE_LIMIT
          - :guilabel:`csSPEED_LIM_REC`
          - :guilabel:`cfSPEED_LIM_CAN`
          - ``csVCM_L_TSC_Mode_R``


**or from DIGITAL INPUTS:**

.. list-table:: Speed Request - Torque Control - DIGITAL Input
        :widths: 15, 15,  15, 15
        :header-rows: 1

        * - Request amount
          - Activating Function
          - Priority
          - Fix

        * - ``vsPtoObj``
          - ``vsPtoActive``
          - 3 (lowest)
          - :guilabel:`vsRPM_SET1_PTO`

        * - ``vsCcObj``
          - ``vsCCActive``
          - 2 (medium)
          - :guilabel:`vsCCObjFix`

        * - ``vsStartObjVar``
          - ``vsStartStopReqActive``
          - 1 (highest)
          - :guilabel:`vfSTART_OBJ_VAR`

.. tip:: **Request priority arbitrage**

    **CAN Inputs**

    A complex arbitrage block takes into account the different modes and priority of the requests coming from the CAN messages. At the end the unique value of the speed request is ``vsTSC1SpeedReq`` that can be fixed by means of :guilabel:`vfTSC1_SPEED_REQ`. The winning request is indicated by ``vsSpeedReqWin`` enumerating the inputs as following:

        #. ``csSpeedLimVCM_R`` assigned to ``vsValSR1`` according to ``vsPri1`` and ``vsMod1``

        #. ``csSpeedReqVCM_R`` assigned to ``vsValSR2`` according to ``vsPri2`` and ``vsMod2``

        #. ``csSpeedReqVCM2_R`` assigned to ``vsValSR3`` according to ``vsPri3`` and ``vsMod3``

    **Digital Inputs**

    Each activating function let the relative target to be assigned to the final value of the speed target. The logic of selection if made by a cascade of if conditions where the last takes the max priority as per the above table. After the arbitrage the target value is represented by ``vsSpeedReqVCMTmp`` that become ``vsSpeedReqVCM_R``.

    If None of the digital activating conditions is active then the ``vsTSC1SpeedReq`` value pass to ``vsSpeedReqVCM_R``.

    .. note::

        The value of ``vsSpeedReqVCM_R`` is included and transmitted by the CAN message **TSC1_VE_CONTROL** by the ECU.



The TWO-BOXes approach
~~~~~~~~~~~~~~~~~~~~~~

This is the case where a VCM or Body Computer in the vehicle make the arbitrage among different requests. The request taken into account is ``csSpeedReqVCM_R`` described in table Table-2_ . The TWO BOXes mode is selected setting :guilabel:`vfTE_TSC1_VE_CTRL_EN` to 0. After this choice node the value of torque request is ``vsSpeedReqVCM``.

Torque Request - (Speed Control)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The previous chapter `Speed Request - (Speed Control)`_ refers to the definition of an engine speed target when the requested control mode is **Speed Control**.

.. sidebar:: Reminder on control modes

    **Control mode = 1** means engine speed control mode. The ECU will apply all the torque needed to keep the angine at the target engine speed.

    **Control mode = 2** means torque control mode. The ECU follows directly a torque target.

    **Control mode = 0** means no request mode, just idling.

But the ECU operates in a Torque Based engine control architecture. So always needs a Torque target to operate the engine at requested performance level. The translation of the engine speed target into a torque target is made by the **Engine Speed Control Strategy** . .. TODO: The engine speed control strategy chapter

.. tip::

    The engine speed target ``vsSpeedReqVCM`` enter the **Engine Speed Control Strategy**. This last will output a Torque request called ``esSpeed_Req_Per``. The engine speed control explanation is included in chapters dedicated to Cruise Control and PTO.

Torque/Speed Control Mode - (Torque Control)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tip::

    A subfunction called **Control Mode Manager** takes care of "mixing" the requests coming from Torque (i.e. ``esTorqueReqExt``) and Speed (i.e. ``esSpeed_Req_Per``) demands according to the inputs ``vsVCM_C_TSC_Mode2`` and ``vsVCM_C_TSC_Mode`` generating as output the ``esPercCmuReq_Tmp``.

    The subfunction also makes the check on the **Control Mode Off** when a control mode 0 is required, setting to 0 if non control is required. If this happens, the only working mode admitted is the idle.

Low Idle Control - (Torque Saturation)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The target idle speed
~~~~~~~~~~~~~~~~~~~~~

Show must go on! Rock'n'roll can never die! If not really wanted, engine must never stop! The idle control overrides all the above mentioned arbitrages among requests and mode and if nothing else is required by the users, the ECU internally control the engine speed at lowest level to make the engine "Idling".

This peculiar engine speed controller uses many models and related calibrations to provide the most stable engine speed at a desired value. Inside the torque based architecture, the internal torque request to keep the engine on, is again a torque request called ``isPID_PercIdle``.

.. tip::

    The requests for idle mode start as an engine speed target ``isRpmIdleNom`` that is possible calibrate with :guilabel:`ivRPM_IDLE_NOM` vector f(``zsTh2o``). Many corrections of the target idle-speed may be used to compensate drifts from nominal condition:

    A. ``isRpmDeltaVbat`` is an offset to compensate alternator low charging efficiency of the batteries. The value is interpolated in  :guilabel:`ivRPM_DELTA_VBAT` vector f(``zsVBPW``)

    B. :guilabel:`isOFF_RPM_GEAR` is an offset of engine speed to be applied when the gearbox move from not coupled to coupled. Used in atumatic gearbox to compensate the extra friction due to gearbox. It is activated when the ``isGearCoupled``, filtered signal based on ``csGearCoupled``, become True (1). Is possible to calibrate as filter :guilabel:`isTIME_RPM_GEAR_ON` that is a delay to compensate the gearbox mechanical delay.

    C. :guilabel:`isOFF_RPM_COND` is an offset of engine speed to be applied when the air cpnditioning system is activated. It bcome active when ``isAirCondReq`` become true.

    D. :guilabel:`isOFF_RPM_UND_BR` is the under brake offset of engine speed based ``csUnderBrake`` flag.

    The speed target value is completed with two other offsets applied in case of active Recovery **dsR21** :guilabel:`dsOFFSET_RPM` or active Recovery **dsR55** :guilabel:`isAIR_DELTA_RPM`.

    ``isRpmObtStatDiag`` is the final value after the addiction of all the offsets . This valkue is saturated with high and low limits respectively :guilabel:`ifRPM_REG_MAX`, in case not available CAN INPUT  ``csHighIdle`` and :guilabel:`ifRPM_REG_MIN`, in case not available the CAN INPUT ``csLowIdle``.

Another filter is responsible to dynamically blend with a fitting the actual speed with the target when the engine is entering or exiting from the engine idle speed mode. Finally the target to be followed and controlled by the idle governor is ``isRpmObt``.

From target speed to target torque
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This chapter is not intended for explaining the working mode of the idle controller. The idle controller is a complex control, mainly based on a PID where gains are scheduled taking into account a variety of inputs.

.. tip::

    From a general point of view is possible to monitor the activity of this controller by means of the inputs (many) and outputs (many as well) but for the scope of this chapter what is important to know is only the signal ``isPID_PercIdle`` that represents the percentage of internal torque request necessary to keep the engine working at idle.

    The ``isPID_PercIdle`` enter a Maximum function with ``esPercCmuReq_Tmp`` to generate the torque request after the idle controller node. This request is called ``esReCmuAllTst`` [%].

    .. note::

        Despite the ``esReCmuAllTst`` symbol contains the term "cmu", that we already explained means Useful Mean Torque from italian definition Coppia Media Utile, it is still a figure expressed in % (percentage) of maximum torque.

Internal protection - (Torque Limitation)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The value of the torque request is limited on the upper side by a series of torque limitation strategies for thermal, diagnostic and bad fuelling reason. The followings are the main topics and related details about the torque limitation for engine protection.

Recoveries activations for diagnosis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In case one or both the following Recovery Strategy are active, a cascade of limitations applies to the requested torque.

.. topic:: Recovery dsR29, dsI29

    The torque request is limited by means of the limit ``dsLimPerfDerOrig`` interpolated in :guilabel:`dvLIM_PERF_DERAT` f(``bsRPM``).

.. topic:: Recovery dsR53

    The recovery **dsR53** enables the application of a upper limit to the torque request ``esPercCmuIstAbs4Seen`` resulting in ``dsLimPerfDerWOff``

.. topic:: Output

    The resulting torque demand limited by the Recoveries is the symbol ``esEnReLimDiagTst``

Engine protection strategies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The engine protection functions can limit the torque request according to the following modes:

.. topic:: Engine Coolant too Hot (Engine Overheating)

    The limit ``esDeraTh2oLimDer`` comes from the coolant temperature monitoring according 2 different modes:

    **Absolute limits** - mode chosen with :guilabel:`esNEW_DERATE_TH2O` set to 0. The torque limit ``esDerateTh2o`` is interpolated on :guilabel:`etDERATE_TH2O` table f(bsRPM, zsTh2o).

    **Relative limits** - mode chosen with :guilabel:`esNEW_DERATE_TH2O` set to 1. The amount of the limit is calculated according to following function:

        .. math::
            \small Limit_{torque\_request}\ =\ 100\ +\ [ ( esOvTmpPreWar - zsTh2o) *  esKptH2OCtrl]

    where:

        * :math:`\small Limit_{torque\_request}` is the symbol ``esDerateTh20New`` [%] as the max torque request admitted

        * ``esOvTmpPreWar`` is the RAM copy of the calibration :guilabel:`esOV_TMP_PRE_WAR` , as the maximum temperature of coolant before the torque request limitation starts

        * ``zsTh2o`` is the well known measured coolant temperature

        * ``esKptH2OCtrl``  in [% / deg C] is the proportional gain that convert the amount of excess of coolant temperature to a percentage amount of reduction from 100%. The value is interpolated on table  :guilabel:`etKPTH2O_CTRL` f(``bsRPM``, :math:`\small ( esOvTmpPreWar - zsTh2o) * -1` )

    For example, if:

        * :guilabel:`esOV_TMP_PRE_WAR` is set to 97 deg C (``esOvTmpPreWar`` = 97)

        * ``zsTh2o`` measures 110 deg C

        * ``esKptH2OCtrl`` value is 1.5 [% / deg C]

    the limit ``esDerateTh20New`` will be

        .. math::
            \small esDerateTh20New\ =\ 100\ +\ [\ (\ 97\ -\ 110)\ *\  1.5 ]\ =\ 100 + (-13 * 1.5) = 100 - 19.5 = 81.5 \%

.. topic:: Intake Air too Hot (Intercooler Efficiency Failure)

    The limit ``esDeraTairLimDer`` comes from the intake air temperature monitoring according:

    **Relative limits** -  The amount of the limit is calculated according to following function:

        .. math::
            \small Limit_{torque\_request}\ =\ 100\ +\ [ ( esAIR_PRE_WARNING - zsTAir) *  esKptairCtrl]

    where:

        * :math:`\small Limit_{torque\_request}` is the symbol ``esDeraTairLimDer`` [%] as the max torque request admitted

        * :guilabel:`esAIR_PRE_WARNING` is a calibration, as the nominal maximum temperature of intake air before the torque request limitation starts. This value can be fixed locally with :guilabel:`efTAIR_CTRL_OBJ` fix (the calibration :guilabel:`esAIR_PRE_WARNING` is also used for activate the general warning on the Air Temperature).

        * ``zsTAir`` is the well known measured intake air temperature

        * ``esKptairCtrl``  in [% / deg C] is the proportional gain that convert the amount of excess of intake air temperature to a percentage amount of reduction from 100%. The value is interpolated on table  :guilabel:`etKPTAIR_CTRL` f(``bsRPM``, :math:`\small ( :guilabel:`esOV_TMP_PRE_WAR` - ``zsTAir``) * -1` )

    For example, if:

        * :guilabel:`esAIR_PRE_WARNING` is set to 67 deg C

        * ``zsTAir`` measures 85 deg C

        * ``esKptairCtrl`` value is 2.5 [% / deg C]

    the limit ``esDeraTairLimDer`` will be

        .. math::
            \small esDeraTairLimDer\ =\ 100\ +\ [\ (\ 67\ -\ 85)\ *\  2.5 ]\ =\ 100 + (-18 * 2.5) = 100 - 45 = 55 \%

.. topic:: Exhaust Gas too Hot (General Failure)

    The limit ``esDeraTexhLimDer`` comes from the exhaust Gas temperature monitoring according:

    **Relative limits** -  The amount of the limit is calculated according to following function:

        .. math::
            \small Limit_{torque\_request}\ =\ 100\ +\ [ ( esEgtMaxVal - zsTExh) *  esKpTexhCtrl]

    where:

        * :math:`\small Limit_{torque\_request}` is the symbol ``esDeraTexhLimDer`` [%] as the max torque request admitted

        * ``esEgtMaxVal`` is a temperature value, as the maximum temperature of exhaust Gas before the torque request limitation starts. This value is interpolated on  :guilabel:`evEGT_MAX` table f(``bsRPM``)

        * ``zsTExh`` is the well known measured exhaust Gas temperature

        * ``esKpTexhCtrl``  in [% / deg C] is the proportional gain that convert the amount of excess of exhaust Gas temperature to a percentage amount of reduction from 100%. The value is interpolated on table  :guilabel:`etKPTEXH_CTRL` f(``bsRPM``, :math:`\small ( ``esEgtMaxVal`` - ``zsTExh``) * -1` )

    For example, if:

        * ``esEgtMaxVal`` is 750 deg C

        * ``zsTExh`` measures 780 deg C

        * ``esKpTexhCtrl`` value is 0.5 [% / deg C]

    the limit ``esDeraTexhLimDer`` will be

        .. math::
            \small esDeraTexhLimDer\ =\ 100\ +\ [\ (\ 750\ -\ 780)\ *\  0.5 ]\ =\ 100 + (-30 * 0.5) = 100 - 15 = 85 \%


The minimum function
~~~~~~~~~~~~~~~~~~~~

    The three engine protection values calculated as per above descriptions and the active Recovery values enter a minimum function so the most limiting value is applied in case of many concurrent limitations are in place. The resulting value is ``esEngReqLimSup``.

The injector turn down ratio protection
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In some conditions the injectors may not be able to deliver the requested quantity of fuel in the nominal time. If the injection pulse time duration exceed some limits (to be found experimentally) the engine fuelling may become absolutely unstable.

.. tip::

    The amount of the torque limitation due to exceeding injection time is calculated as follows:

    .. math:: \small \text{Max Inj Time Derate Function}

        \small esDerateTinj\ =\ \frac{esMaxTinj}{qsTInj}

    where:

        * ``esDerateTinj`` is a coefficient from 0 to 1 of the actual torque request. It is applied to the ``esReCmuAllTst`` by a multiplication to generate ``esCmuPercReqDef``. It can be fixed by means of :guilabel:`efDERATE_TINJ` calibration fix.

        * ``qsTInj`` is the actual injection pulse time duration in  :math:`\tiny [\mu s]` .

        :math:`\small esMaxTinj\ =\ esT720Tinj\ *\ esAPPLICABLE_TMAX`

        where:

            :math:`\small esT720Tinj\ =\ bsDurataTDC\ *\ bsNUMCYL`

            where:

                * ``esT720Tinj`` is the time duration in :math:`\tiny [\mu s]` of a complete combustion cycle (2 engine rotations in 4 strokes engine).

                * ``bsDurataTDC`` is the time duration of a TDC to TDC interval :math:`\tiny [\mu s]`

                * ``bsNUMCYL`` is the configured number of cylinders for the actual SW version.



        * :guilabel:`esAPPLICABLE_TMAX` is a calibration factor range 0 -1 to be applied to the max physical time for inject the fuel. E.g. setting to 1 means to allow a maximum injection time over the complete combustion cycle (720 eng deg.). Setting to 0.25 means to allow a maximum injection time corresponding to 180 eng deg for all engine speeds.


Speed Limitations - (Torque Limitation)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Two more limitations that are operating in series on the engine speed (limitation and not control):

.. topic:: 1st - the Vehicle speed limiter

    When ``esCmuSpd_Ltd_Mod`` is = 1 means that the vehicle speed limiter is active. So the engine speed limit is set ``vsSpeedLimVCM_R`` if ONE-BOX mode is configured else, on TWO-BOX, the limit is ``csSpeedLimVCM_R``

.. topic:: 2nd - the HIgh Idle Controller

    If a final value of speed limitation is ``esN_Ltd_HighIdle``. This value in [rpm] is used to limit the torque by means of the **High Idle Control**.

Finally the value ``esCmuLtdVirtualT`` :math:`\tiny [N\cdot m]` is the resulting limited torque request and takes into account the vehicle speed limiter and high Idle controller. It is calculated as follows:

    .. math::

        \small esCmuLtdVirtualT\ =\ \frac{esCmu_Max_AbsOut\ *\ esHighIdleLtdDef}{100}

    Where:

        * ``esCmu_Max_AbsOut`` in :math:`\tiny [N\cdot m]` is the maximum torque output expected by the engine

        * ``esHighIdleLtdDef`` is the final output of the High Idle engine speed limitation in [%]


Conversion in N*m - (Engine Fit Torque Request)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The long path is going to end. The generalized torque request expressed in [%] of the maximum torque is going to match the physics of the engine.

.. tip::

    Finally ``esCmuPercReqDef`` [%] is used to calculate the ``esCmuObjNoLimTst`` in :math:`\tiny [N\cdot m]` as follows:

    .. math::

        \small esCmuObjNoLimTst\ =\ \frac{esCmuPercReqDef \cdot (esQAirMaxRef \cdot  esQAir2CmuConv)}{100}

    Where:

        * ``esCmuObjNoLimTst`` in :math:`\tiny [N\cdot m]` is the value of the real torque expected as output from the engine if no limitations are active.

        * ``esCmuPercReqDef`` [%] is the final percentage of the torque request

        * ``esQAirMaxRef`` in :math:`[\frac{mg}{cycle \cdot cyl}]` is the quantity of air charge needed by the engine to produce the maximum expected torque output. The value is interpolated from the calibration table :guilabel:`evQAIR_MAX_REF` f(``qsLamObtFin``).

        * ``esQAir2CmuConv`` in :math:`[\frac{N \cdot m \cdot cycle \cdot cyl}{mg}]` is the coefficient to put in scale the quantity of air charge needed by the engine to produce the maximum expected torque output. The value is interpolated from the calibration table :guilabel:`evQAIR2CMU_CONV` f(``qsLamObtFin``). It can be assumed as the efficiency of the Mixture charge.

    The application of the limitations according to what described in `Speed Limitations - (Torque Limitation)`_ , above chapter, is done by a saturation gate where ``esCmuObjNoLimTst`` is limited by the vale of ``esCmuLtdVirtualT``. This saturation generates the amount of real Torque in :math:`\tiny [N\cdot m]` to be managed by the ECM.

Epilogue - The Air Demand - (Torque to Air)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The long path reaches the end with the conversion from physical Torque Request in :math:`\tiny [N\cdot m]` to Air Flow Rate Request. In accordance with the rule that the control of the quantity of air flow is the key to control the amount of torque generated by the engine.

|torque_040|

.. tip::

    The model of the Air Request for Torque Request is made by means of a **STRATEGIC** table :guilabel:`etCMU2QAIR_OBJ`. ``esCmu2qairObj`` is interpolated on this table f(``bsRPM``, ``esCmuAirLamAdTst``.

    Since the model are never accurate enough, for the maximum accuracy of the torque management a couple of correction are available to take into account the efficiency trends (the famous "umbrella curve") in unction of the Spark Advance and actual Lambda. So the value of ``esCmuObjLimitTst`` is divided once for ``esEtaLamIst`` and again once for ``esEtaDavIst`` according to followings:

    .. math::

        \tiny
        esCmuAirLamTst = \frac{esCmuObjLimitTst}{esEtaLamIst}\\
        \text{and } \\
        esCmuAirLamAdTst = \frac{esCmuAirLamTst}{esEtaDavIst}

    Where:

        * ``esCmuAirLamTst`` is just an intermediate result of the first correction in :math:`\tiny [N\cdot m]`

        * ``esCmuAirLamAdTst`` is the real amount of actual Torque Request  in :math:`\tiny [N\cdot m]` used to interpolate the air request from the table  :guilabel:`etCMU2QAIR_OBJ`

        * ``esEtaLamIst`` is interpolated on :guilabel:`evETA_LAM_ISTA` table f(``fsLamIst``). ``fsLamIst`` is the lambda deviation from the target lambda calculated as :math:`\tiny \frac{1}{fsKO2Tmp}`

        * ``esEtaDavIst`` is interpolated on :guilabel:`evETA_D_ADV_ISTA` table f(``esDAdvIstTst``). ``esDAdvIstTst`` comes from ``jsDAdvIstTst`` taht is the difference from actual spark advance from the Optimal Spark Advance value. It apply only if the strategy of Optimal Spark Advance is used. It can be fixed by means of :guilabel:`jfADV_IST` calibration fix.