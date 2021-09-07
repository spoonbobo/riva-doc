Riva - Python Client
====================

asr_client.py
-------------

.. code-block:: python

    import argparse
    import wave
    import sys
    import grpc
    import time
    import riva_api.audio_pb2 as ra
    import riva_api.riva_asr_pb2 as rasr
    import riva_api.riva_asr_pb2_grpc as rasr_srv


    def get_args():
        parser = argparse.ArgumentParser(description="Streaming transcription via Riva AI Services")
        parser.add_argument("--server", default="localhost:50051", type=str, help="URI to GRPC server endpoint")
        parser.add_argument("--audio-file", required=True, help="path to local file to stream")
        parser.add_argument(
            "--show-intermediate", action="store_true", help="show intermediate transcripts as they are available"
        )
        return parser.parse_args()


    def listen_print_loop(responses, show_intermediate=False):
        num_chars_printed = 0
        idx = 0
        for response in responses:
            idx += 1
            if not response.results:
                continue

            result = response.results[0]
            if not result.alternatives:
                continue

            transcript = result.alternatives[0].transcript

            if show_intermediate:
                overwrite_chars = ' ' * (num_chars_printed - len(transcript))

                if not result.is_final:
                    sys.stdout.write(">> " + transcript + overwrite_chars + '\r')
                    sys.stdout.flush()

                    num_chars_printed = len(transcript) + 3

                else:
                    print("## " + transcript + overwrite_chars + "\n")
                    num_chars_printed = 0
            else:
                if result.is_final:
                    print(f"## {transcript.encode('utf-8')}\n")
                    sys.stdout.buffer.write(transcript.encode('utf-8'))

    CHUNK = 1024
    args = get_args()
    wf = wave.open(args.audio_file, 'rb')

    channel = grpc.insecure_channel(args.server)
    client = rasr_srv.RivaSpeechRecognitionStub(channel)


    config = rasr.RecognitionConfig(
        encoding=ra.AudioEncoding.LINEAR_PCM,
        sample_rate_hertz=wf.getframerate(),
        language_code="en-US",
        max_alternatives=1,
        enable_automatic_punctuation=True,
    )
    streaming_config = rasr.StreamingRecognitionConfig(config=config, interim_results=True)

    # read data
    def generator(w, s):
        yield rasr.StreamingRecognizeRequest(streaming_config=s)
        d = w.readframes(CHUNK)
        while len(d) > 0:
            yield rasr.StreamingRecognizeRequest(audio_content=d)
            d = w.readframes(CHUNK)


    responses = client.StreamingRecognize(generator(wf, streaming_config))
    listen_print_loop(responses, show_intermediate=args.show_intermediate)

Riva ASR service
----------------

.. code-block:: bash

    python3 $RIVA_QS/asr_client.py --audio-file $path