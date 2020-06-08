.. include:: <isonum.txt>
.. include:: ../_static/figures.txt
.. include:: ../_static/lay-eng/figures.txt

Sensors signal conversion
-------------------------

From the engine sensors come to the ECU inputs all the analog signals which are conditioned and associated to a channel name. Then is necessary to specify the equation, table, or map that converts from analog to physical units these variables.   In the FIGURE 6.1 there is the complete list of all sensor managed by HDS ECU with the input pre-scaled channel name, the name of map used to convert the signal and the output scaled channel name.

.. list-table:: List of sensors
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
   * - Oil pressure
     - zsPOilVolt
     - zvPRESSIO_OIL
     - zsPOil
   * - Oil temperature
     - zsTOilVolt
     - zvTEMPER_OIL
     - zsTOil
   * - Boost temperature
     - zsTBoostVolt
     - zvTEMPER_BOOST
     - zsTBoost
   * - Exhaust gas temperature
     - zsTExhVolt
     - zvTEMPER_EXH
     - zsTExh
   * - Coolant temperature
     - zsTh2oVolt
     - zvTEMPER_H2O
     - zsTh2o
   * - After catalyst temperature
     - zsTKatVolt
     - zvTEMPER_KAT
     - zsTKat
   * - Air mass flow temperature
     - zsTMAFVolt
     - zvTEMPER_MAF
     - zsTMAF
   * - Manifold air pressure
     - zsMapVolt
     - zvMAP_BAR
     - zsMap
   * - Rail pressure
     - zsPRailVolt
     - zvPRAIL_BAR
     - zsPRail
   * - Boost pressure
     - zsPBoostVolt
     - zvPRESS_BOOST
     - zsPBoost
   * - Tank pressure
     - zsPTankVolt
     - zvPTANK_BAR
     - zsPTank
   * - UEGO
     - zsUegoIpFilt
     - zvIP_LAM_CONV, zvUEGO_IPCURR_TO_O2CONC
     - zsUegoLambda, zsUegoO2Conc
   * - Lambda before Cat.
     - zsLambdaPreV
     - zvLAMBDA_PRE_TMP
     - zsLambdaPre
   * - Lambda after Cat.
     - zsLambdaPostVolt
     - zvLAMBDA_POST_TMP
     - zsLambdaPost
   * - Accelerator pedal
     - zsPotenzAccVolt
     - zvPOT_ACC_PERC
     - zsPotenzAcc
   * - Air mass flow meter
     - zsQAirDebVolt
     - zvMAF
     - zsQAirDebim
   * - EGR cooler out temperature
     - zsTEgrCoolerVolt
     - zvTEMPER_EGRCOOLER
     - zsTEgrCooler
   * - EGR pressure
     - zsEgrHighPreVolt
     - zvPRES_EGR_HP
     - zsEgrHighPre
   * - EGR mass flow meter
     - zsEgrDeltaPVolt
     - zvPRES_EGRDELTAP
     - zsEgrDeltaP
   * - Battery voltage
     - zsVBattVolt
     - zvVBATT_PRE_PWR
     - zsVBatt
   * - Supply voltage
     - zsVBPWVolt
     - zvVBATT_POST_PWR
     - zsVBPW
   * - Atmospheric pressure
     - zsPAtmVolt
     - zvPRESSIO_ATM
     - zsPAtm
   * - ECU temperature
     - zsTECUVolt
     - zvTEMPER_ECU
     - zsECUTemp

.. tip::

    In order to calibrate each sensor, the name of each relative map can be found out by using the Symbol Explorer in Vector CANape environment (as in the example of FIGURE) or using Variable selection in INCA environment (as in the example of INCA's Figures).

    |lay_010|

    This figure shows Vector Canape Calibration tool

    |lay_020|

    |lay_030|

    These figures show ETAS Inca Canape Calibration tool



