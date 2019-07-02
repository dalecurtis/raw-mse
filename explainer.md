# JSON/RAW MediaSource Byte Streams Explainer

# Introduction
The world of audio and video codecs and container formats is diverse. For reasons of security, licensing, size restrictions, and more browsers can't intrinsically support all codecs and formats for audio / video playback. However that doesn't mean we can't offer a way for developers to plug decoders and/or demuxers written in JavaScript or WebAssembly into our media pipelines.

We propose a new [Media Source Extensions byte stream format](https://www.w3.org/TR/mse-byte-stream-format-registry/) which allows injection of demuxed or raw audio and video frames.


# Use cases

* Current adapative streaming libraries are transmuxing from MPEG2-TS to ISOBMFF / MP4 to handle HLS playback using Media Source Extensions. This would allow them to skip the remxuing step.
* Experimental codecs could be launched as downloadable WebAssembly/JS packages that individual sites can iterate and experiment more quickly with.
* Developers could add support in WebAssembly/JS for obscure or otherwise unsupported codecs; E.g., one could imagine a web based version of ffmpeg's ffplay tool offering support for all of its codecs through this API.


# Proposed API / Example

```Javascript
  let video = document.createElement('video');
  let mse = new MediaSource();

  mse.addEventListener('sourceopen', function() {
    let audio = mse.addSourceBuffer('audio/raw; codecs="raw"');
    let video = mse.addSourceBuffer('video/raw; codecs="avc1.42E01E"');

    video.addEventListener('updateend', function() {
      video.appendBuffer('{"timestamp": 0, "duration": 10}' + VideoFrame);
    }, {once: true});

    audio.addEventListener('updateend', function() {
      audio.appendBuffer('{"timestamp": 0, "duration": 10}' + AudioFrame)
    }, {once: true});

    // Append initialization segments.
    video.appendBuffer(
        '{"width": 1280, "height": 720, "cs_primary": "BT709",' +
        '"cs_transfer": "BT709", "cs_matrix": "BT709"}')
    audio.appendBuffer(
        '{"channels": 2, "channel_layout": ["left", "right"],' +
        '"sample_rate": 48000}, format: "int16"');

    // Raw video example:
    // video.appendBuffer('{"width": 1280, "height": 720, "format": "yuv420"');
  }, {once: true});


  video.src = window.URL.createObjectURL(mse);
```


# Open Questions / Notes / Links
* Should we instead have SourceBuffer.appendFrame(json, raw_data)?
