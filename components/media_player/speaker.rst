Speaker Audio Media Player
==========================

.. seo::
    :description: Instructions for setting up a speaker media player in ESPHome.
    :image: speaker.svg

The ``speaker`` media player platform allows you to play on-device and online audio media via :doc:`speaker components </components/speaker/index>`.

This platform greatly benefits from having external PSRAM. See the :ref:`performance section <speaker_media_player-performance>` for details.

It natively supports decoding ``FLAC``, ``MP3``, and ``WAV`` audio files. Home Assistant (since version 2024.10) can proxy any media it sends and transcode it to a specified format and sample rate to minimize the device's computational load.

It supports two different audio pipelines: announcement and media. Each audio pipeline must output to a unique speaker. Use a :doc:`mixer speaker </components/speaker/mixer>` component to create two different speakers that output to a single audio speaker.

On-device files built directly into the firmware are played without a network connection. Encode on-device files with the configured sample rate, 1 or 2 channels, and 16 bits per sample.

This platform only works on ESP32 based chips using the :ref:`esp32-espidf_framework`.

.. warning::

    Audio and voice components consume a significant amount of resources (RAM, CPU) on the device.

    **Crashes are likely to occur** if you include too many additional components in your device's
    configuration. In particular, Bluetooth/BLE components are known to cause issues when used in
    combination with Voice Assistant and/or other audio components.

.. code-block:: yaml

    # Example minimal configuration entry
    media_player:
      - platform: speaker
        announcement_pipeline:
          speaker: announcment_spk_id

Configuration variables:
------------------------

- **announcement_pipeline** (**Required**, Pipeline Schema): Configuration settings for the announcement pipeline.

  - **speaker** (**Required**, :ref:`config-id`): The :doc:`speaker </components/speaker/index>` to output the audio.
  - **format** (**Optional**, enum): The audio format Home Asssistant will transcode audio to before sending it to the device. One of ``FLAC``, ``MP3``, ``WAV``, or ``NONE``. ``NONE`` disables transcoding in Home Assistant. Defaults to ``FLAC``.
  - **sample_rate** (*Optional*, positive integer): Sample rate for the transcoded audio. Should be supported by the configured ``speaker`` component. Defaults to the speaker's sample rate.
  - **num_channels** (*Optional*, positive integer): Number of channels for the transcoded audio. Must be either ``1`` or ``2``. Defaults to the speaker's number of channels.

- **media_pipeline** (*Optional*, Pipeline Schema): Configuration settings for the media pipeline. Same options as the ``announcement_pipeline``.
- **buffer_size** (*Optional*, positive integer): The buffer size in bytes for each pipeline. Must be between ``4000`` and ``4000000``. Defaults to ``1000000``.
- **codec_support_enabled** (*Optional*, boolean): Enables the MP3 and FLAC decoders and optimizes the WiFi configuration for streaming high quality audio. Defaults to ``true``.
- **task_stack_in_psram** (*Optional* boolean): Run the audio tasks in external memory. Defaults to ``false``.
- **volume_increment** (*Optional*, percentage): Increment amount that the ``media_player.volume_up`` and ``media_player.volume_down`` actions will increase or decrease volume by. Defaults to ``5%``.
- **volume_min** (*Optional*, percentage): The minimum volume allowed. Defaults to ``0%``.
- **volume_max** (*Optional*, percentage): The maximum volume allowed. Defaults to ``100%``.
- **files** (*Optional*, list): A list of media files to build into the firmware for on-device playback.
    - **id** (**Required**, :ref:`config-id`): Unique ID for the file.
    - **file** (**Required**, string): Path to audio file. Can be a local file path or a URL.
- **on_mute** (*Optional*, :ref:`Automation <automation>`): An automation to perform when muted.
- **on_unmute** (*Optional*, :ref:`Automation <automation>`): An automation to perform when unmuted.
- **on_volume** (*Optional*, :ref:`Automation <automation>`): An automation to perform when the volume is changed.
- All other options from :ref:`Media Player <config-media_player>`

.. _speaker_media_player-performance:

Performance
-----------

Decoding audio files is CPU and memory intensive. PSRAM external memory is strongly recommended. To use the component on a memory constrained device, define only the announcement pipeline, decrease the buffer size, set ``codec_support_enabled`` to false, and set the pipeline transcode setting format to ``WAV`` with a low sample rate and only 1 channel.

In general, decoding FLAC has the lowest CPU usage, but requires a strong WiFi connection. Decoding MP3 requires less data to be sent over WiFi but is more CPU intensive to decode. Decoding WAV is only recommended at low sample rates if streamed over a network connection.

Increasing the buffer size may reduce stuttering, but do not set it to the entire size of the external memory. Each pipeline allocates the configured amount, and this setting also does not take into account other smaller buffers allocated throughout the audio stack.

.. _speaker_media_player-examples:

Example Configuration
---------------------

This example outputs audio to an  :doc:`I²S Audio Speaker </components/speaker/i2s_audio>` configured with a 48000 Hz sample rate. It uses a ``mixer`` speaker component to handle combining the two differnet pipelines, and it uses ``resampler`` speaker components to ensure the source speakers uses the same sample rate.

It adds a switch for playing an on-device file for an alarm notification. Any playing media is ducked while the alarm is activated. After the alarm is turned off, the media ducking will gradually stop.

.. code-block:: yaml

    i2s_audio:
        i2s_lrclk_pin: GPIOXX
        i2s_bclk_pin: GPIOXX
    speaker:
      - platform: i2s_audio
        id: speaker_id
        dac_type: external
        i2s_dout_pin: GPIOXX
        sample_rate: 48000
      - platform: mixer
        id: mixer_speaker_id
        output_speaker: speaker_id
        source_speakers:
          - id: announcement_spk_mixer_input
          - id: media_spk_mixer_input
      - platform: resampler
        id: media_spk_resampling_input
        output_speaker: media_spk_mixer_input
      - platform: resampler
        id: announcement_spk_resampling_input
        output_speaker: announcement_spk_mixer_input
    media_player:
      - platform: speaker
        name: "Speaker Media Player"
        id: speaker_media_player_id
        media_pipeline:
            speaker: media_spk_resampling_input
            num_channels: 2
        announcement_pipeline:
            speaker: announcement_spk_resampling_input
            num_channels: 1
        files:
          - id: alarm_sound
            file: alarm.flac # Placed in the yaml directory. Should be encoded with a 48000 Hz sample rate, mono or stereo audio, and 16 bits per sample.
    switch:
      - platform: template
        name: "Ring Timer"
        id: timer_ringing
        optimistic: true
        restore_mode: ALWAYS_OFF
        on_turn_off:
            # Stop playing the alarm
            - media_player.stop:
                announcement: true
            - mixer_speaker.apply_ducking:  # Stop ducking the media stream over 2 seconds
                id: media_spk_mixer_input
                decibel_reduction: 0
                duration: 2.0s
        on_turn_on:
            # Duck media audio by 20 decibels instantly
            - mixer_speaker.apply_ducking:
                id: media_spk_mixer_input
                decibel_reduction: 20
                duration: 0.0s
            - while:
                condition:
                    switch.is_on: timer_ringing
                then:
                    # Play the alarm sound as an announcement
                    - media_player.speaker.play_on_device_media_file:
                        media_file: alarm_sound
                        announcement: true
                    # Wait until the alarm sound starts playing
                    - wait_until:
                        media_player.is_announcing:
                    # Wait until the alarm sound stops playing
                    - wait_until:
                        not:
                          media_player.is_announcing:


Automations
-----------

.. _media_player-speaker-play_on_device_media_file:

``media_player.speaker.play_on_device_media_file`` Action
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This action will play a on-device media file.

.. code-block:: yaml

    on_...:
      # Simple
      - media_player.speaker.play_on_device_media_file: file_id

      # Full
      - media_player.speaker.play_on_device_media_file:
          media_file: wake_word_trigger_sound
          announcement: true

Configuration variables:

- **media_file** (**Required**, :ref:`config-id`): The ID of the media file.
- **announcement** (*Optional*, boolean): Whether to play back the file as an announcement or media stream. Defaults to ``false``.


See also
--------

- :doc:`/components/speaker/index`
- :doc:`index`
- :ghedit:`Edit`
