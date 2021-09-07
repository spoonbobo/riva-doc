Riva - Python API - Sample Services
===================================

This session illustrates the samples that demonstrate how Riva clients use Riva services through Riva Speech API server. To facilate the understanding of the usage of Riva Speech API, you might read `Python API <https://docs.nvidia.com/deeplearning/riva/user-guide/docs/development-python.html>`_

Prerequisites
-------------

To use the Riva Speech API from the client side, install :code:`riva-speech` Python wheel from Riva Quick Start scripts (Riva 1.5.0 is used in this demo).

.. code-block:: bash

    pip install riva_api-1.5.0-beta-py3-none-any.whl

.. note::

    Refer :ref:`riva_start_guide` to download Riva Quick Start scripts.

After installing :code:`riva_api-1.5.0-beta-py3-none-any.whl`, run :file:`riva_start_client.sh` to start a client container

.. code-block:: bash

	bash riva_start_client.sh

You may then host a jupyter server, and create notebooks to use the sample services.

.. code-block:: bash

	jupyter notebook --allow-root --ip=0.0.0.0

ASR - Recognize
---------------

1. Import necessary libraries

.. code-block:: python

	import grpc
	import librosa
	import io
	import IPython.display as ipd
	import riva_api.riva_asr_pb2 as rasr
	import riva_api.riva_asr_pb2_grpc as rasr_srv
	import riva_api.riva_audio_pb2 as ra

2. Establish connections between client and server

.. code-block:: python

	channel = grpc.insecure_channel("localhost:50051")
	server = rasr_srv.RivaSpeechRecognitionStub(channel)

3. Read audio

.. code-block:: python

	path = "/work/wav/sample.wav"
	audio, sr = librosa.core.load(path, sr=None)
	with io.open(path, 'rb') as fh:
		content = fh.read()

.. note::

    * Besides :file:`.wav` audio files, Riva also accept :file:`.alaw`, :file:`.mulaw`, :file:`.flac` formatted files with single channel.

4. Create a :code:`RecognizeRequest`

.. code-block:: python

    req = rasr.RecognizeRequest()
    req.audio = content                            
    req.config.encoding = ra.AudioEncoding.LINEAR_PCM 
    req.config.sample_rate_hertz = sr 
    req.config.language_code = "en-US" 
    req.config.max_alternatives = 1  
    req.config.enable_automatic_punctuation = True 
    req.config.audio_channel_count = 1  

.. seealso::

    * :ref:`riva_asr_proto`
  
    .. code-block:: proto

        message RecognizeRequest {
          // Provides information to recognizer that specifies how to process the request.
          RecognitionConfig config = 1;
          // The raw audio data to be processed. The audio bytes must be encoded as specified in
          // `RecognitionConfig`.
          bytes audio = 2;
        }

    .. code-block:: proto

        message RecognitionConfig {
            // The encoding of the audio data sent in the request.
            //
            // All encodings support only 1 channel (mono) audio.
            AudioEncoding encoding = 1;

            //  Sample rate in Hertz of the audio data sent in all
            // `RecognizeAudio` messages. 
            int32 sample_rate_hertz = 2;

            // Required. The language of the supplied audio as a
            // [BCP-47](https://www.rfc-editor.org/rfc/bcp/bcp47.txt) language tag.
            // Example: "en-US".
            // Currently only en-US is supported
            string language_code = 3;

            // Maximum number of recognition hypotheses to be returned.
            // Specifically, the maximum number of `SpeechRecognizeAlternative` messages
            // within each `SpeechRecognizeResult`.
            // The server may return fewer than `max_alternatives`.
            // If omitted, will return a maximum of one.
            int32 max_alternatives = 4;

            // The number of channels in the input audio data.
            // ONLY set this for MULTI-CHANNEL recognition.
            // Valid values for LINEAR16 and FLAC are `1`-`8`.
            // Valid values for OGG_OPUS are '1'-'254'.
            // Valid value for MULAW, AMR, AMR_WB and SPEEX_WITH_HEADER_BYTE is only `1`.
            // If `0` or omitted, defaults to one channel (mono).
            // Note: We only recognize the first channel by default.
            // To perform independent recognition on each channel set
            // `enable_separate_recognition_per_channel` to 'true'.
            int32 audio_channel_count = 7;

          // If `true`, the top result includes a list of words and
            // the start and end time offsets (timestamps) for those words. If
            // `false`, no word-level time offset information is returned. The default is
            // `false`.
            bool enable_word_time_offsets = 8;

            // If 'true', adds punctuation to recognition result hypotheses.
            // The default 'false' value does not add punctuation to result hypotheses.
            bool enable_automatic_punctuation = 11;

            // This needs to be set to `true` explicitly and `audio_channel_count` > 1
            // to get each channel recognized separately. The recognition result will
            // contain a `channel_tag` field to state which channel that result belongs
            // to. If this is not true, we will only recognize the first channel. The
            // request is billed cumulatively for all channels recognized:
            // `audio_channel_count` multiplied by the length of the audio.
            bool enable_separate_recognition_per_channel = 12;

            // Which model to select for the given request. Valid choices: Jasper, Quartznet
            string model = 13;

            // The verbatim_transcripts flag enables or disable inverse text normalization.
            // 'true' returns exactly what was said, with no denormalization.
            // 'false' applies inverse text normalization, also this is the default
            bool verbatim_transcripts = 14;

            // Custom fields for passing request-level
            // configuration options to plugins used in the
            // model pipeline.
            map<string, string> custom_configuration = 24;


        }

.. seealso::

    * :ref:`riva_audio_proto`

    .. code-block:: proto
      
        /*
          * AudioEncoding specifies the encoding of the audio bytes in the encapsulating message.
        */
        enum AudioEncoding {
            // Not specified.
            ENCODING_UNSPECIFIED = 0;

            // Uncompressed 16-bit signed little-endian samples (Linear PCM).
            LINEAR_PCM = 1;

            // `FLAC` (Free Lossless Audio
            // Codec) is the recommended encoding because it is
            // lossless--therefore recognition is not compromised--and
            // requires only about half the bandwidth of `LINEAR16`. `FLAC` stream
            // encoding supports 16-bit and 24-bit samples, however, not all fields in
            // `STREAMINFO` are supported.
            FLAC = 2;

            // 8-bit samples that compand 14-bit audio samples using G.711 PCMU/mu-law.
            MULAW = 3;

            // 8-bit samples that compand 13-bit audio samples using G.711 PCMU/a-law.
            ALAW = 20;
        }

5. Remote procedure call :code:`Recognize`

.. code-block:: python

    response = server.Recognize(req)

.. seealso::

    * :file:`Recognize` remote procedure call is defined as:

    .. code-block:: proto

        rpc Recognize(RecognizeRequest) returns (RecognizeResponse) {}

.. seealso::

    * :ref:`riva_asr_proto`

    .. code-block:: proto

        message RecognizeResponse {
            // Sequential list of transcription results corresponding to
            // sequential portions of audio. Currently only returns one transcript.
            repeated SpeechRecognitionResult results = 1;
        }

    .. code-block:: proto

        // A speech recognition result corresponding to the latest transcript
        message SpeechRecognitionResult {

          // May contain one or more recognition hypotheses (up to the
          // maximum specified in `max_alternatives`).
          // These alternatives are ordered in terms of accuracy, with the top (first)
          // alternative being the most probable, as ranked by the recognizer.
          repeated SpeechRecognitionAlternative alternatives = 1;

          // For multi-channel audio, this is the channel number corresponding to the
          // recognized result for the audio from that channel.
          // For audio_channel_count = N, its output values can range from '1' to 'N'.
          int32 channel_tag = 2;

          // Length of audio processed so far in seconds
          float audio_processed = 3;
        }

    .. code-block:: proto

        // Alternative hypotheses (a.k.a. n-best list).
        message SpeechRecognitionAlternative {
          // Transcript text representing the words that the user spoke.
          string transcript = 1;

          // The non-normalized confidence estimate. A higher number
          // indicates an estimated greater likelihood that the recognized words are
          // correct. This field is set only for a non-streaming
          // result or, of a streaming result where `is_final=true`.
          // This field is not guaranteed to be accurate and users should not rely on it
          // to be always provided.
          float confidence = 2;

          // A list of word-specific information for each recognized word. Only populated
          // if is_final=true
          repeated WordInfo words = 3;
        }

6. Print the response from Riva server

.. code-block:: python

    print(response)

.. code-block::

    Full Response Message:
    results {
      alternatives {
        transcript: "What is natural language processing? "
        confidence: 1.0
      }
      channel_tag: 1
      audio_processed: 4.800000190734863
    }

NLP - Core Services
-------------------

1. Import necessary libraries 
   
.. code-block:: python

    import grpc
    import riva_api.riva_nlp_pb2 as rnlp
    import riva_api.riva_nlp_pb2_grpc as rnlp_srv

2. Establish connections between client and server

.. code-block:: python

    channel = grpc.insecure_channel('localhost:50051')
    server = rnlp_srv.RivaLanguageUnderstandingStub(channel)

3. Select one of core NLP services below to continue.

.. note::

    * Notice for every core NLP service, :code:`NLPModelParams model` metadata (name of models to be used) is included in every request to specify model characteristics/ requirements.

    * :ref:`riva_nlp_proto`

    .. code-block:: proto

        // NLPModelParams is a metadata message that is included in every request message
        // used by the Core NLP Service and is used to specify model characteristics/requirements
        message NLPModelParams {
          // Requested model to use. If unavailable, the request will return an error
          string model_name = 1;
        }

    * You can use :code:`docker logs riva-speech` to check which models are available for services.

TransformText
~~~~~~~~~~~~~

4. Create a :code:`TextTransformRequest` (string -> string)

.. code-block:: python

    req = rnlp.TextTransformRequest()
    req.model.model_name = "riva_punctuation"
    req.text.append("add punctuation to this sentence")
    req.text.append("do you have any NVDA stocks")
    req.text.append("i buy one apple four oranges and two watermelons "
                    "for my after-dinner it's going to be very cool")

.. seealso::

    * :ref:`riva_nlp_proto`

    .. code-block:: proto

        message TextTransformRequest {
          // Each repeated text element is handled independently for handling multiple
          // input strings with a single request
          repeated string text = 1;
          uint32 top_n = 2; // 
          NLPModelParams model = 3;
        }

5. remote procedure call :code:`TransformText`

.. code-block:: python

    nlp_resp = server.TransformText(req)

.. note::

    * :code:`TransformText` remote procedure call is defined as:
       
    .. code-block:: proto

   		  rpc TransformText(TextTransformRequest) returns (TextTransformResponse) {}

    * :ref:`riva_nlp_proto`

    .. code-block:: proto

        // TextTransformResponse is returned by the TransformText method. Responses
        // are returned in the same order as they were requested.
        message TextTransformResponse {
          repeated string text = 1;
        }

6. Print the response from Riva server

.. code-block:: python3

    print(nlp_resp)

.. code-block::

    text: "Add punctuation to this sentence."
    text: "Do you have any NVDA stocks?"
    text: "I buy one apple, four oranges and two watermelons for my after-dinner It\'s going to be very cool."


ClassifyText
~~~~~~~~~~~~

4. Create a :code:`TextClassRequest` (text -> label)

.. code-block:: python

    request = rnlp.TextClassRequest()
    request.model.model_name = "riva_text_classification_domain" 
    request.text.append("Is it going to snow in Burlington, Vermont tomorrow night?")
    request.text.append("What causes rain?")
    request.text.append("What is your favorite season?")

.. seealso::

    * :ref:`riva_nlp_proto`

    .. code-block:: proto

        // TextClassRequest is the input message to the ClassifyText service.
        message TextClassRequest {
          // Each repeated text element is handled independently for handling multiple
          // input strings with a single request
          repeated string text = 1;
          
          // Return the top N classification results for each input. 0 or 1 will return top class, otherwise N.
          // Note: Current disabled.
          uint32 top_n = 2;
          NLPModelParams model = 3;
        }

5. Remote procedure call :code:`ClassifyText`

.. code-block:: python3

    ct_response = riva_nlp.ClassifyText(request)

.. note::

  * :code:`ClassifyText` remote procedure call is defined as:

  .. code-block:: proto

      rpc ClassifyText(TextClassRequest) returns (TextClassResponse) {}

  * :ref:`riva_nlp_proto`

  .. code-block:: proto

      message TextClassResponse {
        repeated ClassificationResult results = 1;
      }

  .. code-block:: proto

      // ClassificationResults contain zero or more Classification messages
      // If the number of Classifications is > 1, top_n > 1 must have been
      // specified.
      message ClassificationResult {
        repeated Classification labels = 1;
      }

  .. code-block:: proto

      // Classification messages return a class name and corresponding score
      message Classification {
        string class_name = 1;
        float score = 2;
      }

6. Print the response from the Riva Server

.. code-block:: python

    print(ct_response.results)

.. code-block::

    [labels {
      class_name: "weather"
      score: 0.9975590109825134
    }
    , labels {
      class_name: "meteorology"
      score: 0.984375
    }
    , labels {
      class_name: "personality"
      score: 0.984375
    }
    ]

ClassifyTokens
~~~~~~~~~~~~~~

4. Create a :code:`TokenClassRequest` (token -> label)

.. code-block:: python

    req = rnlp.TokenClassRequest()
    req.model.model_name = "riva_ner"
    req.text.append("Jensen Huang is the CEO of NVIDIA Corporation, "
                "located in Santa Clara, California")

.. seealso::

    * :ref:`riva_nlp_proto`

    .. code-block:: proto

        // TokenClassRequest is the input message to the ClassifyText service.
        message TokenClassRequest {
          // Each repeated text element is handled independently for handling multiple
          // input strings with a single request
          repeated string text = 1;

          // Return the top N classification results for each input. 0 or 1 will return top class, otherwise N.
          // Note: Current disabled.
          uint32 top_n = 3;
          NLPModelParams model = 4;
        }

5. Remote procedure call :code:`ClassifyTokens`

.. code-block:: python3

    resp = server.ClassifyTokens(req)

.. note::

    * :code:`ClassifyTokens` remote procedure call is defined as:

    .. code-block:: proto

        rpc ClassifyTokens(TokenClassRequest) returns (TokenClassResponse) {}

    * :ref:`riva_nlp_proto`

    .. code-block:: proto

        // TokenClassResponse returns a single TokenClassSequence per input request
        message TokenClassResponse {
          repeated TokenClassSequence results = 1;
        }

    .. code-block:: proto

        // TokenClassSequence is used for returning a sequence of TokenClassValue objects
        // in the original order of input tokens
        message TokenClassSequence {
          repeated TokenClassValue results = 1;
        }

    .. code-block:: proto
      
        // TokenClassValue is used to correlate an input token with its classification results
        message TokenClassValue {
          string token = 1;
          repeated Classification label = 2;
          repeated Span span = 3;
        }

    .. code-block:: proto
        
        // Span of a particular result
        message Span {
          uint32 start = 1;
          uint32 end = 2;
        }

6. Print the response from the Riva server

.. code-block:: python

    print(resp)

.. code-block::

    results {
      results {
        token: "jensen huang"
        label {
          class_name: "PER"
          score: 0.9972559809684753
        }
        span {
          end: 11
        }
      }
      results {
        token: "nvidia corporation"
        label {
          class_name: "ORG"
          score: 0.9613900184631348
        }
        span {
          start: 27
          end: 44
        }
      }
      results {
        token: "santa clara"
        label {
          class_name: "LOC"
          score: 0.9977059960365295
        }
        span {
          start: 58
          end: 68
        }
      }
      results {
        token: "california"
        label {
          class_name: "LOC"
          score: 0.9961509704589844
        }
        span {
          start: 71
          end: 80
        }
      }
    }
    results {
      results {
        token: "hku"
        label {
          class_name: "ORG"
          score: 0.830374002456665
        }
        span {
          start: 26
          end: 28
        }
      }
    }


NLP - AnalyzeIntent
-------------------

1. Import necessary libraries

.. code-block:: python

    import grpc
    import riva_api.riva_nlp_pb2_grpc as rnlp_srv
    import riva_api.riva_nlp_pb2 as rnlp

2. Establish connections between client and server

.. code-block:: python

    channel = grpc.insecure_channel('localhost:50051')
    server = rnlp_srv.RivaLanguageUnderstandingStub(channel)

3. Create a :code:`AnalyzeIntentRequest`

.. code-block:: python

    req = rnlp.AnalyzeIntentRequest(query="How is the weather today in Hong Kong")

.. seealso::

   * :ref:`riva_nlp_proto`

   .. code-block:: proto

        // AnalyzeIntentRequest is the input message for the AnalyzeIntent service
        message AnalyzeIntentRequest {
          // The string to analyze for intent and slots
          string query = 1;
          // Optional configuration for the request, including providing context from previous turns
          // and hardcoding a domain/language
          AnalyzeIntentOptions options = 2;
        }

4. Remote procedure call :code:`AnalyzeIntent`

.. code-block:: python

    resp = server.AnalyzeIntent(req)

.. note::

	* :code:`AnalyzeIntent` remote procedure call is defined as:
    
	.. code-block:: proto

		 rpc AnalyzeIntent(AnalyzeIntentRequest) returns (AnalyzeIntentResponse) {}

.. seealso::

   * :ref:`riva_nlp_proto`

   .. code-block:: proto

        // AnalyzeIntentResponse is returned by the AnalyzeIntent service, and includes information
        // related to the query's intent, (optionally) slot data, and its domain.
        message AnalyzeIntentResponse {
          // Intent classification result, including the label and score
          Classification intent = 1;
          // List of tokens explicitly marked as filling a slot relevant to the intent, where the
          // tokens may not exactly match the input (based on the recombined values after tokenization)
          repeated TokenClassValue slots = 2; 
          // Returns the inferred domain for the query if not hardcoded in the request. In the case where
          // the domain was hardcoded in AnalyzeIntentRequest, the returned domain is an exact match to the
          // request. In the case where no domain matches the query, intent and slots will be unset.
          // 
          // DEPRECATED, use Classification domain field.
          string domain_str = 3;

          // Returns the inferred domain for the query if not hardcoded in the request. In the case where
          // the domain was hardcoded in AnalyzeIntentRequest, the returned domain is an exact match to the
          // request. In the case where no domain matches the query, intent and slots will be unset.
          Classification domain = 4;
        }

5. Print the response from Riva server:

.. code-block:: python

    print(resp)

.. code-block:: 

    intent {
      class_name: "weather.weather"
      score: 0.9965819716453552
    }
    slots {
      token: "today"
      label {
        class_name: "weatherforecastdaily"
        score: 0.8901410102844238
      }
    }
    slots {
      token: "hong"
      label {
        class_name: "weatherplace"
        score: 0.7496770024299622
      }
    }
    slots {
      token: "kong"
      label {
        class_name: "weatherplace"
        score: 0.5112429857254028
      }
    }
    domain_str: "weather"
    domain {
      class_name: "weather"
      score: 0.9970700144767761
    }

TTS - SynthesizeSpeech
----------------------

1. Import necessary libraries

.. code-block:: python

    import grpc
    import numpy as np
    import IPython.display as ipd
    import riva_api.riva_tts_pb2 as rtts
    import riva_api.riva_tts_pb2_grpc as rtts_srv
    import riva_api.audio_pb2 as ra

2. Establish connections between client and server

.. code-block:: python

    channel = grpc.insecure_channel("localhost:50051")
    server = rtts_srv.RivaSpeechSynthesisStub(channel)

3. Create a :code:`SynthesizeSpeechRequest`

.. code-block:: python

    req = rtts.SynthesizeSpeechRequest()
    req.text = "Buy Nvidia stocks, You will get rich"
    req.language_code = "en-US"                 
    req.encoding = ra.AudioEncoding.LINEAR_PCM    
    req.sample_rate_hz = 22050       
    req.voice_name = "ljspeech"

.. seealso::

    * :ref:`riva_tts_proto`

    .. code-block:: proto

        message SynthesizeSpeechRequest {
            string text = 1;
            string language_code = 2;
            // audio encoding params
            AudioEncoding encoding = 3;
            int32 sample_rate_hz = 4;
            // voice params
            string voice_name = 5;
        }

4. Remote procedure call :code:`SyntheSize` 

.. code-block:: python

    resp = server.Synthesize(req)

.. note::

    * :code:`SyntheSize` remote procedure call is defined as:

    .. code-block:: proto

        rpc Synthesize(SynthesizeSpeechRequest) returns (SynthesizeSpeechResponse) {}

    .. code-block:: proto

        message SynthesizeSpeechResponse {
            bytes audio = 1;
        }


5. Convert buffer to numpy array and listen

.. code-block:: python

    audio_samples = np.frombuffer(resp.audio, dtype=np.float32)
    ipd.Audio(audio_samples, rate=22050)
