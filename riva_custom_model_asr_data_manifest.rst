.. _nemo_data_manifest:

Riva - Custom Model - NeMo - Data Manifest - ASR
================================================

The standardized manifest (a json file storing all metadata of a given dataset) format for NeMo is that

* each line corresponds to one sample of audio
* Number of lines equals to number of samples

A sample of a line in the manifest

.. code-block:: bash

    {"audio_filepath": "path/to/audio.wav", "duration": 3.45, "text": "transcript of audio.wav"}