.. include:: <isonum.txt>
.. include:: ../_static/figures.txt
.. include:: ../_static/fcn/boost/figures.txt

Boost Pressure (Turbocharger) Control
-------------------------------------

.. sidebar:: The turbocharging
    :subtitle: Forced induction of air

    The topics has a strategic importance in engine development. The Wikipedia article `Turbocharger <https://en.wikipedia.org/wiki/Turbocharger>`_  and proposed bibliography is a good starting point for gathering technical and physical information.

    |boost_010|

Preface: the boost function strategic approach
##############################################

The boost control function in HDS9 system takes into account the air to be and / or actually intaked.

The boost pressure control strategy supplied with the HDS9 control unit has been designed taking into account the following general principles:

    #. generalization of the engine air supply system. Although the vast majority of the engines to which the HDS9 control unit is dedicated are of the turbocharged type, there is the possibility of applying the system to naturally aspirated engines.

    #. generalization of the turbocharging unit management system. There are many boost pressure management systems ranging from variable geometry systems to zero control turbochargers. As well as the pressure could be managed by a pneumatic system not controlled by the control unit.

    #. allow the exploitation of sophisticated turbocharging units allowing full integration with the management of the performance request.

Based on the above principles, the boost pressure control strategy is structured as follows:

    A. the boost pressure control strategy operates almost independently from the air quantity control.

    B. as a consequence of the previous feature, the control of the intake air takes account of the boost pressure as an *independent parameter*.

    C. boost pressure control is based on a single sensor and on a single actuator.

    D. In the implementations with complete control and with great authority, the definition of the pressure target can be very precisely linked to the performance requirement.

The boost pressure working principles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    |boost_020|

As explained in previous chapters, the boost pressure ``zsPBoost`` is determined and operates in many ways into HDS9 strategies. It is important to understand all the impacts of this parameters before to decide and operates a calibration of the impacted functions. You must know that ...

.. topic:: The Air Request Actuation

    As well known the amount of air intaked by the engine mainly depends from the throttle opening. The throttle opening depends from the measured boost pressure  ``zsPBoost``. As a result in the intake manifold a certain density of the air is established given that the intake air density mainly depends from its pressure. To decide what throttle opening must be actuated a target intake manifold air pressure is calculated from a sort of simplified speed density inverted function: the ``hsPcollReqDef``. The throttle characteristics model uses a dedicated parameter called ``hsBeta`` calculated as :math:`\tiny \frac{hsPcollReqDef}{zsPBoost}`, in other words from the ratio between the calculated target Intake manifold pressure and the measured actual boost pressure. **In this case the boost pressure is an independent parameter**. The throttle model can be generalised for any kind of working situation of engine boost pressure management configuration.

.. topic:: The target Boost Pressure

    The target Boost Pressure ``tsRifPress`` is a dependent parameter that will be explained later. By now you must only notice that it can be determined:

        A. by a complex calculation chain taking into account Air Demand, environmental conditions, diagnostic/recovery status, engine working points, etc....

        B. simply assuming as boost target the same target pressure for the air demand, as per what explained in previous paragraph, the ``hsPcollReqDef``.

    The big difference between the two (selectable with :guilabel:`tsEN_RIF_OBJ_CALC` calibration flag) modes is that

        A.  :guilabel:`tsEN_RIF_OBJ_CALC` set to False (0) in the first case the boost pressure will be controlled to a target value that will be most of the times bigger than what strictly needed by the engine to perform the expected flow rate so the requested performance. The throttle controls (limits) the flow.

        B. in the second case the intake system will be forced, by definition since :math:`\tiny hsBeta\ =\ \frac{hsPcollReqDef}{zsPBoost}` to operate at ``hsBeta`` = 1, giving mostly the condition of "minimum" usage of the throttle body limitation.

        .. warning::

            This operating mode, that we can call "de-throttled" is feasible only if the Boost pressure control has an adequate authority to give a stable, fast and repeatable setting of the ``zsPBoost`` in all working points of the engine. Evaluate the capability of the actuation chain before enable this control mode.

.. topic:: The Control of the Boost pressure

    If the turbocharger system allow the control of a regulating device, i.e. the waste gate, the control of the pressure to reach and maintain the target value can be done in different modes:

        - full open loop, by means of a feed forward table

        - full closed loop, using only the PID control

        - hybrid control using both contributions

.. warning::

    Before start reading and actuate the following chapters you must know very well the system you are going to calibrate to define preliminarily the main implementation / calibration strategy.


The Boost Pressure Control Calibration
######################################

.. note:: **What must we calibrate?**

    The turbocharger calibration on the HDS-9 control system provides to set precisely three specific parts:

        #. the definition of the **target of boost pressure**. This calibration, which has a significant impact on the performance and reliability characteristics of the engine as well as the turbocharger unit, cannot be separated from an in-depth knowledge of the characteristics of the turbocharger itself.

        #. calibrating the **operating limits** of the supercharging system. Environmental parameters such as barometric pressure and air temperature can heavily affect both the performance and the reliability of the turbocharger. At this stage, it is a question of establishing the operating limits in order to obtain the maximum degree of efficiency and safety in all conditions of the supercharging system.

        #. the calibration of the **controller** that will manage the achievement of the targets previously defined.

        #. the calibration of the **limits of the controller** to avoid unrealistic, unwanted or dangerous working mode of the turbocharger controller.

    The skills and equipment needed to calibrate the three aforementioned topics can vary significantly as well as the operating environments where the calibration is performed. For example, the limitation of the boost pressure target according to the atmospheric pressure could require a very particular type of test room that is difficult to obtain, while it can be carried out with extreme ease in a test and calibration phase on the road by driving the vehicle at a quotas where the barometric pressure assumes the limit values important for the application project. A urban bus may be tested in the town where will be exploited while a truck would be driven to mountain side at 2500 m u.sl.

.. note:: **What must we know before?**

    |boost_030|

    The fundamental characteristic of the turbocharger unit to be known is the **compressor map** in function of the flow rate and the compression ratio. Knowledge of the maximum speed of rotation of the turbocharger unit and the maximum temperature of the exhaust gas entering the turbine is not secondary. Starting the boost system calibration without knowledge of these data can lead to an inconsistent and unsafe calibration.

Boost Pressure Target Calibration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section deals with the calibration of the boost pressure target and the limits of use of the turbocharger.

.. tip::

    The target boost pressure is ``tsRifPress`` [mbarA]. It can be calculated in two main modes depending on the calibration of the selector flag  :guilabel:`tsEN_RIF_OBJ_CALC`.

Mapped Boost Pressure Target
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When  :guilabel:`tsEN_RIF_OBJ_CALC` is set to False (0) the ``tsRifStatico`` is calculated equal to ``tsRifPress``.

``tsRifPress`` is calculated as follows:

.. tip::

    The base nominal target boost pressure comes from table :guilabel:`ttRIF_BAS_EC_Q_N` f(``bsRPM``, ``hsQAir_Request``). The interpolated value ``tsRifBasEcQN`` is multiplied by the correction factor ``tsCorrTAir`` coming from interpolation on the table :guilabel:`tvCORR_TEMP_AIR` f(``zsTAir``). We obtain ``tsRifCorr`` as:

    .. math:: \small tsRifCorr\ =\ tsRifBasEcQN\ *\ tsCorrTAir

    ``tsRifCorr`` is saturated to a maximum value that can be:

    A: if :guilabel:`tsEN_LTD_SUPCALC` is set to False (0) the maximum value is calculated as:

        .. math:: \small Saturation_{high}\ =\ tsSogliaMax\ *\ tsCorrTAir

        where:

            * ``tsSogliaMax`` [mbarA] comes from interpolation on the table :guilabel:`ttSOGLIA_MAX` in f(``bsRPM`` , ``zsPAtm``).

            *  ``tsCorrTAir`` comes from interpolation on the table :guilabel:`tvCORR_TEMP_AIR` in f(``zsTAir``)

    B: if :guilabel:`tsEN_LTD_SUPCALC` is set to True (1) the maximum value is ``tsPLtdSupTheo`` calculated as:

        .. math:: \small tsPLtdSupTheo\ =\ tsRifPressMax\ *\ \frac{esRefCurveConfig}{100}

        where:

            * ``esRefCurveConfig`` [%] is the resulting limit of the Torque Limitation for engine safety (for Hot Coolant, Hot Air, Hot Exhaust).

.. _tsRifPressMax:

            * ``tsRifPressMax`` is calculated as follows:

            .. math:: \small tsRifPressMax\ =\ tsRifPresOffAirV\ +\ tsRifPresInter

            where:

                * ``tsRifPresOffAirV`` is calculated as follows:

                .. math:: \small tsRifPresOffAirV\ =\ tsAirSpeed\ *\ tsGAIN_DP_SP_AIR

                where:

                    * :guilabel:`tsGAIN_DP_SP_AIR` is an [adm] gain to be calibrated (default set to 8).

                    * ``tsAirSpeed`` is calculated as follows:

                    .. math:: \tiny tsAirSpeed\ =\ \frac{\frac{\frac{min(bsRPM,esP2EndSpeedGov)\ * esQair\_MaxAbsOut}{tsIntakeArea}}{\frac{esQair\_MaxAbsOut}{asVCylEff}}*\frac{bsNUMCYL}{2}}{60}

                    where:

                        * ``tsIntakeArea`` is calculated as follows:

                        .. math:: \tiny tsIntakeArea\ =\ {tsTHR\_DIAMETER}^2\ *\ 0.7854

                        where:

                            * :guilabel:`tsTHR_DIAMETER` [mm] is the diameter of the throttle body to be calibrated

                        * ``esQair_MaxAbsOut`` is equal to ``esQAirMaxRef`` the reference Air Charge for the maximum torque

                        * ``bsRPM`` [rpm] is the actual engine speed

                        * ``esP2EndSpeedGov`` is the actual engine speed maximum for high idle control

                        *  ``asVCylEff`` is the RAM copy of the calibration :guilabel:`asVEFF` the engine unit displacement in :math:`[cm^3]`

                        * ``bsNUMCYL`` is the number of cylinders in SW configuration (**not a callibration**)

                * ``tsRifPresInter`` is calculated as follows:

                .. math:: \tiny tsRifPresInter\ =\ tsRifPresBas\ +\ tsRifPresOffExh

                where:

                    * ``tsRifPresBas`` is calculated as follows:

                    .. math:: \tiny tsRifPresBas\ =\ [tsDensAirMaxTorq\ *\ (zsTAir\ +\ 273)]* 2.9056

                    where:

                        * ``tsDensAirMaxTorq`` is :math:`\tiny \frac{esQair\_MaxAbsOut}{asVCylEff}` which terms are aforementioned

                        * ``zsTAir`` is the actual intake Air Temperature [degC]

                    * ``tsRifPresOffExh`` is calculated as follows:

                    .. math:: \tiny tsRifPresOffExh\ =\ \frac{max(asPsca , zsPAtm)}{asRappComp}

                    where:

                        * ``asPsca`` is the exhaust manifold pressure [mbar] from calibration table :guilabel:`atPRE_SCARICO`

                        * ``zsPAtm`` is the actual Atmospheric pressure [mbar]

                        * ``asRappComp`` is the RAM copy of the calibration :guilabel:`asRAPP_COMPRES` , the geometric compression ratio of the engine.

The saturated value of ``tsRifCorr`` becomes ``tsRifStatico`` if, as already explained, :guilabel:`tsEN_RIF_OBJ_CALC` is set to False (0).

Model Based Boost Pressure Target
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In :ref:`hsBeta` section is explained the ``hsBeta`` parameter calculation. For fat remind, ``hsBeta`` parameter is one of the two entry for the throttle model used to operate the throttle opening for a certain Air Request. The boost control function plays a strategic role in the Air Request achievement. It is easy to understand how it is necessary to control this parameter in order to control how the throttle is used. Or seen from the other side how important it is to correctly control the Boost pressure in order to obtain a certain value of the HS Beta parameter and consequently obtain the desired functioning of the throttle.

To simplify and to give a very representative example of the concept we can use the principle of operation with a **Complete Throttle Opening** , many times also referred to as Wide Open Throttle and synthesized with the acronym **WOT**. From an engine point of view, this way of using the throttle allows to minimize the efficiency losses due to pumping. The WOT condition is identified by a ``hsBeta`` really at 1.

Since ``hsBeta`` is calculated as :

    .. math:: \small \ hsBeta\ =\ \frac{hsPcollReqDef}{zsPBoost}

    where:

    * ``zsPBoost`` is the measured pressure upstream the throttle valve

    * ``hsPcollReqDef`` is the estimated target pressure in intake manifold, downstream the throttle valve.

we can obtain ``hsBeta`` = 1 when the ``hsPcollReqDef`` and ``zsPBoost`` are equal. Finally we can say that in order to get ``hsBeta`` = 1 we must ensure that ``zsPBoost`` never exceed ``hsPcollReqDef``. So the target of the boost pressure must be equal to ``hsPcollReqDef``.

.. warning:: Pay attention to ``hsPcollReqDef`` accuracy.

    The parameter ``hsBeta`` is saturated to a maximum theoretical value of 1. It is not possible to establish in the intake manifold a pressure bigger than the boost one. So the ratio between instale manifold pressure and boost may be only <= 1. The  ''hsPcollReqDef'' is a calculated (not measured) pressure value. The accurate calibration of this parameter is prerequisite to go on with the boost model based pressure target approach.


.. tip::

    Setting :guilabel:`tsEN_RIF_OBJ_CALC` to True (1) the value of ``hsPcollReqDef`` becomes ``tsRifStatico``.

The boost pressure target final saturations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``tsRifPress`` is the final target of the boost pressure sent to the controller. It is based on the value of ``tsRifStatico`` saturated by following functions:

.. tip::

    * **normal model based high saturation**  ``tsRifPressMax`` (see in tsRifPressMax_) that can be fixed by means of  :guilabel:`tfRIF_PRESS_MAX` fix calibration.

    * **dsR10 Recovery active** that introduce a reduction of :guilabel:`dsOFF_RIF_PRESS` [mbar] offset (deafult calibration 300). The offset value is subtracted by ``tsRifStatico`` generating the parameter ``tsRifStaticoPR`` post recovery.

    * **normal low saturation** to the value of ``zsPAtm``. Also this saturation can be  fixed by means of  :guilabel:`tfRIF_PRESS_MIN` fix calibration.

Boost pressure controller calibration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section deals with the calibration of the boost pressure controller and the operating limits of the controller itself. The scope is to calibrate the controller in such a way that ``tsWGDuty`` parameter is calculated as a percentage of Duty Cycle of the Waste Gate actuator. ``tsWGDuty`` is based on the ``tsPercWGDirect`` parameter that is the calculation of the controller itself before the application of some operating limitation and/or adaptation that will be explained in a next chapter `Waste Gate Actuator Management Calibration`_

.. warning::

    The contents of this section can only be applied if you have previously calibrated the target boost pressure.

Boost pressure controller enable calibration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    #. ``tsEnClosedLoop`` is the general flag that indicates that the boost pressure controller is enabled. The switching is based on a hysteresis gate sensing the actual load index ``asEtasp`` against an enabling threshold ``tsSogliaSupCo`` from table :guilabel:`tvSOGLIA_SUP_CO` and a disabling one ``tsSogliaInfCo`` from table :guilabel:`tvSOGLIA_INF_CO` both of them f(``bsRPM``). The status of ``tsEnClosedLoop`` can be fixed by means of  :guilabel:`tfABILITA_CL` fix calibration.

    #. ``tsBoostCl`` is the enable of the closed loop control of the general controller. It can be fixed by means of  :guilabel:`tfEN_PB_DIRECT` fix calibration. It become True (1) if all following conditions are satisfied (AND gate):

        #. ``tsCompression`` > :guilabel:`tsEN_COMPR_INF` calibration threshold where :math:`\tiny tsCompression\ =\ \frac{tsRifPress}{zsPAtm}`

        #. ``ssEngState`` is equal to *RUN*

        #. ``tsEnClosedLoop`` is True (1). This condition can be fixed locally by means of :guilabel:`tfEN_CL_PREV` fix calibration.

Boost pressure controller Feed Forward calibration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::

    The Feed Forward (aka Open Loop) term applied by the controller is called  ``tsFfwWg`` and comes from the calibration table :guilabel:`ttFFW_WG` f(``tsCompression``, ``asQahMapSD``). It represents the model of the waste gate duty cycle actuation, given a certain speed density estimated Air Flow ``asQahMapSD`` (the one coming from the sensed intake air pressure and temperature) and a requested compression ratio ``tsCompression`` to as closest as possible to the target pressure.

    The calibration of this table strongly depends from the general calibration approach:

        - **full open loop** - the calibration must be very accurate since no other contributions will be applied. Thi approach is reliable only for stationary with limited variations of environmental parameters. Save development and calibration costs but cannot warranty the engine / system aging drift of the turbocharger performances.

        - **full closed loop** - the calibration may be done only to give a base "preload" of the turbocharger to be ready in all condition. In other words calibrate the table in order to reach a minimum safety boost pressure. I.e. , given a  ``tsCompression``, that is the target compression ratio of the compressor, the real compression ratio achieved by the turbocharging system by means of Feed Forward could be a 20% of the target. E.g. if   ``tsRifPress`` 1800 [mbar] and ``zsPAtm`` 1000 [mbar], the resulting ``tsCompression`` will be 1.8 [adm]: the calibration of the Feed Forward term in :guilabel:`ttFFW_WG` should be the one to reach ``zsPBoost`` of 1160 [mbar] that realize a real compression ratio of 1.16 [adm] (20% of target).

        - **hybrid control** - this is the most used option for heavy transient mission engine like bus applications - in this case the Feed Forward term can be set in order to reach target boost pressure saving a 5 to 10% safety marging left to the closed loop term. E.g. if   ``tsRifPress`` 1800 [mbar] and ``zsPAtm`` 1000 [mbar], the resulting ``tsCompression`` will be 1.8 [adm]: the calibration of the Feed Forward term in :guilabel:`ttFFW_WG` with a 10% safety margin should be the one to reach ``zsPBoost`` of 1620 [mbar] that realize a real compression ratio of 1.62 [adm] (90% of target).

    Depending on the approach, the Feed Forward term can be calibrated forcing the closed loop contribution to 0. This condition can be achieved setting the PID gains fixes to 0.

        * Proportional- :guilabel:`tfKP_WG_CTRL` = 0

        * Integral - :guilabel:`tfKI_WG_CTRL` = 0

        * Derivative - :guilabel:`tfKD_WG_CTRL` = 0

Boost pressure controller PID calibration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The boost pressure controller features a standard PID based on boost pressure error as input for P and I terms as well ad a derivative of the error for D term. The calibration of the PID gains can be done on tables that allow the gain scheduling feature based on the target compression ratio of compressor and the controller error.

.. tip::

    The error of the controller ``tsErrClipped`` is calculated as:

    .. math:: \small tsBoostCtrlErr\ =\ tsRifPress\ -\ zsPBoost

    where:

        * ``tsBoostCtrlErr`` is the puntual actual error calculated every 50 [ms] (by boost controller task scheduling)

        * ``tsRifPress`` is the target boost pressure

        * ``zsPBoost`` is the measured actual boost pressure

    .. warning::

        Hence the ``tsBoostCtrlErr`` is positive when the target pressure is bigger than the real (underscore status of the controller) and negative when vice versa (overboost). This kind of implementation gives  the sign of the correction to be applied to the control variable: to increase the boost pressure the closed loop contribution is positive (additive). At this status is not possible to invert the direction of the closed loop term. **In case of inverted actuation logics** , where the controlled variable must decrease to increase the boost pressure, the feature is included in next chapter `Waste Gate Actuator Management Calibration`_ that take care of the HW implementation of the controller.

    After a system saturation between the values -2000 / + 2000 [mbar], ``tsBoostCtrlErr`` becomes ``tsErrClipped``

    The derivative error ``tsDerErrPboost`` [:math:`\tiny [\frac{mbar}{s}]`] is updated every 50 [ms] based on ``tsErr`` , extrapolated to 1000 [ms] by a 20 gain and averaged in a reason of 2.

    |boost_040|

    In the picture an example of the behaviour of the boost pressure controller errors.



Waste Gate Actuator Management Calibration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``bsWGTDuty`` [%] is the last parameter of the calculation chain before to be applied to the actuation Hardware.

