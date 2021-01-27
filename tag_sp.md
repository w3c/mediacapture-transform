# TAG Review: Security and Privacy questionnaire

### 2.1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?
This API exposes raw audio and video frames from MediaStreams and metadata associated with them. The API also exposes a way to create MediaStream tracks containing arbitrary audio and video data which can be sent to any MediaStream sink (e.g., media elements, WebRTC peer connections, media recorders, Web Audio). This exposure is required to allow processing of raw audio and video data, which is the main use case this API intends to support. The media data exposed by this API is already exposed to the web in less ergonomic ways by other APIs. For example, video frames can be exposed by playing the MediaStream on a media element, and drawing the element into a canvas; audio frames can be exposed by sending them to WebAudio. Similarly, the ability to write arbitrary data to a MediaStream sink already exists, since it is possible to capture the data from a canvas or Web Audio into a MediaStream that can be sent to any sink. The main difference is that the proposed API provides a more ergonomic way to use the media data.
In addition to raw media frames, this API exposes a small number of control signals between MediaStream sinks, tracks and sources in order to allow them to better adapt to operating conditions. The data contained in the signals is not sensitive, but they might provide some limited insight into how a system is working. We have chosen to expose only signals that do not expose details about the internal configuration of the system, but are still useful for custom sinks and sources.
### 2.2. Do features in your specification expose the minimum amount of information necessary to enable their intended uses?
Yes. The need to expose media frame data is obvious based on the intended use case (raw media processing). With regards to control signals, the exposure is needed to allow custom sinks and sources to better interact with the system-provided sources and sinks.
### 2.3. How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?
No extra PII is exposed by this feature beyond what MediaStreams already expose.
### 2.4. How do the features in your specification deal with sensitive information?
No extra sensitive information is exposed by this API.
### 2.5. Do the features in your specification introduce new state for an origin that persists across browsing sessions?
No.
### 2.6. Do the features in your specification expose information about the underlying platform to origins?
No.
### 2.7. Do features in this specification allow an origin access to sensors on a user’s device
No.
### 2.8. What data do the features in this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.
As mentioned above, this API exposes raw media data in a manner that makes it easy to do media processing. MediaStreamTracks that are tainted with another origin cannot be accessed with this API, as that would break the isolation rule.
### 2.9. Do features in this specification enable new script execution/loading mechanisms?
No.
### 2.10. Do features in this specification allow an origin to access other devices?
No.
### 2.11. Do features in this specification allow an origin some measure of control over a user agent’s native UI?
No.
### 2.12. What temporary identifiers do the features in this specification create or expose to the web?
None.
### 2.13. How does this specification distinguish between behavior in first-party and third-party contexts?
MediaStreamTracks that are tainted with another origin cannot be accessed with this API, as that would break the isolation rule.
### 2.14. How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?
No difference.
### 2.15. Does this specification have both "Security Considerations" and "Privacy Considerations" sections?
Yes.
### 2.16. Do features in your specification enable downgrading default security characteristics?
No.
### 2.17. What should this questionnaire have asked?
The questions seem adequate.

