Project Name Text Sensor
===================

.. seo::
    :description: Instructions for setting up project name text sensors.
    :image: new-box.svg

The ``project_name`` text sensor platform exposes the ESPHome Project name the firmware
was compiled against as a text sensor.

.. figure:: images/version-ui.png
    :align: center

.. code-block:: yaml

    # Example configuration entry
    text_sensor:
      - platform: project_name
        name: "ESPHome Project Name"

Configuration variables:
------------------------

- All other options from :ref:`Text Sensor <config-text_sensor>`.

See Also
--------

- :apiref:`project_name/project_name_text_sensor.h`
- :ghedit:`Edit`
