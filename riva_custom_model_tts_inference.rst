Riva - Custom Model - NeMo - TTS - Inference
============================================

Two-stage Approach
------------------

Download and load models
~~~~~~~~~~~~~~~~~~~~~~~~
.. code-block:: python

    import soundfile as sf
    from nemo.collections.tts.models.base import SpectrogramGenerator, Vocoder

    ## spectrogram generators
    spec_generator = SpectrogramGenerator.from_pretrained(model_name="tts_en_fastpitch").cuda()
    # spec_generator = SpectrogramGenerator.from_pretrained(model_name="tts_en_tacotron2").cuda()
    # spec_generator = SpectrogramGenerator.from_pretrained(model_name="tts_en_glowtts").cuda()
    # spec_generator = SpectrogramGenerator.from_pretrained(model_name="tts_en_tts_en_talknet").cuda()
    # spec_generator = SpectrogramGenerator.from_pretrained(model_name="tts_en_fastspeech2").cuda()

    ## Audio generators
    vocoder = Vocoder.from_pretrained(model_name="tts_hifigan").cuda()
    # vocoder = Vocoder.from_pretrained(model_name="tts_waveglow").cuda()
    # vocoder = Vocoder.from_pretrained(model_name="tts_squeezewave").cuda()
    # vocoder = Vocoder.from_pretrained(model_name="tts_uniglow").cuda()
    # vocoder = Vocoder.from_pretrained(model_name="tts_melgan").cuda()

    # TBA for griffin-lim

.. note::

    * See :ref:`tts_arch` for details of models

Convert to audio 
~~~~~~~~~~~~~~~~

.. code-block:: python

    parsed = spec_generator.parse("You can type your sentence here to get nemo to produce speech.")
    spectrogram = spec_generator.generate_spectrogram(tokens=parsed)
    audio = vocoder.convert_spectrogram_to_audio(spec=spectrogram)

.. note::

    Mel Spectrogram generators have two helper functions:

    * :code:`parse()``: Accepts raw python strings and returns a torch.tensor that represents tokenized text
    * :code:`generate_spectrogram()`: Accepts a batch of tokenized text and returns a torch.tensor that represents a batch of spectrograms

    Vocoder have just one helper function:

    * :code:`convert_spectrogram_to_audio()`: Accepts a batch of spectrograms and returns a torch.tensor that represents a batch of raw audio

Save Audio
~~~~~~~~~~

.. code-block:: python

    sf.write("speech.wav", audio.to('cpu').detach().numpy()[0], 22050)

End-to-end Approach
-------------------

Download and load models
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python
    
    import soundfile as sf
    from nemo.collections.tts.models.base import SpectrogramGenerator, Vocoder, TextToWaveform

    e2e_model = TextToWaveform.from_pretrained("tts_en_e2e_fastpitchhifigan").cuda()
    # e2e_model = TextToWaveform.from_pretrained("tts_en_e2e_fastspeech2hifigan").cuda()

Convert to audio 
~~~~~~~~~~~~~~~~

.. code-block:: python

    parsed = e2e_model.parse("You can type your sentence here to get nemo to produce speech.")
    audio = e2e_model.convert_text_to_waveform(tokens=parsed)[0]

.. note::

    End-to-end models have two helper functions:

    * :code:`parse()`: Accepts raw python strings and returns a torch.tensor that represents tokenized text
    * :code:`convert_text_to_waveform()`: Accepts a batch of tokenized text and returns a torch.tensor that represents a batch of raw audio

Save Audio
~~~~~~~~~~

.. code-block:: python

    sf.write("speech.wav", audio.to('cpu').detach().numpy()[0], 22050)