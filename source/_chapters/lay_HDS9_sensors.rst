.. include:: <isonum.txt>
.. include:: ../_static/figures.txt
.. include:: ../_static/lay-eng/figures.txt

Sensors signal conversion
-------------------------

From the engine sensors come to the ECU inputs all the analog signals which are conditioned and associated to a channel name. Then is necessary to specify the equation, table, or map that converts from analog to physical units these variables.   In the table :ref:`table_list_sensors` below there is the complete list of all sensor managed by HDS ECU with the input pre-scaled channel name, the name of map used to convert the signal and the output scaled channel name.

.. note:: Filtering the analog inputs.

    One important piece of the analog signal conditioning is the filtering. The filtering is implemented in two stages:

    1. HW :term:`RF` filter operating directly on the electric signal, these low pass filters have cutoff frequencies @ 72 Hz.

    2. Digital filter: a low pass first order filter which constant operates as the Input New Ratio Factor on the Updated Output Value. The general filter simplified function is

        .. math::
            y_n =  x * k_f + (1 - k_f) * y_{n-1}
            :label: *LP Filter* simplified function

    The filter coefficients of single analog inputs are in calibration. Refer to dedicated session for indication of the calibration *symbol* of each analog input filter coefficient.

    .. warning::

        The analog inputs have its own sample time that can be 4 ms or 100 ms. The filter coefficients with same value may give different RT (Rising Time) depending on the input sample time. Refer to dedicated session for indication of the sample time of each analog input.

.. _table_list_sensors:
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


Temperature sensors
###################

The temperature sensors are usually thermistors. Thermistor is an  electronic component whose resistance is highly dependent on temperature. Used in conjunction with a pull-up resistor (Rs), it becomes part of a potential divider circuit as shown in figure 6.5. A constant supply voltage is applied across the pull-up resistor and thermistor series circuit with the output voltage measured from across the thermistor. The voltage Vout is read by the ECU to provide temperature measurement.

|lay_040|

Two thermistors type can be utilized: NTC (Negative Temperature Coefficient) e PTC (Positive Temperature Coefficient). NTC thermistors decrease their resistive value as the operating temperature around them increases. In reverse PTC thermistors increase their resistive value as the operating temperature increases.
Wire output depends on the sensor installed, though two-wire output is commonly used.

.. note::

    For electrical connection from sensor to the ECU refer to the harness drawing, a document specific for each application.

Below examples of the HDS conversion map. Both cases, CANape and INCA maps are shown.


Air Temperature sensor
^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Oil temperature sensor
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

|lay_050_C|

|lay_050_I|

Rail temperature sensor
^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Rail temperature sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsTRailVolt``
     - :guilabel:`zvTEMPER_RAIL`
     - ``zsTRail``
     - 100 ms
     - :guilabel:`zsKFHW_TRAIL`

The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in the rail temperature sensor datasheet. Pull-up resistor in HDS ECU is usually 5 kΩ.

|lay_060_C|

|lay_060_I|

Oil temperature sensor
^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Oil temperature sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsTOilVolt``
     - :guilabel:`zvTEMPER_OIL`
     - ``zsTOil``
     - 100 ms
     - :guilabel:`zsKFHW_TOIL`


The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in the oil temperature sensor datasheet. Pull-up resistor in HDS ECU is usually 5 kΩ.

|lay_070_C|

|lay_070_I|

Boost temperature sensor
^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Boost temperature sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsTBoostVolt``
     - :guilabel:`zvTEMPER_BOOST`
     - ``zsTBoost``
     - 100 ms
     - :guilabel:`zsKFHW_TBOOST`

The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in the oil temperature sensor datasheet. Pull-up resistor in HDS ECU is usually 5 kΩ.

|lay_080_C|

|lay_080_I|

Exhaust gas temperature sensor
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Exhaust gas temperature sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsTExhVolt``
     - :guilabel:`zvTEMPER_EXH`
     - ``zsTExh``
     - 100 ms
     - :guilabel:`zsKFHW_TEXH`

The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in the oil temperature sensor datasheet. Pull-up resistor in HDS ECU is usually 5 or 1 kΩ.

|lay_090_C|

|lay_090_I|

Coolant temperature
^^^^^^^^^^^^^^^^^^^

.. list-table:: Coolant temperature sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsTh2oVolt``
     - :guilabel:`zvTEMPER_H2O`
     - ``zsTh2o``
     - 100 ms
     - :guilabel:`zsKFHW_TH2O`

The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in the coolant temperature sensor datasheet. Pull-up resistor in HDS ECU is usually 5 kΩ.

|lay_100_C|

|lay_100_I|

Catalyst temperature sensor
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: After catalyst temperature sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsTKatVolt``
     - :guilabel:`zvTEMPER_KAT`
     - ``zsTKat``
     - 100 ms
     - :guilabel:`zsKFHW_TKAT`


The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in the catalyst temperature sensor datasheet. Pull-up resistor in HDS ECU is usually 5 or 1 kΩ.

|lay_110_C|

|lay_110_I|

Air flow temperature sensor
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Air mass flow temperature sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsTMAFVolt``
     - :guilabel:`zvTEMPER_MAF`
     - ``zsTMAF``
     - 100 ms
     - :guilabel:`zsKFHW_TMAF`

The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in the oil temperature sensor datasheet. Pull-up resistor in HDS ECU is usually 5 kΩ.

|lay_120_C|

|lay_120_I|

Outlet cooler EGR temperature sensor
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: EGR cooler out temperature sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsTEgrCoolerVolt``
     - :guilabel:`zvTEMPER_EGRCOOLER`
     - ``zsTEgrCooler``
     - 4 ms
     - :guilabel:`zsKFHW_TEGRCOOLER`

The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in the EGR cooler outlet temperature sensor datasheet. Pull-up resistor in HDS ECU is usually 5 o 1kΩ tabella soto 1K.

|lay_130_C|

|lay_130_I|

Pressure sensors
################

Pressure transducers are specifically designed to provide an output signal proportional to the pressure. In automotive applications voltage output pressure transducers are normally used. These transducers have built in signal amplifiers which boost the low-level signal produced by the transducer sensing element. There are many types of voltage outputs but in automotive is preferred to use 0 to 5 volts output pressure transducers in order to take advantage of the reference voltage of the control unit. This type of sensor has usually a three-wire output (supply voltage, ground and output signal).

.. note:: For electrical connection from sensor to the ECU refer to the harness drawing, specific for each application.

Below examples of the HDS conversion map. Both cases, CANape and INCA maps are shown.


Oil pressure sensor
^^^^^^^^^^^^^^^^^^^

.. list-table:: Oil pressure sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsPOilVolt``
     - :guilabel:`zvPRESSIO_OIL`
     - ``zsPOil``
     - 100 ms
     - :guilabel:`zsKFHW_POIL`


The calibration of this map (FIGURES below) is performed according the relationship between the pressure and the voltage in the oil pressure sensor datasheet.

|lay_140_C|

|lay_140_I|

Manifold Absolute Pressure (MAP) sensor
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The MAP sensor is used to provide intake manifold pressure measurement which can be used as an engine load indicator.

.. note:: Sometimes this is also referred to as Manifold Air Pressure, however the use of the word Absolute is more descriptive as it must be appreciated that the pressure being measured is not gauge but absolute. Note that gauge pressure refers to pressure quantity above atmospheric pressure. MAP sensors are typically three wires (ground, signal and supply) and vary in their pressure measuring range depending on application. Naturally aspirated engines typically utilize 100kPa sensors while turbocharged (or supercharged) engines utilize 200kPa or 300kPa sensors.

.. list-table:: Manifold air pressure sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsMapVolt``
     - :guilabel:`zvMAP_BAR`
     - ``zsMap``
     - 4 ms
     - :guilabel:`zsKFHW_MAP`

The calibration of this map (FIGURES below) is performed according the relationship between the pressure and the voltage in manifold pressure sensor datasheet.

|lay_150_C|

|lay_150_I|

Rail pressure sensor
^^^^^^^^^^^^^^^^^^^^

.. list-table:: Injectors Rail pressure sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsPRailVolt``
     - :guilabel:`zvPRAIL_BAR`
     - ``zsPRail``
     - 4 ms
     - :guilabel:`zsKFHW_PRAIL`

The calibration of this map (FIGURES below) is performed according the relationship between the pressure and the voltage in rail pressure sensor datasheet.

|lay_160_C|

|lay_160_I|

Boost pressure sensor
^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Boost pressure sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsPBoostVolt``
     - :guilabel:`zvPRESS_BOOST`
     - ``zsPBoost``
     - 4 ms
     - :guilabel:`zsKFHW_PBOOST`

The calibration of this map (FIGURES below) is performed according the relationship between the pressure and the voltage in boost pressure sensor datasheet.

|lay_170_C|

|lay_170_I|

Tank pressure sensor
^^^^^^^^^^^^^^^^^^^^

.. list-table:: Tank pressure sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsPTankVolt``
     - :guilabel:`zvPTANK_BAR`
     - ``zsPTank``
     - 4 ms
     - :guilabel:`zsKFHW_PTANK`

The calibration of this map (FIGURES below) is performed according the relationship between the pressure and the voltage in tank pressure sensor datasheet.

|lay_180_C|

|lay_180_I|

EGR pressure sensor
^^^^^^^^^^^^^^^^^^^

.. list-table:: EGR pressure sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsEgrHighPreVolt``
     - :guilabel:`zvPRES_EGR_HP`
     - ``zsEgrHighPre``
     - 4 ms
     - :guilabel:`zsKFHW_EGR_HP`

The calibration of this map (FIGURES below) is performed according the relationship between the pressure and the voltage in EGR pressure sensor datasheet.

|lay_190_C|

|lay_190_I|

EGR flow meter differential pressure sensor
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: EGR mass flow meter sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsEgrDeltaPVolt``
     - :guilabel:`zvPRES_EGRDELTAP`
     - ``zsEgrDeltaP``
     - 4 ms
     - :guilabel:`zsKFHW_EGRDELTAP`

In applications using exhaust gas recirculation is important to know the amount of gas recirculated from the exhaust manifold to the intake manifold, for both diagnosis and control reasons.
One method to evaluate the exhaust gas recirculation flow is to use a differential pressure measured across an orifice in the exhaust flow path upstream of the EGR control valve. Then, this pressure is used by the ECU to infer the actual EGR flow.
The calibration of this map (FIGURES below) is performed according the relationship between the pressure and the voltage in differential pressure sensor datasheet.

|lay_200_C|

|lay_200_I|

EGR Valve Position
^^^^^^^^^^^^^^^^^^

.. list-table:: EGR valve position sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``dsEgrPosV``
     - none
     - ``zsEgrPos``
     - 4 ms
     - :guilabel:`zsKFHW_EGRP`

.. warning::

    The EGR position is measured directly in voltage outputted from the sensor. No linearization vs. percentage or physical parameter are available.

Lambda sensors
##############

Exist three types of oxygen sensors for combustion control: titanium-based binary oxygen sensors, zirconia-based binary oxygen sensors and wide band oxygen sensors. The most significant differences between the three sensors are the operating principle and the output signal. The working principle of titania sensor is resistive. At high temperatures, the sensing element changes its resistance in front of variations of oxygen concentration. The resistance is minimal for rich mixtures and high for lean mixtures. The sensor is fed with a constant voltage and its output response is read by the ECU by means of a voltage divider. In zirconia sensor the working principle is electrochemical and the output signal is an electric voltage. Also wide band sensor uses an electrochemical principle, but in this case the output is an electric current.

|lay_210|

.. warning::

    HDS control unit can manage two types lambda sensors: the zirconia-based binary sensors (HEGO – Heated Exhaust Gas Oxygen sensor) and the wide band sensors (UEGO – Universal Exhaust Gas Oxygen).

UEGO sensor
^^^^^^^^^^^

Broadband lambda sensors can provide a signal proportional to the residual oxygen content of the exhaust gas, allowing closed loop controls even with non-stoichiometric lambda (λ ≠ 1). This signal is represented by the pumping current used by the electronic circuit of the probe to pump the oxygen ions into the pumping cell. The current is positive for lean mixtures, negative for rich mixtures and null for the case of the stoichiometric mixture.

|lay_212|

.. warning::

    The complex management of this type os sensor, needs a dedicated component embedded in the ECU HW (:term:`ASIC`). The actual HDS configuration only allow the usage of a dedicated type of Wideband Oxygen Sensor: BOSCH LSU 4.9. This sensor is very popular and easy to be found. Other brands or same brand but different models can not be used.

.. tip::

    There are two conversion maps for this sensor, to be calibrated accordingly to its datasheet. One to convert the pumping current (Ip) value to the corresponding lambda value. The other to convert the Ip value to the corresponding oxygen concentration value. For the example see following FIGURES.

    |lay_214|

    |lay_215|

    |lay_216|

    |lay_217|

HEGO sensor before catalyst
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. sidebar:: Narrow band oxygen sensor

    Narrow band lambda sensors provide an output voltage corresponding to the quantity of oxygen in the exhaust relative to that in the atmosphere. Oxygen sensitive voltage generation ranges from 800 to 1000 millivolts for rich mixtures to as low as 100 millivolts for lean mixtures. The transition from rich to lean corresponds to 450 millivolts. This generates a response curve with a characteristic jump at λ equal to one.

    |lay_218|

.. list-table:: Lambda before Cat. sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsLambdaPreV``
     - :guilabel:`zvLAMBDA_PRE_TMP`
     - ``zsLambdaPre``
     - 4 ms
     - :guilabel:`zsKFHW_LAM_PRE`


HEGO lambda sensor before catalyst is used as a regulating sensor. The probe has an output voltage range of 0V to 1V. Therefore, to increase the precision of the measure, the signal is amplified with a gain of 4.01(or about 4) so as to reach a voltage value close to the maximum AD converter input.
The calibration of this map (FIGURE 6.5) is performed to convert the amplified lambda analogue signal in the zsLambdaPre value in mV.

|lay_218_C|

|lay_218_I|

HEGO sensor after catalyst
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Lambda after Cat. sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsLambdaPostVolt``
     - :guilabel:`zvLAMBDA_POST_TMP`
     - ``zsLambdaPost``
     - 4 ms
     - :guilabel:`zsKFHW_LAM_POST`

HEGO lambda sensor after catalyst is used as a diagnostic sensor. The probe has an output voltage range of 0V to 1V. Therefore, to increase the precision of the measure, the signal is amplified with a gain of 4.01 so as to reach a voltage value close to the maximum AD converter input.
The calibration of this map (FIGURE 6.5) is performed to convert the amplified lambda analogue signal in the zsLambdaPost value in mV.

|lay_219_C|

|lay_219_I|

.. TODO: Emulated Lambda explanation and calibration

Accelerator pedal position
##########################

.. list-table:: Accelerator pedal position sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsPotenzAccVolt``
     - :guilabel:`zvPOT_ACC_PERC`
     - ``zsPotenzAcc``
     - 4 ms
     - :guilabel:`zsKFHW_POT_ACC`

The calibration of this map (FIGURES below) is performed according to the relationship between the accelerator pedal position and the potentiometer output voltage.

|lay_220_C|

|lay_220_I|


Mass Air flow Meter
###################

.. list-table:: Air mass flow meter sensor
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsQAirDebVolt``
     - :guilabel:`zvMAF`
     - ``zsQAirDebim``
     - 4 ms
     - :guilabel:`zsKFHW_QDEB`

The calibration of this map (FIGURES below) is performed according the relationship between the mass air flow and the voltage in the air flow meter datasheet.

|lay_230_C|

|lay_230_I|

.. note:: MAF Only for development

    Although the air mass flow sensor could be used for calculating the air to be used for the amount of fuel to be injected, it would only be effective under stationary engine conditions. Due to the positioning of the sensor itself, especially in the case of engine applications with a turbocharger unit, the measurement of the quantity of air would always lag significantly compared to the other available methods (e.g. speed density and flow predictor). On the other hand, it is very useful for the proper development of the other air calculation methods as it makes the measured air flow available in the development environment to be continuously compared with the calculated quantities.

Embedded sensors
################

There are also four physical quantity that are measured directly inside the ECU, without the aid of external sensors. They are battery voltage, atmospheric pressure and ECU temperature.
Below examples of the HDS conversion map. Both cases, CANape and INCA maps are shown.

.. warning:: System level calibration!

    The following information are provided for completeness. The calibration of the characteristics of the following sensing devices is demanded to the system supplier. HDS SW will always come with default calibrations including the right calibration of the embedded sensors.

Battery voltage before the power relay (direct)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Battery voltage sensing
   :widths: 50 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
     - LP coef. @ crank
   * - ``zsVBattVolt``
     - :guilabel:`zvVBATT_PRE_PWR`
     - ``zsVBatt``
     - 4 ms
     - :guilabel:`zsKFHW_BATTV`
     - :guilabel:`zsKFHW_CRK_BATTV`

In HDS hardware configuration the AD converter maximum input voltage is 5 V. For this reason, to measure the supply voltage directly connected to the battery is necessary to scale it by means of a potential divider circuit and later scale it again via software to obtain the right physical value.
The calibration of this map (FIGURES below) is performed to convert the divider output tension in the ECU direct power supply voltage value in mV.

|lay_240_C|

|lay_240_I|

Battery voltage after the power relay
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Supply voltage sensing
   :widths: 50 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
     - LP coef. @ crank
   * - ``zsVBPWVolt``
     - :guilabel:`zvVBATT_POST_PWR`
     - ``zsVBPW``
     - 4 ms
     - :guilabel:`zsKFHW_VBPW`
     - :guilabel:`zsKFHW_CRK_VBPW`

In HDS hardware configuration the AD converter maximum input voltage is 5 V. For this reason, to measure the supply voltage connected to the power relay output is necessary to scale it by means of a potential divider circuit and later scale it again via software to obtain the right physical value.
The calibration of this map (FIGURES below) is performed to convert the divider output tension in the ECU power supply voltage value in mV.

|lay_250_C|

|lay_250_I|

Atmospheric pressure
^^^^^^^^^^^^^^^^^^^^

.. list-table:: Atmospheric pressure sensing
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsPAtmVolt``
     - :guilabel:`zvPRESSIO_ATM`
     - ``zsPAtm``
     - 100 ms
     - :guilabel:`zsKFHW_PATM`

The calibration of this map (FIGURES below) is performed according the relationship between the pressure and the voltage in onboard atmospheric pressure sensor datasheet.

|lay_260_C|

|lay_260_I|

ECU temperature
^^^^^^^^^^^^^^^

.. list-table:: ECU temperature sensing
   :widths: 50 50 50 50 50
   :header-rows: 1

   * - Input
     - Map name
     - Output
     - Sample time
     - LP filter coef.
   * - ``zsTECUVolt``
     - :guilabel:`zvTEMPER_ECU`
     - ``zsECUTemp``
     - 1000 ms
     - :guilabel:`zsKFHW_TECU`

The calibration of this map (FIGURES below) is performed according the relationship between the temperature and the voltage in onboard  temperature sensor datasheet.

|lay_270_C|

|lay_270_I|