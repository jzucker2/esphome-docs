Project Version Text Sensor
===========================

.. seo::
    :description: Instructions for setting up project version text sensors.
    :image: new-box.svg

The ``project_version`` text sensor platform exposes the ESPHome Project version the firmware
was compiled against as a text sensor.

.. figure:: images/version-ui.png
    :align: center

.. code-block:: yaml

    # Example configuration entry
    text_sensor:
      - platform: project_version
        name: "ESPHome Project Version"

Configuration variables:
------------------------

- **hide_timestamp** (*Optional*, boolean): Allows you to hide the compilation timestamp from the version string. Defaults to ``false``.
- All other options from :ref:`Text Sensor <config-text_sensor>`.

Disabling the compilation timestamp:
------------------------------------

.. code-block:: yaml

    # Example configuration entry
    text_sensor:
      - platform: project_version
        name: "ESPHome Project Version"
        hide_timestamp: true

This will, for example, change the output of the sensor from:

``1.0.0-dev May 30 2024, 09:07:35`` to just ``1.0.0-dev``


See Also
--------

- :apiref:`project_version/project_version_text_sensor.h`
- :ghedit:`Edit`
