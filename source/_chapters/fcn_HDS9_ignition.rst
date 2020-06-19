.. include:: <isonum.txt>
.. include:: ../_static/figures.txt
.. include:: ../_static/fcn/ignition/figures.txt

Ignition Management
-------------------

.. sidebar:: Spark
    :subtitle: Good quality combustion

    The spark advance timing is the most important control parameter for the combustion control.

    |ign_010|

Preface: the simple / complex - static / dynamic model
######################################################

Spark advance or Ignition Advance (hereinafter indicated with S.A.) timing is one of the most important parameters of the whole HDS system. Managing ignition strategy with different condition is allowed by ECU with different maps and strategies.

**The simple aspect** of the ignition advance calculation strategy is given by the fact that all the terms that contribute to the calculation of the final ignition advance are composed of a chain of sums and subtractions between them.

**The complex aspect** of the strategy is instead given by the fact that the calibration of the dependencies from the multiple parameters that affect the determination of optimal ignition advance it requires an accurate empirical characterization. This aspect can be complicated by the existence of optimization criteria that may also be in contrast with each other.

**The static aspect** of the strategy is based on the realization of the calculation model which provides for the prevalent use of tables to be interpolated with parameters that vary with the "mean values" of the engine operating point,

while **the dynamic aspect** is given by the presence in the calculation chain of contributions that can vary from TDC to TDC for each combustion.

The following section of the manual is dedicated to the stationary part of the ignition advance calculation chain. The dynamic terms that compose it are treated in the sections dedicated to the specific strategies that calculate them.

The final value of the S.A. ``jsAdv`` is calculated as:

.. math::
    SA_\mathrm{final} = {SA_\mathrm{Steady}} + {SA_\mathrm{Dynamic}}
    :label: SA general

where:

    :math:`\small{SA_\mathrm{Steady} = Filtered \  and \ Saturated (SA_\mathrm{Base})  }`

Till now everything is simple. Let's do the complex job.

The choice of the base SA model
###############################

.. note::

    The SA calculation strategy runs every TDC.

.. tip:: **The base SA model**

    The first calibration refer to the choice of the base SA model to use by means of calibration symbol :guilabel:`jsADV_FULL_SEL` :

    1. With :guilabel:`jsADV_FULL_SEL` == 1 the full model of the base SA depending from:

        a. Engine Speed ``bsRPM``

        b. Engine Load Index ``asEtasp``

        c. engine mode ``msEngMod``

        d. Coolant temperature ``zsTh2o``

        e. Intake Air temperature ``zsTAir``

        f. Lambda Target ``qsLamObtFin``

        g. The target EGR percentage ``wsPercEGRObjFin``

    2. With :guilabel:`jsADV_FULL_SEL` == 0 the simple model of the base SA depending from:

        a. Engine Speed ``bsRPM``

        b. Engine Load Index ``asEtasp``

    Depending from the calibration value in :guilabel:`jsADV_FULL_SEL` the base SA ``jsAdvBase`` is calculated usgin two different calculation chain as follows.

    **The full calculation**

    With :guilabel:`jsADV_FULL_SEL` == 1

    :math:`\small jsAdvBase = jsAdvBaseFull`

    ``jsAdvBaseFull`` is calculated as:

    * if ``msEngMod`` == 1 ( **crank** ) then ``jsAdvBaseFull`` = ``jsAdvCrk`` where ``jsAdvCrk`` is interpolated in table :guilabel:`jtADV_CRK` f( ``bsRPM`` , ``zsTh2o``);

    * if ``msEngMod`` == 2 ( **idle** ) then ``jsAdvBaseFull`` = ``jsAdvBaseIdle``

        where ``jsAdvBaseIdle`` is equal to ``jsAdvIdleAcc`` interpolated in table :guilabel:`jtADV_IDLE_ACC` f( ``bsRPM`` , ``isRpmObtDinam``) + ``jsAdvCorrection``;

    * if ``msEngMod`` == 3 ( **normal** ) then ``jsAdvBaseFull`` = ``jsAdvBaseNorm``

        where ``jsAdvBaseNorm`` is equal to ``jsAdvNormal`` interpolated in table :guilabel:`jtADV_NORMAL` f( ``bsRPM`` , ``asEtasp``) + ``jsAdvCorrection``;

    * if ``msEngMod`` == 4 ( **cutoff** ) then ``jsAdvBaseFull`` = ``jsAdvCutoff``

        where ``jsAdvCutoff`` is interpolated in table :guilabel:`jvADV_CUTOFF` f( ``bsRPM`` ;

    both ``jsAdvBaseIdle`` and ``jsAdvBaseIdle`` include the term ``jsAdvCorrection`` that is calculated as follow:

       ``jsAdvCorrection`` = ``jsAdvCorrLamObt`` + ``jsAdvEgrCor`` + *Thermal corrections sum*

        where:

        * ``jsAdvCorrLamObt`` is interpolated in table :guilabel:`jvADV_CORR_LAMOBT` f( ``qsLamObtFin`` );

        * ``jsAdvEgrCor`` is interpolated in table :guilabel:`jtADV_EGR_COR` f( ``bsRPM`` , ``wsPercEGRObjFin`` ) ;

        *  *Thermal corrections sum* is the summing node of  ``jsTairEtaspCor`` + ``jsTh2oEtaspCor`` + ``jsAdvAirRpmC``

            where:

            * ``jsTairEtaspCor`` is interpolated in table :guilabel:`jtTAIR_ETASP_COR` f( ``zsTAir`` , ``asEtasp`` ) ;

            * ``jsTh2oEtaspCor`` is interpolated in table :guilabel:`jtTH2O_ETASP_COR` f( ``zsTh2o`` , ``asEtasp`` ) ;

            * ``jsAdvAirRpmC`` is interpolated in table :guilabel:`jtTAIR_RPM_CORR` f( ``zsTAir`` , ``bsRPM`` ) **only IF** ``zsTAir`` is inside the upper and lower limits in calibration respectively :guilabel:`jsTHR_TAIR_1` and :guilabel:`jsTHR_TAIR_2` **AND** ``msEngMod`` == NORMAL (3) **OTHERWISE** is equal to 0 .


    **The simple calculation**

    With :guilabel:`jsADV_FULL_SEL` == 0

    :math:`\small jsAdvBase = jsAdvSempl`

    where ``jsAdvSempl`` is interpolated in table :guilabel:`jtADV_SEMPL_BASE` f( ``bsRPM`` , ``asEtasp``) .

The mixing of the base S.A. (smoothing)
#######################################

This part of the strategy applies **only if** :guilabel:`jsADV_FULL_SEL` == 1. Therefore following description applies to ``jsAdvBaseFull``

When engine mode ``msEngMod`` change the base value of S.A. may also change of a significative amount. Big change of S.A. may reflect in a fast variation of the torque produced by engine. A mixing function is provided by HDS to make the change of S.A. smoother.

The function is based on the trigger ``jsChgMod`` that is set to 1 when a ``msEngMod`` change is detected (by comparison within ``msEngMod`` at TDC :sub:`n` and ``msEngMod`` at TDC :sub:`n-1`).

When the trigger is risen the last value of ``jsAdvBaseFull`` is saved and ``jsAdvTDCCntTot`` counter is initialized to 0 and the evolution is in following mode.

.. tip::

    **IF** ``zsThrotPos`` **>** :guilabel:`jsTHROT_ADV_BASE` calibration symbol

        **THEN**  ``jsAdvTDCCntTot`` is set equal to :guilabel:`jsADV_TDC_CNT` calibration symbol (that means to disable the smoothing function immediately)

        **ELSE**  ``jsAdvTDCCntTot`` :sub:`n` is set to  0 and every TDC assumes the value of ``jsAdvTDCCnt``

            where ``jsAdvTDCCnt`` is the old value ``jsAdvTDCCntTot`` :sub:`n-1`   + (``jsNormIn`` **IF** ``msEngMod`` **~=** **NORMAL**   or ``jsNormOut`` **IF** ``msEngMod`` **==** **NORMAL** ) saturated up to :guilabel:`jsADV_TDC_CNT`

                where:

                * ``jsNormIn`` is interpolated in table :guilabel:`jvNORM_IN` f( ``bsRPM`` ) ;

                * ``jsNormOut`` is interpolated in table :guilabel:`jvNORM_OUT` f( ``bsRPM`` ) ;

When  ``jsAdvTDCCntTot`` reach the value of :guilabel:`jsADV_TDC_CNT` calibration symbol the smoothing function is disabled. During the smoothing period when ``jsAdvTDCCntTot`` goes from 0 to :guilabel:`jsADV_TDC_CNT`  the ``jsAdvBaseFull`` assumes the value of ``jsAdvBaseTrans``

where ``jsAdvBaseTrans`` is calculated as ``jsAdvBaseTrans`` = [ ``jsAdvBaseLast`` * ( :guilabel:`jsADV_TDC_CNT` - ``jsAdvTDCCntTot`` ) + ``jsAdvBaseTmp`` * ``jsAdvTDCCntTot`` ] / :guilabel:`jsADV_TDC_CNT` .

    where:

    * ``jsAdvBaseTmp`` is the target value at the end of the smoothing function activation. Updated every new TDC.

    * ``jsAdvBaseLast`` is the old last value of S.A. frozen before the smoothing function activation.

S.A. general filter
###################

This filter works always and make a smoothing of S.A. ``jsAdvBase`` by means of allowing a maximum variation TDC by TDC. The output of this filter is ``jsAdvBaseFilt``.

.. tip::

    * **POSITIVE STEP** . The maximum step of S.A. allowed in one TDC for positive variations is set by the calibration symbol  :guilabel:`jsMAX_DELTA_POS` .

    * **NEGATIVE STEP** . The maximum step of S.A. allowed in one TDC for negative variations is set by the calibration symbol  :guilabel:`jsMAX_DELTA_NEG` .

.. warning::

    While the filter explained in `The mixing of the base S.A. (smoothing)`_ applies only when the engine mode change (e.g. from NORMAL to IDLE or vice versa) so occurs a switch to a different calibration table, the present filter is working continuously. Verify that the two dynamic of the filters are acting properly in a scheme where the two filter are in cascade mode and the one in `The mixing of the base S.A. (smoothing)`_ is upstream the present one.

S.A. for Idle control contributes
#################################

Two different contributes can be used to control the idle engine speed with S.A. that affects the torque generation of the engine. Hereby the descriptions.

Idle control S.A. delta
^^^^^^^^^^^^^^^^^^^^^^^

The fastest torque control actuation at IDLE engine mode can be realized with the S.A. variations applied to the nominal S.A. optimized for the working point.

This module takes care of calculating a possible contribution in S.A. delta for the Low Idle control, by representing it with ``isPID_DAdvIdle`` symbol.
This value is then used summing the contributions other accessories to obtain the ``jsAdvCorr`` .
The ``isPID_DAdvIdle`` value is set to 0 (no correction) when the idle control is disabled ( ``isEnableIdleCtr`` symbol) and the engine has been in run ( ``ssEngState`` == 2).

.. tip::

    It is possible to disable this suitably the symbol :guilabel:`ifABILITA_PD_AD` fixando correction, so as not to use this correction in the strategy for calculating S.A. . Otherwise the value of the correction is calculated through the implementation of a PD control, can be represented as follows:

    .. math:: \small isPID\_Dadv = isKpDeltaAdv  + isKdDeltaAdv
        :label: isPID_DAdvIdle calculation

    Where:

    * :math:`\small isKpDeltaAdv = isKpSpdPerDATst * jsAdvBaseIdle / 100`

    * :math:`\small isKdDeltaAdv = isKdSpdPerDATst * jsAdvBaseIdle / 100`

    .. warning::

        ``jsAdvBaseIdle`` can be bypassed by :guilabel:`ifADV_REF_IDLE fix` . This operation is mandatory in case of usage of negative, null or even small values of S.A. at idle, since the usage of the normal value would disable or even strongly limits the authority of the control.
        where:

    * ``isKpSpdPerDATst`` represents the contribution of the proportional branch and is calculated as the difference between the revolutions static goal and those actually measured ( ``isRpmObtStat`` ``bsRpm`` ), saturated in upper and lower sides respectively through :guilabel:`isSAT_ERR_KP_ADV_SUP` and  :guilabel:`isSAT_ERR_KP_ADV_INF` calibration sylbols; the resulting value is multiplied by the *Kp* chosen ( :guilabel:`isKP_IDLE_DAV` ) and then saturated between -100 and 100%.

    * ``isKpSpdPerDATst`` represents the contribution of the derivative branch and is calculated as: (- ``esDer_RPM_100ms`` ) multiplied to the value of the *Kd* chosen ( :guilabel:`isKD_IDLE_DAV` )

        where:

        ``esDer_RPM_100ms`` represents the derivative of the engine speed calculated every 100ms.

``isPID_DAdvIdle`` is summed to the base S.A. ``jsAdvBaseFilt`` to obtain ``jsAdvCorr``

Torque management at idle
^^^^^^^^^^^^^^^^^^^^^^^^^

.. warning::

    This strategy of S.A. application at idle needs an accurate calibration to be performed on vehicle final context using preliminary data coming from the test bed testing. An accurate test of engine operating at idle must be done in order to determinate the torque evolution in function of the engine speed and spark advance in a relatively wide range of engine speeds and S.A. setting around the nominal idle working point.

.. tip::

    The value of :guilabel:`jsDADV_TANK_TORQU` calibration symbol enable with 1 the application of a negative S.A. delta.

    **IF** the engine mode ``msEngMod`` == IDLE (2) **AND** ``bsRPM`` **>** ``isRpmObtDinam`` the following procedure is enabled **otherwise** the value of ``jsAdvCorr`` si not modified.

    The S.A. delta is ``jsTankTorque`` interpolated in the table :guilabel:`jvTANK_TORQUE` f( ``jsEtaTankTorque`` ).

    ``jsEtaTankTorque`` is the ratio of the distance between the actual value of engine speed from the target and the engine speed variation magnitude of the S.A. variation. It is the result of following calculation:

    .. math:: \small jsEtaTankTorque = \frac {bsRPM - isRpmObtDinam} {jsRPM\_TANK\_TORQUE - isRpmObtDinam }
        :label: jsEtaTankTorque calculation

    where:

    * :guilabel:`jsRPM_TANK_TORQUE` calibration symbol [rpm] is the maximum engine speed value where the application of the delta is calculated and applied. It also affect the calculation of the ``jsEtaTankTorque`` value used to enter the :guilabel:`jvTANK_TORQUE` table.

    .. warning::

        The value into table :guilabel:`jvTANK_TORQUE` must be positive, the calculation chain subtract the value of ``jsTankTorque`` from  ``jsAdvCorr`` so the contribution results in a negative delta.

    The value of the total S.A. resulting from the operation ``jsAdvCorr`` - ``jsTankTorque`` is then saturated at a minimum value ``jsAdvMinIst`` interpolated in the table :guilabel:`jtADV_MIN_IST` f( ``bsRPM`` , ``asEtasp`` )

    The final output of this strategy is ``jsAdvCorrAdjDef`` after another S.A. i.e. ``esDelta_Adv_Req`` correction is applied .

Knocking Limit Control S.A.
###########################

.. warning:: 1

    The present part of the S.A. calculation strategy used for **STATIC control of the knocking** level of the engine must not be confused with the dynamic control of the knocking level performed by another specific strategy based on the feedback coming from knocking sensor signals.

    .. TODO: insert the link of the knocking control strategy chapter

.. warning:: 2

    The calculation chain of this S.A. contribution comes from **Torque management strategy** .

A value ``esDelta_Adv_Req`` of S.A. can be subtracted from the ``jsAdvCorr`` value taking into account a static model of the knocking phenomena.

.. tip::

    The general enable of this strategy is done with :guilabel:`esCM_ADV_ACTIVE` . **IF**  :guilabel:`esCM_ADV_ACTIVE` == 1 **THEN** ``esDelta_Adv_Req`` = ``esDeltaAdvFilt`` **otherwise** ``esDelta_Adv_Req`` = 0 .

    ``esDeltaAdvFilt`` is the filtered value of ``esDelta_Adv_Tmp`` . The filter is a LPF of first order with coefficient :guilabel:`esKFILT_DADV` .

    ``esDelta_Adv_Tmp`` is equal to the sum of  ``esEta2Dadv`` and ``esDeltaAdvKnockCtrl``

    .. math:: \small esDelta\_Adv\_Tmp =  esEta2Dadv + esDeltaAdvKnockCtrl
        :label: esDelta_Adv_Tmp calculation

    where:

    * ``esEta2Dadv`` is interpolated in the table :guilabel:`evETA2DADV` f( ``esEtaDadvReDefTs`` )

        where:

        * **IF** ``esCmu2AdvOn`` is True(1) **THEN** ``esEtaDadvReDefTs`` is a real value coming from *Torque control strategy* **OTHERWISE** ``esEtaDadvReDefTs`` = 1. The term ``esEtaDadvReDefTs`` is the torque limited for high idle ''esCmuObjLimitTst'' divided by ``esCmuDadvReqTst`` f( ``esEtaLamIst`` ). See specific chapter in Torque Model.

    * ``esDeltaAdvKnockCtrl`` is the sum of ``esKnockCtrlTasp`` + ``esKnockCtrlTq``

        .. math:: \small {esDeltaAdvKnockCtrl =  esKnockCtrlTasp + esKnockCtrlTq}
            :label: esDeltaAdvKnockCtrl calculation

        where:

        *  ``esKnockCtrlTasp`` is interpolated in the table :guilabel:`etKNOCK_CTRL_TASP` f( ``zsTAir`` , ``asEtasp`` )

        *  ``esKnockCtrlTq`` is interpolated in the table :guilabel:`etKNOCK_CTRL_THSP` f( ``zsTh2o`` , ``asEtasp`` )

The final value ``esDelta_Adv_Req`` positive is subtracted by ``jsAdvCorr`` inside tha calculation chain of the S.A. resulting in the symbol ``jsAdvTemp``.

Variable Valve Actuation term
#############################

HDS feature a control mode for engine fitted with special intake/exhaust controlled valves called *Variable Valve Actuation* (VVA).

Since it is dedicated to a very peculiar engine configuration take care of the recommendations in :ref:`recommendation_1` section about :guilabel:`efVALV_MOD_ID` and :guilabel:`esVVA_IGNT3_SHIF` calibration symbols that may affect the resulting value of  ``esDelta_Ignition`` that is provided into external dynamic terms of the S.A. calculation chain. Recommendations are given to ensure a null value of  ``esDelta_Ignition`` that gives null effect on the S.A. calculation.

Bypass and Limits
#################

The final value of S.A. is limited by two upper and lower limits in calibration. The final value can be fixed using a fix calibration symbol.

.. tip::

    * **HIGH LIMIT** . The maximum value of S.A. allowed is set by the calibration symbol  :guilabel:`jsMAX_IGN_ADV` .

    * **LOW LIMIT** . The minimum value of S.A. allowed is set by the calibration symbol  :guilabel:`jsMIN_IGN_ADV` .

    * **FIXED VALUE** . The final value of S.A. applied can be fixed by the fix calibration symbol  :guilabel:`jfADV` . It is applied to *jsAdvTmp* not available as symbol and the output is ``jsAdvTemp`` .

.. warning::

    The fix calibration symbol :guilabel:`jfADV` does not affect the correction of the S.A. cylinder by cylinder (so TDC by TDC) coming from the **Knocking detection and control** strategy. In fact the final S.A. value applied to the single cylinder at TDC :sub:`n` is given by  ``jsAdv``  = ``jsAdvTemp`` + ``ksAdvKnockCorr`` being this last the n :sup:`th` S.A. correction coming from knocking strategy.


jsAdv
#####

The really final value of applied S.A. ``jsAdv``  = ``jsAdvTemp`` + ``ksAdvKnockCorr`` jsAdv is calculated **every tDC** for the next cylinder indexed by index cylinder number ``bsNextCyl`` .