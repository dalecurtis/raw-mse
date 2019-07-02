# JSON/RAW MediaSource Byte Streams Explainer

# Introduction
The world of audio and video codecs and container formats is diverse. For reasons of security, licensing, size restrictions, and more browsers can't intrinsically support all codecs and formats for audio / video playback. However that doesn't mean we can't offer a way for developers to plug decoders and/or demuxers written in JavaScript or WebAssembly into our media pipelines.

We propose a new [Media Source Extensions byte stream format](https://www.w3.org/TR/mse-byte-stream-format-registry/) and method, SourceBuffer.appendFrames(), which allows injection of demuxed or raw audio and video frames.


# Use cases

* Current adapative streaming libraries are transmuxing from MPEG2-TS to ISOBMFF / MP4 to handle HLS playback using Media Source Extensions. This would allow them to skip the remxuing step.
* Experimental codecs could be launched as downloadable WebAssembly/JS packages that individual sites can iterate and experiment more quickly with.
* Developers could add support in WebAssembly/JS for obscure or otherwise unsupported codecs; E.g., one could imagine a web based version of ffmpeg's ffplay tool offering support for all of its codecs through this API.


# Proposed API / Example

```Javascript
  let video = document.createElement('video');
  let mse = new MediaSource();

  mse.addEventListener('sourceopen', function() {
    let audio = mse.addSourceBuffer('audio/json; codecs="raw"');
    let video = mse.addSourceBuffer('video/json; codecs="avc1.42E01E"');

    video.addEventListener('updateend', function() {
      video.appendFrames(
        '{"frames": [{"timestamp": 0, "duration": 10}]', videoData);
    }, {once: true});

    audio.addEventListener('updateend', function() {
      audio.appendFrames(
        '{"frames": [{"timestamp": 0, "duration": 10}]', audioData)
    }, {once: true});

    // Append initialization segments.
    video.appendFrames(
        '{"config": {"width": 1280, "height": 720, "cs_primary": "BT709",' +
        '"cs_transfer": "BT709", "cs_matrix": "BT709"}}');
    audio.appendFrames(
        '{"config": {"channels": 2, "channel_layout": ["left", "right"],' +
        '"sample_rate": 48000}, format: "int16"}}');

    // Raw video example:
    // video.appendFrames('{"width": 1280, "height": 720, "format": "yuv420"');
  }, {once: true});

  video.src = window.URL.createObjectURL(mse);
```


# Open Questions / Notes / Links
* Originally this proposal reused appendBuffer(), but there's no idiomatic way to convert strings int ArrayBuffers so it felt gross.
* Is there a better name than appendFrames() since we're also appending the initialization segment?
* Should the mime-type by "\*/json" or "\*/raw"?
* TODO: Specify format options and structure.
