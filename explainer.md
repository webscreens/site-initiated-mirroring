# Tab Self Mirroring API

## Authors

mark a. foltz [@mfoltzgoogle](https://github.com/mfoltzgoogle)

[<mfoltz@google.com>](mailto:mfoltz@google.com)

## Participate

- [Latest Draft](https://mfoltzgoogle.github.io/tab-self-mirroring/explainer.html)
- [GitHub Repository](https://github.com/mfoltzgoogle/tab-self-mirroring)
- [File an Issue](https://github.com/mfoltzgoogle/tab-self-mirroring/issues)

## Introduction

We're proposing an API to allow Web pages to request mirroring themselves to a
secondary display.  They will be allowed to request customization of the
mirroring implementation in certain respects.

There exist two APIs to allow Web sites to share content with a seconary
display.  The first, the [Presentation
API](https://w3c.github.io/presentation-api/), allows the site to request the
secondary display show an entirely new document.  The second, the [Remote
Playback API](https://w3c.github.io/remote-playback/), allows a single video
element to be shown on a secondary display.

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

### Sample Code

```javascript
// The active mirroring session.
var theConnection = null;

// Define a PresentationRequest for mirroring.  Exact URL TBD.
var request = new PresentationRequest("about:self");

// Initial parameters for mirroring.
var initialParams = { latencyHint: 'default', audioPlayback: 'remote' };

// Presentation availability works the same as other URLs.
// You can also set it as a default request in the top level frame.
navigator.presentation.defaultRequest = request;
navigator.presentation.defaultRequest.onconnectionavailable = function(evt) {
    configureConnection(evt.connection);
};

// A button allowing initiation of site mirroring.
// Starting a presentation works the same as other URLS.
document.getElementById("mirrorBtn").onclick() = function() {
    request.start().then(connection =>
        configureConnection(connection, initialParams));
};

// Configure mirroring parameters on the connection, if supported.
function configureConnection(newConnection, params) {
  var theConnection = newConnection;
  // Can only update connections that are starting or active.
  if (!(theConnection.state == 'connecting' ||
        theConnection.state == 'connected')) {
      return;
  }

  // Feature detect if capture parameters are supported.
  if (!theConnection.captureParams) {
    return;
  }

  // Update latency hint.
  if (theConnection.captureParams.latencyHint
      theConnection.captureParams.latencyHint != params.latencyHint) {
      theConnection.captureParams.latencyHint = params.latencyHint;
  }

  // Update audio capture.
  if (theConnection.captureParams.audioPlayback &&
      theConnection.captureParams.audioPlayback != params.audioPlayback) {
      theConnection.captureParams.audioPlayback = params.audioPlayback;
  }
}

// Parameters can be changed while the connection state is "connected."
// TODO: Show an example of how that can be done.
```

## Detailed design discussion

### IDL

```
partial interface PresentationConnection {
  // Only non-null on a connection started by mirroring.
  attribute CaptureParameters captureParams = null;
};

dictionary CaptureParameters {
  CaptureLatency latencyHint = 'default';
  AudioPlaybackDestination audioPlayback = 'remote';
};

enum CaptureLatency { "default", "high", "low" };

enum AudioPlaybackDestination { "remote", "local" };
```

### Open Questions

There are several open questions to address:

1. Whether navigation in the tab, within the same origin or to a different origin,
should terminate the mirroring session.
1. Whether this API should be restricted to top-level frames.
1. Whether there's a use case to play audio out locally and remotely at the same
time.
1. Whether multiple URLs should be allowed in the PresentationRequest.
1. The choice of token or URL to indicate self-mirroring as a presentation source.
1. Whether captureParams should be available for other connection types (like 1-UA mode).
1. How to handle messaging.

## Considered alternatives

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

TBD

## References & acknowledgements

Many thanks for valuable feedback and advice from:

Takumi Fujimoto [<takumif@google.com>](mailto:takumif@google.com]

