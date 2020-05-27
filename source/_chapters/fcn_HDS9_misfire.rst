.. include:: <isonum.txt>
.. include:: ../_static/figures.txt
.. include:: ../_static/fcn/eobd_misfire/figures.txt

Misfire Diagnosis
-----------------

Preface
+++++++

Misfire diagnosis is used to detect occurrences of missed or partial combustion events to a degree such as to cause a significant instantaneous deceleration of the engine rotation speed. An algorithm that measures the TDC by TDC instant acceleration monitor the regular behaviour of the engine rotation. A Counter of misfire event drive the status indicator of malfunction as well as DTC log.

.. TODO: Integrate the function description from function book

Instrumentation setup
+++++++++++++++++++++

The instrumentation for calibrate the misfire diagnosis strategy is needed for generate arbitrary events of missed combustion.

.. warning:: **Misfires damage the catalysts**

    The main reason why OBD2 (or EOBD) requires the presence of a missed combustion diagnosis is the danger of this phenomenon for the integrity of the catalyst. The most dangerous effect of misfire is to release large quantities of ready non-combusted mixture into the engine exhaust:

    1. if the misfire is due to a lack of fuel supply, a high oxygen concentration will occur at the exhaust with consequent increase in the oxidation activity of the catalyst

    2. if, on the other hand, the misfire is caused by a failed ignition (for example, a broken spark plug), at the exhaust there will be not only a high concentration of oxygen but also a considerable quantity of fuel which, if not before, will certainly be burned as soon as it arrives at the operating site of the catalyst with consequent abnormal increase in its temperature which can lead to the melting of the monolith.

    Catalyst over-temperature may lead in best cases to missing conversion efficiency and in worst cases to dangerous vehicle overheating. Cases of vehicles set on fire by catalysts exposed to continuous misfire are not uncommon.

**Unplugged Spark Plugs**
    The most easy way to generate misfire, and also the most common fault that can lead to have real misfire on a vehicle is a spark plug not working. The disconnection of a high voltage cable from the spark plug generates a safe and clean misfire. The bad thing with this simple method is the fixed ratio of generated misfire, in the reason of:

    .. math:: Misfire_\mathrm{generated} = \frac{N_\mathrm{disconnected plugs}}{N_\mathrm{cylinders}} * 100
       :label: Generated misfire with unplugged spark plugs

    As per the following table:

    +---------------------------+---------------------------+------------------------+
    | Engine total cylinders    | Unplugged spark plugs     |   Resulting Percentage |
    +===========================+===========================+========================+
    | 4                         | 1                         |   25                   |
    |                           +---------------------------+------------------------+
    |                           | 2                         |   50                   |
    |                           +---------------------------+------------------------+
    |                           | 3                         |   75                   |
    +---------------------------+---------------------------+------------------------+
    | 6                         | 1                         |   16.6                 |
    |                           +---------------------------+------------------------+
    |                           | 2                         |   33.3                 |
    |                           +---------------------------+------------------------+
    |                           | 3                         |   50.0                 |
    |                           +---------------------------+------------------------+
    |                           | 4                         |   66.6                 |
    |                           +---------------------------+------------------------+
    |                           | 5                         |   83.3                 |
    +---------------------------+---------------------------+------------------------+
    | 8                         | 1                         |   12.5                 |
    |                           +---------------------------+------------------------+
    |                           | 2                         |   25.0                 |
    |                           +---------------------------+------------------------+
    |                           | 3                         |   37.5                 |
    |                           +---------------------------+------------------------+
    |                           | 4                         |   50.0                 |
    |                           +---------------------------+------------------------+
    |                           | 5                         |   62.5                 |
    |                           +---------------------------+------------------------+
    |                           | 6                         |   75.0                 |
    |                           +---------------------------+------------------------+
    |                           | 7                         |   87.5                 |
    +---------------------------+---------------------------+------------------------+

    It is quite evident that the resolutions of the achievable percentages is low. May be good enough for general setting of safety misfire for overtemperature monitoring in case of single(s) cylinder(s) misfire events, but is totally inadequate for detecting random misfire generating emissions over the allowed threshold.

**Disconnected injectors**
    Another easy way for generate misfire is disconnect injectors. It have the same low percentage resolution drawback of previous method. Furthermore the generated misfire may be not enough clean for cross feeding problems of engine intake (very common in diesel engine style intake manifold without runners). The generated misfire is not representative for catalyst protection since the catalyst oveheating is lees than the one in presence of missed ignition. At same time the absence of unburnt fuel make this system also not suitable for emission limits testing. Can be used for long time testing on vehicle bad road calibration validation being more safe from the firing events point of view.

**Misfire Generators**
    It the formally accepted and best solution. Generally they are systems that intercept the coils command and can generates random patterns of missed ignitions in low percentages. See SAE recommendation at `SAE Web page <https://www.sae.org/standards/content/j2901_201106/>`_. Many supplier can provide systems for 6, 8 or 12 cylinders.

    |misf_010|

    The picture above shows one of the commercially available device.

.. tip::
    The internal ECU misfire generator (part of misfire detection strategy) is not a suitable system nor for calibration or certification purposes. It can only provide a fast checking method but is not reliable for long term testing as well as is not accepted by certification authorities begin an ECU internal device that can be assimilated to a defeat device.

.. note:: **Useful CPS**
    The availability of CPS during the testing of the misfire diagnosis strategy can give a string support to confirm the validity of the misfire generation method.

Calibration
+++++++++++

The diagnosis of Misfire is divided functionally into 2 different main topics:

1.  :ref:`**DETECTION** <misfire_detection>` - Set the right parameters for:

    a. measure the crankshaft speed wheel corrections

    b. enable the detection function

    c. calculate a misfire index

    d. disable the detection when the general condition may generate false detection

2. :ref:`**MALFUNCTION TRIGGERING** <misfire_malfunction_triggering>` - Set the right parameters for trigger the malfunctions of different types:

    i. 200 Engine Revolutions statistical window set for catalyst protection

    ii. 1000 Engine Revolutions statistical window set for OBD2 / EOBD emission threshold exceeding

.. _misfire_detection:

Misfire Detection
~~~~~~~~~~~~~~~~~

Crankshaft speed wheel corrections
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before proceed with any following calibration operations, the so called "flywheel cutting errors" measure and save must be performed.

|misf_020|

The correction is measured during a number of :term:`DFSO` performed in neutral gear or with no load applied at dynamometer.

.. tip::

    :guilabel:`osNCICCOFFG` is the number of DFSO to be performed when the valid engine / vehicle conditions are verified by followings:


    ``msEngMod`` == **CUTOFF**

        `and`

    ``zsTh2o`` > :guilabel:`osSTH2ODISG`


        `and`

    ``vsVehicleSpeed`` == **0** (zero)

        `and`

    ``osNCutOffG`` < :guilabel:`osNCICCOFFG`

    If all conditions are valid the flag ``osCondAtt`` is set to True (1)


Calibrations must be set in order to fit in the best way the normal behaviour of the engine.

|misf_030|

.. tip:: The boundaries of crankshaft speed wheel corrections procedure

    In above picture, an example (not real) of one DFSO, the labels in colored frames shows the parameters to be set localized in the controlled parameters positions.

    The corrections are calculated for each engine speed cluster which number is specified by :guilabel:`osFASCRPMG`.

    :guilabel:`osTHRPMMAXG` is the maximum engine speed value including a correction cluster (yellow rectangles)

    :guilabel:`osTHRPMMING` is the minimum engine speed value including a correction cluster (yellow rectangles)

    The width of the engine rpm range of the cluster is calculated as the difference (:guilabel:`osTHRPMMAXG` -
    :guilabel:`osTHRPMMING`) divided by :guilabel:`osFASCRPMG`.


The grey curve in figure is ``bsCounterTDC`` parameter, that is a cyclical counter of all :term:`TDC` since the engine started. This counter id used to measure, like a ticks clock, the duration of the DFSO operation. For filtering abnormal FDSO, i.e. with non conventional inertia or missed fuel shut off, the duration of the DFSO must be compared with two not exceeding limits. The validity of the DFSO is determined by the following calibrations.

.. tip:: :term:`DFSO` duration validity
    Perform a DFSO with a preliminary standard acceleration in neutral gear. Usually the "High Idle" controller will let the engine speed reach the maximum value. The accelerator pedal can be operated with a slow increase for a gentle acceleration since there are no check validity for this operation. Once the engine reached the maximum speed the following deceleration must be performed by means of a fast accelerator pedal release in order to trigger in the fastest way the condition for entering the engine state in "CUT OFF" mode. The engine state is monitored with ``msEngMod``.

    Measuring ``bsCounterTDC`` in TDC base time (4 ms may be good as well but more expensive for :term:`XCP` resources) at same time of engine speed ``bsRPM`` is possible to detect the dynamic of the :term:`DFSO`.

    The number of TDC occuring when engine speed fall from the value of :guilabel:`osTHRPMMAXG` till the value in :guilabel:`osTHRPMMEDG` is compared with the calibration :guilabel:`osTHRE1CORG` . The :term:`DFSO` is valid if the number of TDC counted in the interval of engine speed fall if less then the value of :guilabel:`osTHRE1CORG`.

    The number of TDC occuring when engine speed fall from the value of :guilabel:`osTHRPMMEDG` till the value in :guilabel:`osTHRPMMING` is compared with the calibration :guilabel:`osTHRE2CORG` . The :term:`DFSO` is valid if the number of TDC counted in the interval of engine speed fall if less then the value of :guilabel:`osTHRE2CORG`.

.. note::
    It is easy to understand that the calibration :guilabel:`osTHRPMMEDG` is used to divide the deceleration into 2 distinct parts. In this way it is possible to validate the deceleration dynamics even in the presence of asymmetries that could occur, for example, of the evolution at high speed of rotation in comparison with the evolution at lower engine speeds. The variable in calibration :guilabel:`osTHRPMMEDG` can therefore be  set at the value of the change of deceleration ratio. If it were not necessary because the tendency is homogeneous between the values :guilabel:`osTHRPMMAXG` and the value :guilabel:`osTHRPMMING`, a median value between :guilabel:`osTHRPMMAXG` and :guilabel:`osTHRPMMING` can be used for the variable :guilabel:`osTHRPMMEDG` Consequently, the 2 :guilabel:`osTHRE1CORG` and :guilabel:`osTHRE2CORG` thresholds for TDC counters can be calibrated with the same number of TDC.

When the flywheel correction procedure has been completed, therefore after making the specified number of valid accelerates, the parameter ``osMedocOkG`` assumes the value one.

.. tip:: **Number of clusters**

    The number of engine RPM bands :guilabel:`osFASCRPMG` in which to divide the range between the maximum and the minimum is susceptible of the engine type. A general criterion to be applied could be to calculate a number of bands such that the single cluster includes at least 250 engine RPM point. For example for an engine on which the misfire detection strategy is expected to be used Between 1 and 2 thousand rpm, the number of bands to be specified could be 4, that is, 1000 range revolutions between 2000 and 1000 divided by 250 which is the width of the single band.

The period of flywheel portion
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The period of a flywheel angular portion is calculated, starting from single flywheel Tooth-Hole time; the most "informative contents" part of the flywheel associated signal must be selected during each expansion phase in order to maximize the signal-noise ratio.

In other words, the non-combustion event generates a significant deceleration of the flywheel. The flywheel angular portion that highlights most effectively this deceleration must be identified during this calibration phase.

.. tip:: The base flywheel portion period

|misf_040|

The calibration of the angular portion of the engine position sensor used for the basic measurement of time and then to calculate the misfire indices is the most important calibration to obtain good results from the misfire detection strategy.

.. tip::
    The 2 :guilabel:`osK1MdcNcWinStart` and :guilabel:`osK1MdcNcWinLength` parameters contribute to the definition of the aforementioned angular window as explained in the figure above.

    Since the measurement of the time that elapses for the rotation of the motor in the window defined as the start with the parameter :guilabel:`osK1MdcNcWinStart` and as the duration by the parameter :guilabel:`osK1MdcNcWinLength` is performed by the low-level software by detecting the edges of the single tooth, both quantities must be expressed numerically as multiples of the angular amplitude of a single tooth of the engine position sensor.

    **After having completed procedure** in previous chapter `Crankshaft speed wheel corrections`_ is possible to start the optimization of the flywheel portion.  Using a light graphics chart to show the ``bsRPM`` , ``zsMap``, ``osK1MdcNcg`` and ``osCMedocG`` make a **parametric determination** of the best configuration of the angular portion in 5 different working points: idle, low load @ low engine speed, high load @ low engine speed, low load @ high engine speed and high load @ high engine speed.

    .. math::
        \dot{\omega}_n * \alpha * \frac{ osK1MedocG_{n-1} - osK1MedocG_{n} }{{T_{cycle}}^3} = osCMedocG_n
        :label: *osCMedocG* calculation

    Observing ``osCMedocG`` build a robust statistical measure cylinder by cylinder using a single misfire generation every 10 * Cyl Number firing events. E.g. for a 6 cylinder engine the misfire generation pattern must be 1 event of misfire every 10 * 6 cyls. = 1 on 60 firing TDC. 60 firing TDCs happens in 20 engine revolutions. ``osCMedocG`` is calculated acceleration TDC by TDC. This parameter is optimized when:

        A. In normal firing sequences (when the misfire is absent) must be as much as possible close to zero with low spread.

        B. After misfire events must reach the highest possible value.

    Investigate both parameters weight, starting  :guilabel:`osK1MdcNcWinStart` and length :guilabel:`osK1MdcNcWinLength`. Once found the best with the first, move the second having fixed the first.

.. warning::
    **Every time the angular window change, the procedure** in previous chapter `Crankshaft speed wheel corrections`_ **must be repeated**. To reset the ECU flash memory in order to refresh the flywheel corrections proceed with following steps:

        1. update  :guilabel:`osK1MdcNcWinStart` and :guilabel:`osK1MdcNcWinLength` calibration (in working page only)

        2. with :term:`MST`, Calibration 2 page, erase the misfire data. **Operation must be dine with engine stopped**.

        3. With engine stopped and key on flash the ECU with new calibrations. At same time ECU would save the clean misfire data. Wait the normal 5 second after complete download for safe power latch save ECU's operation.

    At following ECU restart, the flywheel correction procedure will be requierd again to allow the measure of the ``osK1MdcNcg`` parameter.

.. warning::
    **Engine dynamometer torque components** - If possible avoid the

    .. math:: n  = K
       :label: *n=K* dyno mode

    control mode of engine dynamometer. In this control mode the dynamometer attempts to keep constant the engine speed at a imposed value. Since the control is made of course regulating the resistant torque, depending from the dyno control calibration, it may introduce resistant torque variation that can result in angular accelerations or decelerations not coming from engine torque generation dynamics but induced from external. These can be considered as noise. **Better to use** the mode

    .. math:: Mn^2
       :label: *Mn2* dyno mode

    |misf_050|

    In this last mode the resistant torque variation induced by the misfire event is much more similar to the real behaviour of a vehicle driveline.

The best calibration of :guilabel:`osK1MdcNcWinStart` and :guilabel:`osK1MdcNcWinLength` parameters is the one that maximize for all cylinders and all engine working points the difference of measured ``osK1MdcNcg`` with presence of misfire and normal firing.

|misf_090_B|

The above picture shows a complete procedure of testing of one engine rotation portion. It begins with the flywheel corrections procedure, then 3 engine modes are sampled. The test bed is set in Mn^2 mode and a request 25%, 75% and 0% (idle) is actuated with and without presence of Misfire Generation.

|misf_090_A|

The zoom within times 1' 45" and 2' 15" shows the different value of acceleration calculated in presence or absence of generated misfire. The target of the optimization is to get the most possible sepafration among acceleration calculated with and without misfire generation.

.. TODO: Examples of different setting of :guilabel:`osK1MdcNcWinStart` and :guilabel:`osK1MdcNcWinLength` from canape measures

Detection enabling
^^^^^^^^^^^^^^^^^^

The misfire detection is enabled when ``osMisfDetEn`` is = 1. Some calibration are needed to set the enabling conditions move from **not active** (``osMisfDetEn`` = 0) to **active** (``osMisfDetEn`` = 1) and vice versa.

.. warning::

    The calibration of enabling conditions set at test bed must be considered **Preliminary calibration**. The real calibration refinement and freezing needs test on real operation field at vehicle level.

.. tip::

    All following enabling conditions are verified with a **AND** boolean gate.

    1. **Engine working points** defined by means of ``bsRPM`` and ``asEtasp`` must be enabled setting to 1 the table :guilabel:`otENGPOINT_ENMISF`. The calibration must be done in a `cluster type` table. See example in figure below.

    |misf_100|

    2. **Air mass flow limit** in table :guilabel:`otAIRFLOW_ENMISF` a lower limit of the ``asQacMain``. The condition is True when  ``asQacMain`` is bigger than the value interpolated on the table  :guilabel:`otAIRFLOW_ENMISF` in function of ``bsRPM`` and ``zsTair``.

    3. **Engine speed range** where the condition is True if :guilabel:`osMAX_RPM_ENMISF`  > ``bsRPM`` > :guilabel:`osMIN_RPM_ENMISF`.

    4. **Engine Coolant temperature range** where the condition is True if :guilabel:`osMAX_TH2O_ENMISF`  > ``zsTh2o`` > :guilabel:`osMIN_TH2O_ENMISF`.

    5. **Throttle position steadiness** where the condition is True if the module of the difference between `zsThrotPos` at actual :term:`TDC` and `zsThrotPos` at :term:`TDC` `bsNUMCYL` before is lower than :guilabel:`osMAX_DELTATHR_ENMISF`. In other words is True if the absolute variation of `zsThrotPos` in previous `bsNUMCYL` :term:`TDC` s  is lower than :guilabel:`osMAX_DELTATHR_ENMISF` threshold.

Once all conditions are verified True (``osMisfDetEn`` == 1) the calculation of the *Cyclicity* starts.

Misfire indexes calculation - Cyclicity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Two different indexes are calculated as base of misfiring detection:

* The first index, called ``osCic_Cec`` (Half-cyclicity) compares the crankshaft slice acceleration at the TDC, with the accelerations at the previous and following TDCs.

* The second index, called ``osCic_1`` (First-cyclicity), compares the crankshaft acceleration at the TDC under consideration with the acceleration at the second TDC before it and with the second TDC hereafter.

The two different indexes are related to different misfiring patterns which need to be detected: i.e. ``osCic_Cec`` is more effective for detecting a single misfire; ``osCic_1`` is more dedicated to detection of  subsequent misfires.

.. warning:: **The effectiveness of the HDS misfire strategy on different engine layouts**

    The misfire detection strategy based on angular acceleration of the motor, such as that available in the HDS system, decreases the effectiveness with increasing engine fractionation. The reason is due to several factors:

    - the increase in the number of angularly equally spaced cylinders that insist on the same crankshaft lead to the overlapping of effects whereby a lack of combustion can produce its maximum decelaration effect after the next cylinder has already started the acceleration due to its combustion .

    - an increase in the number of cylinders is generally accompanied by an increase in inertia which tends to dampen the phenomena of angular acceleration / deceleration due to the presence or absence of combustions

    - likewise the previous one, the length of the crankshaft increasing with the number of cylinders is susceptible to torsional deformations which can generate oscillation patterns of the angular speed. These phenomena increase background noise by reducing the signal-to-noise ratio.

.. note:: The Delayed Calculation

    In general, the calculations executed by the engine control unit take place in the so-called task *real time*. Most of the injction and ignition calculation are executed at same :term:`TDC` task event when the actuation is made. This is not true for the calculation of the cyclicity that are needed for the misfire detection. In fact, having to calculate parameters that are based on cyclical events, that is to say, to cyclical speed variations of the crankshaft, the calculation module must have accelerations that occurred both in the past and those that "will" occur after the :term:`TDC` [n] under analysis. We will refer to **n** as TDC index of the "actual" TDC for which we want detect the presence or not of misfire. With **n-x** we will refer to TDC occurred x TDC events before **n** while with **n+x** we refer to TDC occurred x after n. The total length of the delay FIFO buffer is 16 elements.

    |misf_110|

.. tip:: Cyclicity calibrations

    Two cyclicities are calculated ``osCic_Cec`` and ``osCic_1``. Both of them are calculated according a general formula

    .. math::
        osCMedocG_{n-x} - {k * osCMedocG_{n} } + osCMedocG_{n+x} = osCic_{x=f(mode)}
        :label: ``osCic_Cec`` and ``osCic_1`` calculation

    where:

    *n* is the index of TDC under analysis

    *x* is the previous or following TDC distance in TDC and may vary depending on the engine configuration expressed by ``bsNUMCYL`` (RAM copy of :guilabel:`asNUMCYL` calibration)

    *k* is a gain in calibration which symbol is :guilabel:`osCYCLICITY_GAIN` that must be in the range 2 to 10.

    Scope of :guilabel:`osCYCLICITY_GAIN` calibration is the best separation within  ``osCic_Cec`` and ``osCic_1`` calculated in TDC with misfire and those same parameters calculated without misfire.

.. _misfire_malfunction_triggering:

MALFUNCTION TRIGGERING
~~~~~~~~~~~~~~~~~~~~~~

Misfire event detection: the detection threshold
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A single misfire is detected when one of the two cyclicities  ``osCic_Cec`` and ``osCic_1`` overcome the respective detection thresholds ``osSgl_Cec`` and ``osSgl_1G``.

.. tip:: The Thresholds for detecting misfire.

    The general formula is:

    .. math::
        osTthcic_{mode} * osTwamisg * osQacMain_{delayed} = osSgl_{mode}
        :label: ``osSgl_Cec`` and ``osSgl_1G`` general calculation

    where:

    *mode* can be `Cec` or `1G`

    *osTthcic* comes from interpolation on respective tables

    *osQacMain_delayed* is a delayed copy of the `asQacMain`

    The table :guilabel:`otTTHCIC_CEC` is used for setting the global factor for calculate the detection threshold to be compared with ``osCic_Cec``. The table is interpolated in function of ``bsRPM`` and ``asEtasp``.

    The table :guilabel:`otTTHCIC_1G` is used for setting the global factor for calculate the detection threshold to be compared with ``osCic_C1G``. The table is interpolated in function of ``bsRPM`` and ``asEtasp``.

    The table :guilabel:`otTWAMISG` is used for setting a common correction factor for calculate the detection threshold to be compared both with ``osCic_Cec`` and ``osCic_C1G``. The table is interpolated in function of ``zsTh2o`` and ``asEtasp``.

    Finally the :guilabel:`ofQAC_MAIN_DELAYED` fix (aka bypass) allow the application of a gain factor on the both thresholds ``osSgl_Cec`` and ``osSgl_1G``:

        **Enabled to value 1** means to disable the correction factor at all.

        **Enabled to value different from 1** means to apply a general gain to both ``osSgl_Cec`` and ``osSgl_1G``. The gain is exactly the value set in the fix_value parameter.

        **Disabled** don't care about the value means to apply a general gain which value is ``asQacMain`` to both ``osSgl_Cec`` and ``osSgl_1G``.

.. note::

    The misfire detection strategy exploits the high numerical resolution of floating point allowing the adjustment of the numerical magnitude of Cyclicity and related threshold quite freely. The calibration of the gains spread in many point of calculation allow to use big or small numbers in table and in development environment.

Filtering the cyclicity
^^^^^^^^^^^^^^^^^^^^^^^

The strategy of misfire detection allows the filtering of the mean noise using two coefficients :guilabel:`osKMSGLMDCG` and :guilabel:`osKPSGLMDCG` respectively the Lower and Upper limits of the cyclicity to allow the filtering of them. The mean noise to be removed from ``osCic_Cec`` and ``osCic_1`` before the detection comparison with the threshold is calculated as following:

1. if the ``osCic_Cec`` or ``osCic_1`` are respectively not above the values ``osSgl_Cec`` *  :guilabel:`osKPSGLMDCG` and ``osSgl_1G`` * :guilabel:`osKPSGLMDCG` **AND** not below  ``osSgl_Cec`` *  :guilabel:`osKMSGLMDCG` and ``osSgl_1G`` * :guilabel:`osKMSGLMDCG` then

2. the value of a subtractive offset is updated saturated respectively to a positive value of  ``osSgl_Cec`` and ``osSgl_1G`` multiplied by  :guilabel:`osKPSGLMDCG` and a negative value of  ``osSgl_Cec`` and ``osSgl_1G``  multiplied by  :guilabel:`osKMSGLMDCG`.

The physical meaning of this filtering operation is the removal of the symmetrical or asymmetrical noises for example due to torsional vibration or bad road vibrations transferred to crankshaft though the driveline.

.. tip::

    The preliminary calibration of :guilabel:`osKPSGLMDCG` and :guilabel:`osKMSGLMDCG` must be set to **0** until the calibration level is not ready for vehicle refinement.

    The calibration of :guilabel:`osKPSGLMDCG` and :guilabel:`osKMSGLMDCG` must be performed on the vehicle when assessing the misfire detection strategy calibration on real road. Set both of them to 0, so allowinh no filtering at all. Ensure that in a dirty road but possible a real usage one in absence of misfire events the detection of misfire would not trigger any misfire. In case it happens, try to increase with steps of 0.2 and -0.2 the two calibration :guilabel:`osKPSGLMDCG` and :guilabel:`osKMSGLMDCG` till the false detections disappear.

    After this preliminary calibration operates some real misfire and verify that on the same road, nevertheless the operating filtering, the detection works properly.

    Shortly:

    a. setting :guilabel:`osKPSGLMDCG` and :guilabel:`osKMSGLMDCG` to **0** the filtering is disabled.

    b. setting :guilabel:`osKPSGLMDCG` to **+1** and :guilabel:`osKMSGLMDCG` to **-1** the filtering is maximum allowing the filter to remove a mean value up to the thresholds.

    c. setting :guilabel:`osKPSGLMDCG` and :guilabel:`osKMSGLMDCG` to **not symmetrical values** the filtering can remove polarized offsets for example due to torsional phased resonances.

After applying the filtering the cyclicity ``osCic_Cec`` become ``osCic_CecfG``. Compare the two value to assess the filtering efficiency.

After applying the filtering the cyclicity ``osCic_1`` become ``osCic_1fG``. Compare the two value to assess the filtering efficiency.

The selection of the best cyclicity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The strategy provides many cyclicities and only one detection flag. Is possible to configure the detection using a single cyclicity, all cyclicity in a OR boolean node and even NONE, in case for example we want disable the final detection leaving the calculations running.

.. tip:: **The CYCLICITY SELECTOR**

    The flag indicating the result of the comparison operation among ``osCic_CecfG`` and ``osSgl_Cec`` is called ``osCicCecMisfDetect``.

    The flag indicating the result of the comparison operation among ``osCic_1fG`` and ``osSgl_1`` is called ``osCic1MisfDetect``.

    :guilabel:`osCIC_SELECTOR` can be set as following:

    :guilabel:`osCIC_SELECTOR` set to **1** enable the usage of the ``osCicCecMisfDetect`` **only** to detect a single misfire.

    :guilabel:`osCIC_SELECTOR` set to **2** enable the usage of the``osCic1MisfDetect`` **only** to detect a single misfire.

    :guilabel:`osCIC_SELECTOR` set to **4** enable the usage of both ``osCicCecMisfDetect`` and ``osCic1MisfDetect`` **in OR** to detect a single misfire.

    :guilabel:`osCIC_SELECTOR` set to **5** set permanently the resut of misifre detection to FALSE, in fact disabling the detection.

    The choice of the right setting for :guilabel:`osCIC_SELECTOR` must be done based on empirical assessment on vehicle refinement and depends on many factors including the engine architecture, the vahicle usage, the scope of the misfire strategy calibration, ...., etc....

The final judgment of the single event is recorded in the general e final flag ``osMisfDetect``.

Detection of the misfire failure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The single detection is the elementary part of the misfire malfunction triggering. A single :term:`TDC` can be fail or pass depending on the threshold exceeding or not. The single TDC concurs to a statistical aggregation made by means of progressive counters and finally a ratio threshold will determine the accurance of a failure or not.

From emission regulations (EOBD or OBD2) it is required to activate a **continuous** the monitor of the misfire. When the strategy is active two discrete integrating "events windows" are updated. The adjective discrete means that each windows is initialized and worked out till the end before to start a new one.

**200 Revolutions CAT DANGER Window**
    The single windows makes a statistic based on counting the misfire detected events for 200 revolutions of the engine. So the number of TDC monitored is 200 * number of cylinder / 2. 400 TDCs for 4 cyls engine, 600 for 6 cyls and 800 for 8 cyls.

    The function of this **FAST** window is to detect heavy percentage of misfire that can results in a dangerous temperature of the catalyst the may damage it.

**1000 Revolutions EMISSION LIMIT Window**
    The single windows makes a statistic based on counting the misfire detected events for 1000 revolutions of the engine. So the number of TDC monitored is 1000 * number of cylinder / 2. 2000 TDCs for 4 cyls engine, 3000 for 6 cyls and 4000 for 8 cyls.

    The function of this **SLOW** window is to detect small percentage of misfire that can not results in a dangerous temperature of the catalyst but the may increase the emission of unburnt hydrocarbon over the admitted threshold.

The counters inside the two above depicted windows are divided and specific for two different type of failures.

**Cylinder Indexed Failure**
    Every cylinders own a single counter that is incremented when a msifre event is recognized on that cylinder. The two windows by cylinder counters are called:

        ``osMisfCylCnt200`` vector of cyls. number elements counters for 200 revolution window

        ``osMisfCylCnt1000`` vector of cyls. number elements counters for 1000 revolution window

**Random Cylinder Failure**
    Every misfire detected event nevertheless the cylinder where occurs is integrated in a global counter that is called "Random" cylinder counter. The two random counters are:

        ``osMisfRndCnt200`` a single scalar counter integrating all detected msifre events happened despite of the cylinder in the 200 revolutions window.

        ``osMisfRndCnt1000`` a single scalar counter integrating all detected msifre events happened despite of the cylinder in the 1000 revolutions window.


.. note:: The thresholds for the counters. Detecting the failure.

    As already said, the misfire detection strategy id a diagnosis. The ultimate and general scope of a diagnosis function is to advise the driver / operator that something wrong is going on with the engine combustion. The most used system to give the signal is to light on a lamp. Often also to buzz some alert sound when the status of a lamp change.  Refer to the specific chapter of the manual :term:`WIP`.

    .. TODO: to update the link to the lamp type and mode calibration for a generic diagnosis, when the module of the manual will be ready

Interpolation on the thresholds table
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Since the counters of the statistical validation of the misfire failure works on discrete events windows of 200 and 1000 revolutions, at same time of counting the "misfire detected" event ratios against the "total events" the average engine working point indexes are calculated. The working point indexes are ``bsRPM`` and ``asEtasp``.

``osMeanRPM200`` and ``osMeanEtasp200`` are the average working point indexes for 200 revolutions integrating window.

``osMeanRPM1000`` and ``osMeanEtasp1000`` are the average working point indexes for 1000 revolutions integrating window.

.. tip::

    The tables used for calibrate the threshold are:

    200 revolutions window for single cylinder :guilabel:`otTHR_CYL_MISF200` interpolated with ``osMeanRPM200`` and ``osMeanEtasp200``;

    200 revolutions window for random cylinder :guilabel:`otTHR_RND_MISF200` interpolated with ``osMeanRPM200`` and ``osMeanEtasp200``;

    1000 revolutions window for single cylinder :guilabel:`otTHR_CYL_MISF1000` interpolated with ``osMeanRPM1000`` and ``osMeanEtasp1000``;

    1000 revolutions window for random cylinder :guilabel:`otTHR_RND_MISF1000` interpolated with ``osMeanRPM1000`` and ``osMeanEtasp1000``.

    The number to be placed into each cluster of the table is the **number of maximum misfire detected events** allowed in the **total number of events** sampled by the specific counting window.

    For example, considering a 8 cylinders engine, the following table shows how to calculate ratios on specific counting window:

    .. table:: 8 Cylinders engine - Misfire threshold ratio examples

        +------------+------------+--------------+---------------+---------------+---------------+---------------+
        |  Window    | counter    | TOTAL EVENTS | 1% Threshold  | 10% Threshold | 25% Threshold | 50% Threshold |
        +============+============+==============+===============+===============+===============+===============+
        |  200 REVs  | Single CYL |          100 |            1  |           10  |            25 |           50  |
        +------------+------------+--------------+---------------+---------------+---------------+---------------+
        |  200 REVs  | Random     |          800 |            8  |           80  |           200 |          400  |
        +------------+------------+--------------+---------------+---------------+---------------+---------------+
        | 1000 REVs  | Single CYL |          500 |            5  |           50  |           125 |          250  |
        +------------+------------+--------------+---------------+---------------+---------------+---------------+
        | 1000 REVs  | Random     |         4000 |           40  |          400  |          1000 |         2000  |
        +------------+------------+--------------+---------------+---------------+---------------+---------------+

    While for a 6 cylinder engine the situation is as following table

     .. table:: 6 Cylinders engine - Misfire threshold ratio examples

        +------------+------------+--------------+---------------+---------------+---------------+---------------+
        |  Window    | counter    | TOTAL EVENTS | 1% Threshold  | 10% Threshold | 25% Threshold | 50% Threshold |
        +============+============+==============+===============+===============+===============+===============+
        |  200 REVs  | Single CYL |          100 |            1  |           10  |            25 |           50  |
        +------------+------------+--------------+---------------+---------------+---------------+---------------+
        |  200 REVs  | Random     |          600 |            6  |           60  |           150 |          300  |
        +------------+------------+--------------+---------------+---------------+---------------+---------------+
        | 1000 REVs  | Single CYL |          500 |            5  |           50  |           125 |          250  |
        +------------+------------+--------------+---------------+---------------+---------------+---------------+
        | 1000 REVs  | Random     |         3000 |           30  |          300  |           750 |         1500  |
        +------------+------------+--------------+---------------+---------------+---------------+---------------+

Every TDC the specific counter is compared with the specific threshold. This last is interpolated each TDC on the table based on the updated mean working point indexes. The fault is detected even during the evolution of the single window. The reset of the counter id cone at the end of a integrating windows. The fault is removed when a windows is completed with misfire detected event resulting less than the threshold.

Enabling the diagnosis for each elements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Finally, in order to have the diagnosis Code Managed according to the general setting of every diagnosis, the general enable for the single element to be monitored must be individually enabled as follow, assuming that the minimum number of cylinders managed by HDS system is 4:

.. tip::

    For each cylinder ( 1 to 8 ) 

    :guilabel:`dsMD1_AUTOD` must be set to 1 for enable the diagnosis or to 0 if you want disable for this cylinder.

    :guilabel:`dsMD2_AUTOD` must be set to 1 for enable the diagnosis or to 0 if you want disable for this cylinder.

    :guilabel:`dsMD3_AUTOD` must be set to 1 for enable the diagnosis or to 0 if you want disable for this cylinder.

    :guilabel:`dsMD4_AUTOD` must be set to 1 for enable the diagnosis or to 0 if you want disable for this cylinder.

    :guilabel:`dsMD5_AUTOD` must be set to 1 for enable the diagnosis or to 0 if this cylinder number is not available or you want disable for this cylinder.

    :guilabel:`dsMD6_AUTOD` must be set to 1 for enable the diagnosis or to 0 if this cylinder number is not available or you want disable for this cylinder.

    :guilabel:`dsMD7_AUTOD` must be set to 1 for enable the diagnosis or to 0 if this cylinder number is not available or you want disable for this cylinder.

    :guilabel:`dsMD8_AUTOD` must be set to 1 for enable the diagnosis or to 0 if this cylinder number is not available or you want disable for this cylinder.

    For Random Cylinder Misfire

    :guilabel:`dsMDR_AUTOD` enable the random cylinder :term:`DTC` detection.
