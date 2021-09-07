Riva - gRPC API
===============

.. image:: assets/landing-2.svg
    :width: 800px
    :alt: gRPC architecture

Riva's services are exposed using gRPC to maximise its compatibility with different software infrastructure and easier integrations.

.. _riva_asr_proto:

riva_asr.proto
--------------

.. code-block:: proto
    :linenos:

    // Copyright 2019 Google LLC.
    // Copyright (c) 2021, NVIDIA CORPORATION. All rights reserved.
    //
    // Licensed under the Apache License, Version 2.0 (the "License");
    // you may not use this file except in compliance with the License.
    // You may obtain a copy of the License at
    //
    //     http://www.apache.org/licenses/LICENSE-2.0
    //
    // Unless required by applicable law or agreed to in writing, software
    // distributed under the License is distributed on an "AS IS" BASIS,
    // WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    // See the License for the specific language governing permissions and
    // limitations under the License.
    //

    syntax = "proto3";

    package nvidia.riva.asr;

    option cc_enable_arenas = true;
    option go_package = "nvidia.com/riva_speech";

    import "riva_audio.proto";

    /*
    * The RivaSpeechRecognition service provides two mechanisms for converting speech to text.
    */
    service RivaSpeechRecognition {
        // Recognize expects a RecognizeRequest and returns a RecognizeResponse. This request will block
        // until the audio is uploaded, processed, and a transcript is returned.
        rpc Recognize(RecognizeRequest) returns (RecognizeResponse) {}
        // StreamingRecognize is a non-blocking API call that allows audio data to be fed to the server in
        // chunks as it becomes available. Depending on the configuration in the StreamingRecognizeRequest,
        // intermediate results can be sent back to the client. Recognition ends when the stream is closed
        // by the client.
        rpc StreamingRecognize(stream StreamingRecognizeRequest) returns (stream StreamingRecognizeResponse) {}
    }


    /*
    * RecognizeRequest is used for batch processing of a single audio recording.
    */
    message RecognizeRequest {
        // Provides information to recognizer that specifies how to process the request.
        RecognitionConfig config = 1;
        // The raw audio data to be processed. The audio bytes must be encoded as specified in
        // `RecognitionConfig`.
        bytes audio = 2;
    }


    /*
    * A StreamingRecognizeRequest is used to configure and stream audio content to the
    * Riva ASR Service. The first message sent must include only a StreamingRecognitionConfig.
    * Subsequent messages sent in the stream must contain only raw bytes of the audio
    * to be recognized.
    */
    message StreamingRecognizeRequest {
        // The streaming request, which is either a streaming config or audio content.
        oneof streaming_request {
            // Provides information to the recognizer that specifies how to process the
            // request. The first `StreamingRecognizeRequest` message must contain a
            // `streaming_config`  message.
            StreamingRecognitionConfig streaming_config = 1;
            // The audio data to be recognized. Sequential chunks of audio data are sent
            // in sequential `StreamingRecognizeRequest` messages. The first
            // `StreamingRecognizeRequest` message must not contain `audio` data
            // and all subsequent `StreamingRecognizeRequest` messages must contain
            // `audio` data. The audio bytes must be encoded as specified in
            // `RecognitionConfig`.
            bytes audio_content = 2;
        }
    }

    // Provides information to the recognizer that specifies how to process the request
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

    // Provides information to the recognizer that specifies how to process the request
    message StreamingRecognitionConfig {
                    // Provides information to the recognizer that specifies how to process the request
        RecognitionConfig config = 1;

                    // If `true`, interim results (tentative hypotheses) may be
        // returned as they become available (these interim results are indicated with
        // the `is_final=false` flag).
        // If `false` or omitted, only `is_final=true` result(s) are returned.
        bool interim_results = 2;
    }

    // The only message returned to the client by the `Recognize` method. It
    // contains the result as zero or more sequential `SpeechRecognitionResult`
    // messages.
    message RecognizeResponse {
        // Sequential list of transcription results corresponding to
            // sequential portions of audio. Currently only returns one transcript.
        repeated SpeechRecognitionResult results = 1;
    }

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

    // Word-specific information for recognized words.
    message WordInfo {
        // Time offset relative to the beginning of the audio in ms
        // and corresponding to the start of the spoken word.
        // This field is only set if `enable_word_time_offsets=true` and only
        // in the top hypothesis.
        int32 start_time = 1;

        // Time offset relative to the beginning of the audio in ms
        // and corresponding to the end of the spoken word.
        // This field is only set if `enable_word_time_offsets=true` and only
        // in the top hypothesis.
        int32 end_time = 2;

        // The word corresponding to this set of information.
        string word = 3;
    }


    // `StreamingRecognizeResponse` is the only message returned to the client by
    // `StreamingRecognize`. A series of zero or more `StreamingRecognizeResponse`
    // messages are streamed back to the client.
    //
    // Here are few examples of `StreamingRecognizeResponse`s
    //
    // 1. results { alternatives { transcript: "tube" } stability: 0.01 }
    //
    // 2. results { alternatives { transcript: "to be a" } stability: 0.01 }
    //
    // 3. results { alternatives { transcript: "to be or not to be"
    //                             confidence: 0.92 }
    //              alternatives { transcript: "to bee or not to bee" }
    //              is_final: true }
    //

    message StreamingRecognizeResponse {

        // This repeated list contains the latest transcript(s) corresponding to
        // audio currently being processed.
                    // Currently one result is returned, where each result can have multiple
                    // alternatives
        repeated StreamingRecognitionResult results = 1;
    }

    // A streaming speech recognition result corresponding to a portion of the audio
    // that is currently being processed.
    message StreamingRecognitionResult {
        // May contain one or more recognition hypotheses (up to the
        // maximum specified in `max_alternatives`).
        // These alternatives are ordered in terms of accuracy, with the top (first)
        // alternative being the most probable, as ranked by the recognizer.
        repeated SpeechRecognitionAlternative alternatives = 1;

        // If `false`, this `StreamingRecognitionResult` represents an
        // interim result that may change. If `true`, this is the final time the
        // speech service will return this particular `StreamingRecognitionResult`,
        // the recognizer will not return any further hypotheses for this portion of
        // the transcript and corresponding audio.
        bool is_final = 2;

        // An estimate of the likelihood that the recognizer will not
        // change its guess about this interim result. Values range from 0.0
        // (completely unstable) to 1.0 (completely stable).
        // This field is only provided for interim results (`is_final=false`).
        // The default of 0.0 is a sentinel value indicating `stability` was not set.
        float stability = 3;

        // For multi-channel audio, this is the channel number corresponding to the
        // recognized result for the audio from that channel.
        // For audio_channel_count = N, its output values can range from '1' to 'N'.
        int32 channel_tag = 5;

        // Length of audio processed so far in seconds
        float audio_processed = 6;
    }

.. _riva_nlp_proto:

riva_nlp.proto
--------------

.. code-block:: proto
    :linenos:

    // Copyright (c) 2021, NVIDIA CORPORATION.  All rights reserved.
    //
    // NVIDIA CORPORATION and its licensors retain all intellectual property
    // and proprietary rights in and to this software, related documentation
    // and any modifications thereto.  Any use, reproduction, disclosure or
    // distribution of this software and related documentation without an express
    // license agreement from NVIDIA CORPORATION is strictly prohibited.

    syntax = "proto3";

    package nvidia.riva.nlp;

    option cc_enable_arenas = true;
    option go_package = "nvidia.com/riva_speech";

    /* Riva Natural Language Services implement generic and task-specific APIs.
    * The generic APIs allows users to design
    * models for arbitrary use cases that conform simply with input and output types
    * specified in the service. As an explicit example, the ClassifyText function
    * could be used for sentiment classification, domain recognition, language
    * identification, etc.
    * The task-specific APIs can be used for popular NLP tasks such as
    * intent recognition (as well as slot filling), and entity extraction.
    */

    service RivaLanguageUnderstanding {

        // ClassifyText takes as input an input/query string and parameters related
        // to the requested model to use to evaluate the text. The service evaluates the
        // text with the requested model, and returns one or more classifications.
        rpc ClassifyText(TextClassRequest) returns (TextClassResponse) {}

        // ClassifyTokens takes as input either a string or list of tokens and parameters
        // related to which model to use. The service evaluates the text with the requested
        // model, performing additional tokenization if necessary, and returns one or more
        // class labels per token.
        rpc ClassifyTokens(TokenClassRequest) returns (TokenClassResponse) {}

        // TransformText takes an input/query string and parameters related to the
        // requested model and returns another string. The behavior of the function
        // is defined entirely by the underlying model and may be used for
        // tasks like translation, adding punctuation, augment the input directly, etc.
        rpc TransformText(TextTransformRequest) returns (TextTransformResponse) {}

        // AnalyzeEntities accepts an input string and returns all named entities within
        // the text, as well as a category and likelihood.
        rpc AnalyzeEntities(AnalyzeEntitiesRequest) returns (TokenClassResponse) {}

        // AnalyzeIntent accepts an input string and returns the most likely
        // intent as well as slots relevant to that intent.
        //
        // The model requires that a valid "domain" be passed in, and optionally
        // supports including a previous intent classification result to provide
        // context for the model.
        rpc AnalyzeIntent(AnalyzeIntentRequest) returns (AnalyzeIntentResponse) {}

        // PunctuateText takes text with no- or limited- punctuation and returns
        // the same text with corrected punctuation and capitalization.
        rpc PunctuateText(TextTransformRequest) returns (TextTransformResponse) {}

        // NaturalQuery is a search function that enables querying one or more documents
        // or contexts with a query that is written in natural language.
        rpc NaturalQuery(NaturalQueryRequest) returns (NaturalQueryResponse) {}
    }

    // NLPModelParams is a metadata message that is included in every request message
    // used by the Core NLP Service and is used to specify model characteristics/requirements
    message NLPModelParams {
        // Requested model to use. If unavailable, the request will return an error
        string model_name = 1;
    }

    // TextTransformRequest is a request type intended for services like TransformText
    // which take an arbitrary text input
    message TextTransformRequest {
        // Each repeated text element is handled independently for handling multiple
        // input strings with a single request
        repeated string text = 1;
        uint32 top_n = 2; //
        NLPModelParams model = 3;
    }

    // TextTransformResponse is returned by the TransformText method. Responses
    // are returned in the same order as they were requested.
    message TextTransformResponse {
        repeated string text = 1;
    }

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

    // Classification messages return a class name and corresponding score
    message Classification {
        string class_name = 1;
        float score = 2;
    }

    // Span of a particular result
    message Span {
        uint32 start = 1;
        uint32 end = 2;
    }

    // ClassificationResults contain zero or more Classification messages
    // If the number of Classifications is > 1, top_n > 1 must have been
    // specified.
    message ClassificationResult {
        repeated Classification labels = 1;
    }

    // TextClassResponse is the return message from the ClassifyText service.
    message TextClassResponse {
        repeated ClassificationResult results = 1;
    }

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

    // TokenClassValue is used to correlate an input token with its classification results
    message TokenClassValue {
        string token = 1;
        repeated Classification label = 2;
        repeated Span span = 3;
    }

    // TokenClassSequence is used for returning a sequence of TokenClassValue objects
    // in the original order of input tokens
    message TokenClassSequence {
        repeated TokenClassValue results = 1;
    }

    // TokenClassResponse returns a single TokenClassSequence per input request
    message TokenClassResponse {
        repeated TokenClassSequence results = 1;
    }

    // AnalyzeIntentContext is reserved for future use when we may send context back in a
    // a variety of different formats (including raw neural network hidden states)
    message AnalyzeIntentContext {
        // Reserved for future use
    }

    // AnalyzeIntentOptions is an optional configuration message to be sent as part of
    // an AnalyzeIntentRequest with query metadata
    message AnalyzeIntentOptions {
        // Optionally provide context from previous interactions to bias the model's prediction
        oneof context {
            string previous_intent = 1;
            AnalyzeIntentContext vectors = 2;
        }
        // Optional domain field. Domain must be supported otherwise an error will be returned.
        // If left blank, a domain detector will be run first and then the query routed to the
        // appropriate intent classifier (if it exists)
        string domain = 3;

        // Optional language field. Assumed to be "en-US" if not specified.
        string lang = 4;
    }

    // AnalyzeIntentRequest is the input message for the AnalyzeIntent service
    message AnalyzeIntentRequest {
        // The string to analyze for intent and slots
        string query = 1;
        // Optional configuration for the request, including providing context from previous turns
        // and hardcoding a domain/language
        AnalyzeIntentOptions options = 2;
    }

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

    // AnalyzeEntitiesOptions is an optional configuration message to be sent as part of
    // an AnalyzeEntitiesRequest with query metadata
    message AnalyzeEntitiesOptions {
        // Optional language field. Assumed to be "en-US" if not specified.
        string lang = 4;
    }

    // AnalyzeEntitiesRequest is the input message for the AnalyzeEntities service
    message AnalyzeEntitiesRequest {
        // The string to analyze for intent and slots
        string query = 1;
        // Optional configuration for the request, including providing context from previous turns
        // and hardcoding a domain/language
        AnalyzeEntitiesOptions options = 2;
    }

    message NaturalQueryRequest {
        // The natural language query
        string query = 1;

        // Maximum number of answers to return for the query. Defaults to 1 if not set.
        uint32 top_n = 2;

        // Context to search with the above query
        string context = 3;
    }

    message NaturalQueryResult {
        // text which answers the query
        string answer = 1;
        // Score representing confidence in result
        float score = 2;
    }

    message NaturalQueryResponse {
        repeated NaturalQueryResult results = 1;
    }

.. _riva_tts_proto:

riva_tts.proto
--------------

.. code-block:: proto
    :linenos:

    // Copyright (c) 2021, NVIDIA CORPORATION.  All rights reserved.
    //
    // NVIDIA CORPORATION and its licensors retain all intellectual property
    // and proprietary rights in and to this software, related documentation
    // and any modifications thereto.  Any use, reproduction, disclosure or
    // distribution of this software and related documentation without an express
    // license agreement from NVIDIA CORPORATION is strictly prohibited.


    syntax = "proto3";

    package nvidia.riva.tts;

    option cc_enable_arenas = true;
    option go_package = "nvidia.com/riva_speech";

    import "riva_audio.proto";

    service RivaSpeechSynthesis {
        // Used to request speech-to-text from the service. Submit a request containing the
        // desired text and configuration, and receive audio bytes in the requested format.
        rpc Synthesize(SynthesizeSpeechRequest) returns (SynthesizeSpeechResponse) {}

        // Used to request speech-to-text returned via stream as it becomes available.
        // Submit a SynthesizeSpeechRequest with desired text and configuration,
        // and receive stream of bytes in the requested format.
        rpc SynthesizeOnline(SynthesizeSpeechRequest) returns (stream SynthesizeSpeechResponse) {}
    }

    message SynthesizeSpeechRequest {
        string text = 1;
        string language_code = 2;
        // audio encoding params
        AudioEncoding encoding = 3;
        int32 sample_rate_hz = 4;
        // voice params
        string voice_name = 5;
    }

    message SynthesizeSpeechResponse {
        bytes audio = 1;
    }

    /*
    *
    */

.. _riva_audio_proto:

riva_audio.proto
----------------

.. code-block:: proto
    :linenos:

	// Copyright (c) 2021, NVIDIA CORPORATION.  All rights reserved.
	// NVIDIA CORPORATION and its licensors retain all intellectual property
	// and proprietary rights in and to this software, related documentation
	// and any modifications thereto.  Any use, reproduction, disclosure or
	// distribution of this software and related documentation without an express
	// license agreement from NVIDIA CORPORATION is strictly prohibited.


	syntax = "proto3";

	package nvidia.riva;

	option cc_enable_arenas = true;


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

health.proto
--------------

.. code-block:: proto
    :linenos:

	// Copyright (c) 2021, NVIDIA CORPORATION.  All rights reserved.
	//
	// NVIDIA CORPORATION and its licensors retain all intellectual property
	// and proprietary rights in and to this software, related documentation
	// and any modifications thereto.  Any use, reproduction, disclosure or
	// distribution of this software and related documentation without an express
	// license agreement from NVIDIA CORPORATION is strictly prohibited.


	//
	//Based on gRPC health check protocol - more details found here:
	//https://github.com/grpc/grpc/blob/master/doc/health-checking.md
	//

	syntax = "proto3";
	option go_package = "nvidia.com/riva_speech";

	package grpc.health.v1;


	option cc_enable_arenas = true;

	service Health {
        rpc Check(HealthCheckRequest) returns (HealthCheckResponse);
        rpc Watch(HealthCheckRequest) returns (stream HealthCheckResponse);
	}

	message HealthCheckRequest {
	    string service = 1;
	}

	message HealthCheckResponse {
        enum ServingStatus {
            UNKNOWN = 0;
            SERVING = 1;
            NOT_SERVING = 2;
        }
        ServingStatus status = 1;
	}


.. seealso::

    * `gRPC API <https://docs.nvidia.com/deeplearning/riva/user-guide/docs/development-grpc.html>`_
    * `Python API <https://docs.nvidia.com/deeplearning/riva/user-guide/docs/development-python.html>`_
    * `gRPC & Protocol Buffers <https://docs.nvidia.com/deeplearning/riva/user-guide/docs/development-python.html>`_
