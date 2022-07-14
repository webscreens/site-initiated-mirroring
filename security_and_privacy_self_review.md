# Security & privacy self-review for the Site-Initiated Mirroring API

See [explainer.md](explainer.md) for the API's main explainer.

These questions are taken from
[this questionnaire](https://www.w3.org/TR/security-privacy-questionnaire).

## Authors

- Takumi Fujimoto [@takumif](https://github.com/takumif) |
  [<takumif@google.com>](mailto:takumif@google.com)

## 2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

A website would be able to query (get events fired containing a boolean) whether
the user has devices on the local network that support being a receiver for
site-initiated mirroring (e.g a Chromecast device). Note that it could already
query for devices that support existing uses of Presentation API (i.e. launching
a web app by URL), and given the capabilities of the receiver hardware currently
on the market, the information that can be obtained today already contains the
information that's newly exposed through site-initiated mirroring, although that
may change with the introduction of new receiver hardware with different
capability combinations, e.g. a receiver device that supports mirroring but not
URL presentations.

The exposure of this information is needed for a website to be able to tell when
it should show UI to allow users to initiate site-initiated mirroring -- it
wouldn't be desirable for the user to click through just to find out that they
have no compatible devices.

## 2.2 Do features in your specification expose the minimum amount of information necessary to enable their intended uses?

Yes. We don't expose the number or the name of the available devices.

## 2.3 How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?

Existing Presentation API reduces PII exposure by showing the available list of
devices (device names could contain people's names) in native UI and not
exposing them to the website. Site-initiated mirroring, which extends the
Presentation API, uses the same mechanism.

By having the user agent provide the receiver picker UI, it limits the set of
receivers that can show the mirrored contents (which can potentially contain
sensitive information) to those that it trusts.

## 2.4 How do the features in your specification deal with sensitive information?

See section 2.3 above.

## 2.5 Do the features in your specification introduce new state for an origin that persists across browsing sessions?

No.

## 2.6 Do the features in your specification expose information about the underlying platform to origins?

No.

## 2.7 Does this specification allow an origin to send data to the underlying platform?

No.

## 2.8 Do features in this specification enable access to device sensors?

No.

## 2.9 Do features in this specification enable new script execution/loading mechanisms?

No.

## 2.10 Do features in this specification allow an origin to access other devices?

Yes, the feature allows an origin to choose to mirror its top-level browsing
context to a device on the local network chosen by the user in a native device
picker UI.

## 2.11 Do features in this specification allow an origin some measure of control over a user agent’s native UI?

Yes, the proposed feature uses an existing flow of `PresentationRequest#start()`
triggering a native device picker UI, which can only be triggered with user
activation.

## 2.12 What temporary identifiers do the features in this specification create or expose to the web?

Starting a mirroring session creates a `PresentationConnection` object with
[a presentation identifier](https://w3c.github.io/presentation-api/#dfn-presentation-identifier),
but the ID can be obtained only at creation and the user agent can create an ID
that does not reveal any information about the user or the browsing state.

## 2.13 How does this specification distinguish between behavior in first-party and third-party contexts?

The existing Presentation API spec requires the user agent's native device
picker UI to show the origin that is requesting to present
([link](https://w3c.github.io/presentation-api/#user-interface-guidelines)), so
that the user would know what origin is requesting. The content that is mirrored
is the top-level browsing context regarding of which browsing context is
requesting.

## 2.14 How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?

It behaves the same way as it does for the regular browsing mode. A
site-initiated mirroring session is terminated when a browsing context is
terminated.

## 2.15 Does this specification have both "Security Considerations" and "Privacy Considerations" sections?

See the
[security and privacy considerations](https://github.com/webscreens/site-initiated-mirroring/blob/main/explainer.md#security-and-privacy-considerations)
section of the explainer.

## 2.16 Do features in your specification enable origins to downgrade default security protections?

No.

## 2.17 How does your feature handle non-"fully active" documents?

A non-"fully active" document cannot initiate or maintain a site-initiated
mirroring session:

- Initiating mirroring requires user activation, so it cannot be done by a
  non-"fully active" document.

- When the user navigates away from a document (i.e. when the document is no
  longer fully active) the mirroring session is terminated.

## 2.18 What should this questionnaire have asked?

N/A
