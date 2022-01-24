# Audio in mediacapture-transform

This document contains arguments for including audio processing in the Breakout
Box mechanism, and preserves pieces of text that have been removed from the spec
because there is no WG consensus on including audio.

# Spec changes needed

Include &lt;audio&gt; tags as a possible destination

Under "Use cases supported", include:

- *Audio processing*: This is the equivalent of the video processing use case, but for audio tracks. This use case overlaps partially with the {{AudioWorklet}} interface, but the model provided by this specification differs in significant ways:
    - Pull-based programming model, as opposed to {{AudioWorklet}}'s clock-based model. This means that processing of each single block of audio data does not have a set time budget.
    - Offers direct access to the data and metadata from the original {{MediaStreamTrack}}. In particular, timestamps come directly from the track as opposed to an {{AudioContext}}.
    - Easier integration with video processing by providing the same API and programming model and allowing both to run on the same scope.
    - Does not run on a real-time thread. This means that the model is not suitable for applications with strong low-latency requirements.

    These differences make the model provided by this specification more
    suitable than {{AudioWorklet}} for processing that requires more tolerance
    to transient CPU spikes, better integration with video
    {{MediaStreamTrack}}s, access to track metadata (e.g., timestamps), but
    not strong low-latency requirements such as local audio rendering.

    An example of this would be <a href="https://arxiv.org/abs/1804.03619">
    audio-visual speech separation</a>, which can be used to combine the video
    and audio tracks from a speaker on the sender side of a video call and
    remove noise not coming from the speaker (i.e., the "Noisy cafeteria" case).
    Other examples that do not require integration with video but can benefit
    from the model include echo detection and other forms of ML-based noise
    cancellation.
-  Under Multi-source processing, add: "Audio-visual speech separation, referenced above, is another case of multi-source processing."
- *Custom audio or video sink*: In this use case, the purpose is not producing a processed {{MediaStreamTrack}}, but to consume the media in a different way. For example, an application could use [[WEBCODECS]] and [[WEBTRANSPORT]] to create an {{RTCPeerConnection}}-like sink, but using different codec configuration and networking protocols.

Under 'MediaStreamTrackProcessor', include:

If the track is an audio track, the chunks will be {{AudioData}} objects.

Under "Security and Privacy considerations", include AudioData as an alternative
to VideoFrame.

