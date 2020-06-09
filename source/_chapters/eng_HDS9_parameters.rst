.. include:: <isonum.txt>
.. include:: ../_static/figures.txt
.. include:: ../_static/lay-eng/figures.txt

Engine parameters setting
-------------------------

Before starting the engine, it’s necessary to set some characteristic engine parameters included in the following list:

* :guilabel:`asQAIR_TDC_REF`

    single cylinder displacement reference [mg for each cylinder stroke]. Is used for calculate the load index ``asEtasp`` as ratio with the ``asQacMain`` :

        .. math::
            asEtasp =  \frac {asQacMain} {asQairTdcRef}
            :label: *asEtasp* function

        where :math:`asQairTdcRef = asQAIR\_TDC\_REF`


* :guilabel:`asF1_F2`

    duration of intake phase [degrees]. 

    .. tip:: Refer to engine camshaft phasing diagram or measure the real opening timing directly on the engine.

        |eng_010|

* :guilabel:`asRAPP_COMPRES`: compression ratio (CR)

    .. tip:: Reminder for the CR calculation

        |eng_020|

* :guilabel:`asCORR_VEFF`: correction factor for dead volume. This calibration must be empirically determined when calibrating the air quantities.

* :guilabel:`asVEFF`: unit displacement [m3]. Referring to above picture is identified with volume V\ :sub:`1`

* :guilabel:`asVC`: dead volume [m3]. Referring to above picture is identified with volume V\ :sub:`2`

* :guilabel:`hfVCYL_RIF`: unit displacement [cm3]. Same value as previous :guilabel:`asVEFF` but in a different measure unit.

* :guilabel:`qfAFSTECHIO`: stoichiometric air/fuel ratio of the reference fuel. When the calibration development is done using pure CH\ :sub:`4` as per the *G20* reference gas the value is 17.2.

* :guilabel:`asVOLUME_PLENUM`: volume of the intake manifold [m3]. Meaming the volume included after the throttle body and the inlet ducts (including runners if any are present in the intake manifold).

.. TODO: Place in a right chapter for calibration of speed density * :guilabel:`avOFF_TVALV`: difference of temperature between exhaust and intake valves [°C]

* :guilabel:`asNUMCYL`: number of cylinders of the engine. The calibration usually comes as default with the HDS SW, since the SW branch is dedicated to peculiar engine configurations. The :term:`RAM` copy of this calibration is ``asNCyl``. So :math:`asNCyl = asNUMCYL`

* :guilabel:`vsAPPL_TYPE`: Application Type. System calibration Must always be set to 2. The :term:`RAM` copy of this calibration is ``bsApplType``. So :math:`bsApplType = vsAPPL\_TYPE`. From this calibration descends many other automatic ECM to vehicle application behaviours. Ask to systems owner for more information.