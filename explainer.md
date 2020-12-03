# Explainer - Insertable Streams for MediaStreamTracks

## Problem to be solved

We need an API for processing media that:
* Allows the processing to be specified by the user, not the browser
* Allows the processed data to be handled by the browser as if it came through
  the normal pipeline
* Allows the use of techniques like WASM to achieve effective processing
* Allows the use of techniques like Workers to avoid blocking on the main thread
* Does not negatively impact security or privacy of current communications


## Approach

This document builds on concepts previously proposed by
[Insertable Streams](https://w3c.github.io/webrtc-insertable-streams/), and applies them to the
MediaStreamTrack API in order to build an API that is:

* Familiar to existing MediaStreamTrack users
* Able to support insertion of user-defined components
* Able to support high performance user-specified transformations
* Able to support user defined component wrapping and replacement

The central idea is to expose the content of a MediaStreamTrack as a collection of
streams (as defined by the [WHATWG Streams API](https://streams.spec.whatwg.org/)),
which can be manipulated to introduce new components.


## Use cases

The use cases include:

* Funny Hats (processing inserted before encoding or after decoding)
* Background removal
* Voice processing

## Breakout Box Phases

We can see the Breakout Box as having three phases:
* Phase one: Closed, with no media or control access
* Phase two: Half-open, with only media being processed in JS
* Phase three: Fully open, with both media and signals being processed in JS.

At stage three, there
is no longer any connection between the stream generator and stream consumer that doesn't pass
through Javascript; one can then freely interconnect the streams in whatever fashion one
sees fit.

## Code Examples

Example of processing media before transmitting to a PeerConnection in Phase Two:
<pre>
originalTrack = navigator.getUserMedia(...)
myProcessor = new TransformStream(....)
processingTrack = new ProcessingMediaStreamTrack(originalTrack);
processingTrack.readable.pipeThrough(myProcessor).pipeTo(processingTrack.writable);
pc.addtrack(processingTrack);
</pre>

Example of processing media 

## API

The following are the IDL modifications proposed by this API.
Future iterations may add additional operations following a similar pattern.

<pre>
// Breakout Box Stage Two

interface ProcessingMediaStreamTrack : MediaStreamTrack {
    constructor(MediaStreamTrack source);
    attribute ReadableStream readable;  // Stream of VideoFrame or AudioFrame
    attribute WritableStream writable;  // Stream of VideoFrame or AudioFrame
};

// Breakout Box Stage Three

interface TrackProcessor : MediaStreamTrack {
    constructor(MediaStreamTrack source);
    attribute ReadableStream readable;  // Stream of VideoFrame or AudioFrame
    attribute WritableStream writable;  // Stream of ControlSignal
};

interface TrackGenerator : MediaStreamTrack {
    attribute WritableStream writable;  // Stream of VideoFrame or AudioFrame
    attribute ReadableStream readable;  // Stream of ControlSignal
};

dictionary ControlSignal {
  required ControlSignalName name;
  long width;
  long height;
  double frameRate;
  PixelFormat pixelFormat;
};

enum ControlSignalName {
  "stop",
  "mute",
  "unmute",
  "configure",
};
</pre>

Note that despite using the same names for the streams defined, the meaning is the opposite
to the same attributes in a TransformStream; these are interfaces you plug a TransformStream
into, not interfaces you use to transform something.

<b>Alternative approach</b>: use function calls for createWritable and createReadable that return
the created streams
.
This would give clearer semantics in the case where the stream is passed away to a worker; it
would then be natural that the stream (which has been neutered in the original scope) is no
longer available on the object

## Design considerations ##

This design is built upon the Streams API. This is a natural interface
for stuff that can be considered a "sequence of objects", and has an ecosystem
around it that allows some concerns to be handed off easily.

In particular:

* Sequencing comes naturally; streams are in-order entities.
* With the Transferable Streams paradigm, changing what thread is doing
  the processing can be done in a manner that has been tested by others.
* Since other users of Streams interfaces are going to deal with issues
  like efficient handover and WASM interaction, we can expect to leverage
  common solutions for these problems.

There are some challenges with the Streams interface:

* Queueing in response to backpressure isn't an appropriate reaction in a
  real-time environment. This can be mitigated at the sender by not queueing,
  preferring to discard frames or not generating them.
* How to interface to congestion control signals, which travel in the
  opposite direction from the streams flow. This is addressed in phase three
  by defining explicit access to feedback signals.
* How to integrate error signalling and recovery, given that most of the
  time, breaking the pipeline is not an appropriate action.
  
These things may be solved by use of non-data "frames" (in the forward direction),
by reverse streams of non-data "frames" (in the reverse direction), or by defining
new interfaces based on events, promises or callbacks.

Experimentation with the prototype API seems to show that performance is
adequate for real-time processing; the streaming part is not contributing
very much to slowing down the pipelines.

## Alternatives to Streams ##
One set of alternatives involve callback-based or event-based interfaces; those
would require developing new interfaces that allow the relevant WebRTC
objects to be visible in the worker context in order to do processing off
the main thread. This would seem to be a significantly bigger specification
and implementation effort.

Another path would involve specifying a worklet API, similar to the AudioWorklet,
and specifying new APIs for connecting encoders and decoders to such worklets.
This also seemed to involve a significantly larger set of new interfaces, with a
correspondingly larger implementation effort, and would offer less flexibility
in how the processing elements could be implemented.





