# JSON/RAW MediaSource Byte Streams Explainer

# Introduction
The world of audio and video codecs and container formats is diverse. For reasons of security, licensing, size restrictions, and more browsers can't intrinsically support all codecs and formats for audio / video playback. However that doesn't mean we can't offer a way for developers to plug decoders and/or demuxers written in JavaScript or WebAssembly into our media pipelines.

We propose a a couple of [Media Source Extensions byte stream format](https://www.w3.org/TR/mse-byte-stream-format-registry/) additions which will allow injection of demuxed or raw audio and video frames.


# Audio Proposal
For audio, we propose adding support for 'audio/wav' to handle raw audio. For non-raw audio data we already have 'audio/aac' and 'audio/mp3'.

# Video Proposal
For video, we propose adding support for 'video/raw' which would use a simple 23-byte header based on [IVF](https://wiki.multimedia.cx/index.php/IVF) to encapsulate both raw and encoded samples.

```
bytes 0-3    FourCC (e.g., 'VP80', 'I420', 'AV01', etc)
bytes 4-5    visible width in pixels
bytes 6-7    visible height in pixels
bytes 8-15   64-bit presentation timestamp
byte  16     color space primary id.
byte  17     color space transfer id.
byte  18     color space matrix id.
byte  19     color space full range flag.
bytes 20-23  size of frame in bytes (not including the header)
```

Sample type would be determined by the fourcc embedded in the header; e.g., I420 for planar YUV 4:2:0, P010 for planar 10bit YUV 4:2:0, [etc](https://docs.microsoft.com/en-us/windows/desktop/medfound/video-fourccs). For encoded data, we would use 'AV01', 'VP80', 'VP90', and 'H264'.

Color space values will come from the [AV1 codec specification](https://aomediacodec.github.io/av1-isobmff/#codecsparam).


# Use cases

* Current adapative streaming libraries are transmuxing from MPEG2-TS to ISOBMFF / MP4 to handle HLS playback using Media Source Extensions. This would allow them to skip the remxuing step.
* Experimental codecs could be launched as downloadable WebAssembly/JS packages that individual sites can iterate and experiment more quickly with.
* Developers could add support in WebAssembly/JS for obscure or otherwise unsupported codecs; E.g., one could imagine a web based version of ffmpeg's ffplay tool offering support for all of its codecs through this API.


# Proposed API / Example

```Javascript
  let video = document.createElement('video');
  let mse = new MediaSource();

  mse.addEventListener('sourceopen', function() {
    let audio = mse.addSourceBuffer('audio/wav');
    let video = mse.addSourceBuffer('video/ivf');

    video.addEventListener('updateend', function() {
      video.appendBuffer(videoData);
    }, {once: true});

    audio.addEventListener('updateend', function() {
      audio.timestampOffset = firstVideoPresentationTimestamp;
      audio.appendBuffer(pcmAudioData);
    }, {once: true});
  }, {once: true});

  video.src = window.URL.createObjectURL(mse);
```


# Open Questions / Notes / Links
* Is it folly to use fourcc codes for pixel formats? These don't seem standardized. ffmpeg has [one definition](https://cs.chromium.org/chromium/src/third_party/ffmpeg/libavcodec/raw.c?l=31) and Microsoft [another definition](https://docs.microsoft.com/en-us/windows/desktop/medfound/video-fourccs).
* Unfortunately opus and vorbis don't have specified MPEG1 packetizations -- only  ogg, mp4, and webm. So if developers want to use demuxed Opus or Vorbis it will need to be in an ISO-BMFF or webm container unless we add an Ogg demuxer to MSE.
