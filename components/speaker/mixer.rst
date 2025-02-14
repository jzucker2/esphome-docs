Mixer Speaker
=============

.. seo::
    :description: Instructions for setting up mixer speakers in ESPHome.
    :image: mixer.svg

The ``mixer`` speaker platform allows you to mix audio sent to different source speakers into one output which is sent to another :doc:`speaker component </components/speaker/index>`. Individual source speakers may be ducked (made quieter) with the :ref:`apply ducking action <mixer_speaker-apply_ducking>`.

When mixing multiple audio streams into one, they must have the same sample rate. If they are different, enable queue mode so that only one source speaker's audio is played at a time. Otherwise, use a :doc:`resampler speaker </components/speaker/resampler>` to send audio to the source speakers.

This platform only works on ESP32 based chips.

.. warning::

    Audio and voice components consume a significant amount of resources (RAM, CPU) on the device.

    **Crashes are likely to occur** if you include too many additional components in your device's
    configuration. In particular, Bluetooth/BLE components are known to cause issues when used in
    combination with Voice Assistant and/or other audio components.

.. code-block:: yaml

    # Example configuration entry
    speaker:
      - platform: mixer
        output_speaker: speaker_id
        source_speakers:
          - id: announcement_mixer_input_speaker_id
          - id: media_mixer_input_speaker_id

Configuration variables:
------------------------

- **output_speaker** (**Required**, :ref:`config-id`): The :doc:`speaker </components/speaker/index>` to output the mixed audio.
- **source_speakers** (**Required**, list): A list of source speaker inputs. Must have at least 2 and at most 8 speakers.

    - **buffer_duration** (*Optional*, :ref:`config-time`): The duration of the internal ring buffer. Larger values can reduce stuttering but use more memory. Defaults to ``100ms``.
    - **timeout** (*Optional*, :ref:`config-time`): How long to wait after finishing playback before releasing the bus. Set to ``never`` to never stop the speaker due to a timeout. Defaults to ``500ms``.
    - All other options from :ref:`Speaker Component <config-speaker>`.

- **num_channels** (*Optional*, positive integer): The number of audio channels to send to the output speaker. Either ``1`` or ``2``. Defaults to the output speaker's number of channels.
- **queue_mode** (*Optional*, boolean): Enables queue mode. If enabled, audio isn't mixed but instead each source speaker's audio is played successively, starting with the first listed source speaker.
- **task_stack_in_psram** (*Optional* boolean): Only with ``esp-idf``. Run the audio tasks in external memory. Defaults to ``false``.


Automations
-----------

.. _mixer_speaker-apply_ducking:

``mixer_speaker.apply_ducking`` Action
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This action ducks (reduces the volume of) the media stream.

.. code-block::

    on_...:
      - mixer_speaker.apply_ducking:
          id: media_mixer_source_speaker_id
          decibel_reduction: 20
          duration: 2.0s

Configuration variables:

- **decibel_reduction** (**Required**, int, templatable): The reduction of the media stream in decibels. Must be between 0 and 50.
- **duration** (*Optional*, :ref:`config-time`, templatable): The length of time to transition between the current reduction level and the new reduction level. Defaults to ``0s``.


See also
--------

- :doc:`index`
- :ghedit:`Edit`
