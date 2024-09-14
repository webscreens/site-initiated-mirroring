# Site Initiated Mirroring API

## Authors

mark a. foltz [@markafoltz](https://github.com/markafoltz)

[<mfoltz@google.com>](mailto:mfoltz@google.com)

Takumi Fujimoto [@takumif](https://github.com/takumif)

[<takumif@google.com>](mailto:takumif@google.com)

## Participate

- [Latest Draft](https://github.com/webscreens/site-initiated-mirroring/blob/main/explainer.md)
- [GitHub Repository](https://github.com/webscreens/site-initiated-mirroring)
- [File an Issue](https://github.com/webscreens/site-initiated-mirroring/issues)

## Introduction

We're proposing an API to allow Web pages to request mirroring themselves to a
secondary display.  They will be allowed to request customization of the
mirroring implementation in certain respects.

A secondary display is primarily a device on the local network connected
wirelessly through technologies such as Chromecast or AirPlay, but may also be
physically connected, such as through HDMI. Devices outside the LAN (i.e.
accessible via the Internet) are outside the scope of this spec.

There exist two APIs to allow Web sites to share content with a seconary
display.  The [Presentation API](https://w3c.github.io/presentation-api/) allows
the site to request the secondary display show an entirely new document.  The
[Remote Playback API](https://w3c.github.io/remote-playback/) allows a single
video element to be shown on a secondary display.

We've found there are use cases for allowing a site to request that its existing
content be shown on another display.  This presentation mode is currently being
utilized through the Cast Web SDK by Google Slides and Google Meet.

As tab capture is now available to the Web through
[`MediaDevices.getDisplayMedia()`](https://w3c.github.io/mediacapture-screen-share/#dom-mediadevices-getdisplaymedia)
for WebRTC, it's logical to allow a site to share its existing content with
secondary displays as well.

## Goals

- Allow a site to share its existing rendered content (as rendered inside the
  browser's tab/window) with a secondary, remote display.
- Allow the site to hint to the browser whether it wants low-latency mirroring,
  or the browser's default latency setting.
- Allow the site to control whether to also capture the tab's audio while
  mirroring.
- Allow the capture settings to be changed while mirroring is on-going.
- Design the API in such a way that it can be extended to accommodate more
  mirroring and non-mirroring configurations in the future. See the stakeholder
  feedback section below for possible future extensions.

## Non-goals

- Support mirroring to wired displays.
- Support mirroring of browser windows, or frames or other subelements of a
  single page.
- Provide information about the display being mirrored to (dimensions, display
  capabilities, aspect ratio).
- Support fine-grained control over the mirroring implementation (codecs,
  bitrates, etc.).
- Allow the site to access captured content directly.

## Key scenarios

### Mirroring a videoconference to a Chromecast or similar device

The user wants to participate in a videoconference, but watch it on their TV.
The conference is configured for the "speaker view" so mirroring the current
layout provides a good experience for viewing.

They use a button in-page to initiate mirroring to their TV.

While passively participating in the videoconference, audio is also captured and
routed to the TV.  When the user wishes to speak and has an active microphone,
audio is switched to play out locally (to enable echo cancellation) and the
latency of the mirroring connection is reduced (to improve A/V sync).

### Presenting a slide deck to a smart projector

The user wishes to present a slide deck to a smart projector.  Using UI in the
page they switch the slide deck to presentation mode, and select the smart
projector as the target.  The presentation's audio and video are mirrored to the
smart projector.

## Detailed design discussion

This design option extends the [Presentation API](
https://w3c.github.io/presentation-api/). Other design alternatives are
discussed below.

### Sample Code

```javascript
// The active mirroring session.
let connection = null;

const request = new PresentationRequest([{
  // A special string to indicate mirroring instead of a presentation receiver
  // URL, the exact string TBD.
  url: '_self',
  captureLatency: 'low',
  audioPlayback: 'receiver',
}]);

// Presentation availability works the same as other URLs.
// You can also set it as a default request in the top level frame.
navigator.presentation.defaultRequest = request;

// A button allowing initiation of site mirroring.
// Starting a presentation works the same as other URLS.
document.getElementById('mirrorBtn').onclick() = async function() {
  connection = await request.start();
};

document.getElementById('changeConfigBtn').onclick() = function() {
  const newParams = { latencyHint: 'low', audioPlayback: 'controller' };

  // Can only update connections that are starting or active.
  if (!(connection.state == 'connecting' ||
        connection.state == 'connected')) {
    return;
  }

  if (connection.url != '_self') {
    return;
  }

  // Update latency hint.
  if (connection.source.latencyHint
      connection.source.latencyHint != newParams.latencyHint) {
    connection.source.latencyHint = newParams.latencyHint;
  }

  // Update audio capture.
  if (connection.source.audioPlayback &&
      connection.source.audioPlayback != newParams.audioPlayback) {
    connection.source.audioPlayback = newParams.audioPlayback;
  }
}
```

### IDL

```
partial interface PresentationRequest {
  // This constructor replaces
  //   constructor(sequence<USVString> urls);
  // in the current Presentation API spec.
  constructor(sequence<(USVString or PresentationSource)> sources);
}

partial interface PresentationConnection {
  attribute PresentationSource source;
  [deprecated] readonly attribute USVString url;
};

dictionary PresentationSource {
  readonly USVString url;
  CaptureLatency? latencyHint = "default";
  AudioPlaybackDestination? audioPlayback = "receiver";
}

// Indicates the preferred tradeoff between low latency and smooth streaming.
// The actual behavior is implementation specific. Only applies to mirroring
// sources.
enum CaptureLatency {
  "default",
  // Having a smooth stream is prioritized over having a low latency.
  "high",
  // Having a low latency is prioritized over having a smooth stream.
  "low",
};

// Indicates where the audio playback should occur. Only applies to mirroring
// sources.
enum AudioPlaybackDestination { "receiver", "controller" };
```

### Open Questions

There are several open questions to address:

1. Whether navigation in the tab, within the same origin or to a different
   origin, should terminate the mirroring session. (#7)
1. Whether this API should be restricted to top-level frames. (#8)
1. Whether there's a use case to play audio out locally and remotely at the same
   time. (#9)
1. How to handle messaging. (#10)

## Security and privacy considerations

See [the corresponding section](
https://w3c.github.io/presentation-api/#security-and-privacy-considerations) of
the Presentation API spec.

The section on private browsing modes does not apply because there is no
presentation page that is in a separate browsing context. The section on
messaging also does not apply because there is no presentation page to receive
messages.

Also see the
[security and privacy self-review page](security_and_privacy_self_review.md).

### Personally identifiable information

By [monitoring the list of available presentation displays](
https://w3c.github.io/presentation-api/#monitoring-the-list-of-available-presentation-displays),
a website can determine whether the user has presentation displays compatible
with mirroring, which is one new bit of information that can be used for
fingerprinting. However, nearly all presentation displays support mirroring, so
this additional information does not add significantly to the fingerprinting
surface of the API.

A mirroring presentation URL is not a DIAL URL, so it cannot be used to probe
information about DIAL app support on presentation displays.

### Cross-origin and cross-device access

The proposed API will not grant access to the mirrored content to either
the controlling context initiating site initiated mirroring or other controlling
contexts that join (i.e. `reconnect()` to) an existing mirroring session, so no
new cross-origin or cross-device access to information is provided through the
proposed API.

## Considered alternatives

### Have one configuration for all the Presentation URLs

Another possible way to extend the Presentation API is to have one set of
parameters per PresentationRequest, as shown:

```
partial interface PresentationRequest {
  constructor(sequence<USVString url> urls, CaptureParameters captureParams);
}

dictionary CaptureParameters {
  CaptureLatency latencyHint = 'default';
  AudioPlaybackDestination audioPlayback = 'remote';
};
```

An upside of this approach is that the proposed constructor is more in line with
the existing ones. A downside is that this approach cannot be extended in the
future to accomodate per-URL configurations such as device capability
requirements.

### Extend the Remote Playback API

The [Remote Playback API](https://w3c.github.io/remote-playback/) is another
API that allows media contents to be played on a secondary display. Since it
currently only supports the playback of media elements, it would need to be
extended to allow the remote playback of entire tab contents.

Arguments in favor of extending the Remote Playback API:
1. Sending a mirroring stream is arguably conceptually closer to Remote
Playback than Presentation, given the latter is about setting up a separate
receiver page and bidirectional communication between the controller and the
receiver pages.
1. Configuring latency and audio playback does not apply to the existing use of
Presentation API, but it may for some implementations of the [media remoting](
https://w3c.github.io/remote-playback/#dfn-media-remoting) case of Remote
Playback.

Arguments against it:
1. During Remote Playback, we halt the local playback when it gets
activated, so that'd be inconsistent with site initiated mirroring, in which
playback happens simultaneously on both displays.
1. Remote Playback is intended for media playback, and mirroring a page
is arguably conceptually different.

Some implications of extending the Remote Playback API:
1. It would mean that the user agent would likely want to let the page know
whenever it’s being tab mirrored (via `document.remote.onconnect`), even when
mirroring is initiated via the user agent's UI.
1. If both `document.remote.remote` and `navigator.presentation.defaultRequest`
are set, and the user agent's secondary display picker is shared between the
Remote Playback and Presentation APIs, one API would need to be chosen as the
default when the picker is shown through the user agent's UI (i.e. not through
`RemotePlayback#prompt()` or `PresentationRequest#start()`). We may want a
way for the page to indicate its order of preference. There has also been a
related discussion ([minutes](
https://www.w3.org/2019/09/16-webscreens-minutes.html#x04)) regarding using the
two APIs together.

#### Sample code
```javascript
// Mirroring can be initiated either via prompt() or via the UA's native UI.
document.querySelector('#mirrorBtn').addEventListener('click', () => {
  document.remote.prompt();
});

document.remote.addEventListener('connect', event => {
  if (event.target.captureParams) {
    document.remote.captureParams.latencyHint = 'low';
    document.remote.captureParams.audioPlayback = 'controller';
  }
});
```

#### IDL
```
partial interface Document {
  attribute RemotePlayback remote;
};

partial interface RemotePlayback {
  // CaptureParameters is the same as in the Presentation API proposal.
  attribute CaptureParameters? captureParams;
};
```

### Add to Window Placement

There is a separate API for [Window Placement](https://github.com/webscreens/window-placement/blob/master/EXPLAINER.md)
being incubated in the Second Screen Community Group.  However, support for placing windows on
remote displays is stated as a non-goal of that proposal.  It's also unclear if
mirroring existing windows is in scope for the Window Placement API.

If the scope of this API were expanded to include these additional capabilities,
it could be another way to support these use cases.

### Extend getDisplayMedia()

Because this API already allows sites to obtain the content of browser tabs, it
could possibly be extended to support these scenarios.  However, this would
currently require double prompting - first to allow tab capture, and then to
give permission to stream it to another display.

There is also no API that allows a MediaStream to be sent directly to a
secondary display (without going through WebRTC).  Because a MediaStream can
represent a variety of frame sources, it would be difficult to implement an API
that could successfully send an arbitrary MediaStream to a secondary display.

### Add a new API

Another alternative would be to add a new global API to control this behavior,
like `requestFullscreen()`.  It would end up duplicating many aspects of the
Presentation API.

## Stakeholder Feedback / Opposition

The following features have been requested by websites that currently implement
site initiated mirroring through the Cast Web SDK, some of which are currently
outside the scope of this explainer (i.e. listed as non-goals):

- Choosing whether the audio playback is done on the controller or the receiver
  device
- Limiting the available receivers to those with a specific set of capabilities
  (e.g. audio output, video output)
- Choosing whether to launch a presentation receiver page or site initiated
  mirroring based on the receiver capabilities
- Mirroring only a specific region (e.g. a DOM element) within a page
