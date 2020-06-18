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

    1. With :guilabel:`jsADV_FULL_SEL` = 1 the full model of the base SA depending from:

        a. Engine Speed ``bsRPM``

        b. Engine Load Index ``asEtasp``

        c. engine mode ``msEngMod``

        d. Coolant temperature ``zsTh2o``

        e. Intake Air temperature ``zsTAir``

        f. Lambda Target ``qsLamObtFin``

        g. The target EGR percentage ``wsPercEGRObjFin``

    2. With :guilabel:`jsADV_FULL_SEL` = 0 the simple model of the base SA depending from:

        a. Engine Speed ``bsRPM``

        b. Engine Load Index ``asEtasp``

    Depending from the calibration value in :guilabel:`jsADV_FULL_SEL` the base SA ``jsAdvBase`` is calculated usgin two different calculation chain as follows.

    **The full calculation**

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

       ``jsAdvCorrection`` = ``jsAdvCorrLamObt`` + ``jsAdvEgrCor`` + *Thermal corrections sum* ;



