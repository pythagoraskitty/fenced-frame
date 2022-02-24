# Integration with the web platform
This document goes into the various ways in which fenced frame interacts with the platform. The web platform is massive and so we expect this document to be a running document with ongoing additions.

## Source attribute
The src attribute which defines the url of the fenced frame is subjected to various restrictons or no restrictions depending on the mode of the fenced frame. Refer the modes details [here](https://github.com/shivanigithub/fenced-frame/blob/master/explainer/modes.md) that discusses the src attribute for each of those.

## Origin
The origin of the fenced frame will be the regular origin that it got navigated to. For opaque src, the origin will be that of the url that the opaque urn was mapped to by the browser. Any origin-keyed storage and communication channels with other same-origin frames outside the fenced frame tree will be disallowed by using a partitioning key with a nonce. The storage key will use the same nonce for the nested iframes and the root fenced frame, so that same-origin channels can still work within the fenced frame tree. Essentially, along with the storage partitioning effort, the storage and broadcast channel access will be keyed on <fenced-frame-root-site, fenced-frame-root-origin, nonce> for the root frame and <fenced-frame-root-site, nested-iframe-origin, nonce> for a nested iframe. More details related to this can be found [here](https://github.com/shivanigithub/fenced-frame/blob/master/explainer/storage_cookies_network_state.md).

## Size
To avoid the size attribute being used to communicate user identifying information from the embedding context to the fenced frame, this will be limited to only a few values. E.g. In the opaque-ads mode, some of the popular values that are (relevant for ads)[https://support.google.com/admanager/answer/1100453?hl=en]. We are also considering allowing some of these sizes to be flexible based on the viewport width on mobile.
Note that since size is a channel, these ads cannot be resized by the publisher.
TODO: The actual sizes that will be used for the "opaque-ads" fenced frame is yet to be determined and published here.

## Viewability
Viewability events using APIs like [Intersection Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) can be a communication channel from the embedding context to the fenced frame. Viewability events are important information in the ads workflow for conversion reports and payments and would be made available in FLEDGE and other ads use cases via separate mechanisms. In the first origin trial of fenced frames, intersection observer will be fully supported and it will be phased out only when the alternative mechanism is launched.

## Visibility
APIs like [visibilityState](https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilityState) and the corresponding [visibilityChange event](https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilitychange_event) determine the visibility of the tab based on user actions like backgrounding etc. A fenced frame and the main page could join server-side on the basis of when they lose and gain visibility. This kind of joining is also possible with other joining ids like the fenced frame creation time. The network side channel and future mitigations are described more [here](https://github.com/shivanigithub/fenced-frame/blob/master/explainer/network_side_channel.md).

## Accessibility
For accessibility, fenced frames will behave like iframes, they will be part of the accessibility tree and a11y users can navigate through its contents with a screen reader.
“title” is an attribute read by screen readers for an iframe and that will continue to be used for fenced frames. A script inside a fenced frame cannot access the title attribute via the same-origin channel window.frameElement.title since frameElement is null for fenced frames.

## Interactivity
Fenced frames would have normal user-interactivity as an iframe would.

## Focus
Since fenced frames require interactivity, they would need to get system focus. It could be a privacy concern in the following ways:
* A fenced frame and the main page could join server-side on the basis of when one lost focus and another gained focus. The network side channel and future mitigations are described more [here](https://github.com/shivanigithub/fenced-frame/blob/master/explainer/network_side_channel.md) 
* For programmatic focus, calling element.focus() might invoke blur() on an element that is in the embedding page thus making a channel from the fenced frame to the embedding page. So the mitigations there would be:
  * Only allow programmatic focus()/blur() to be successful if the fenced frame already has system focus.
  * Only allow system focus to move to a FF on a user gesture including hitting the tab key and not because of calling focus().

## Communication via PostMessage
A fenced frame does not allow communication via PostMessage between the embedding and embedded contexts. A fenced frame will be in a separate browsing context group then its embedding page. 

## Session History
Fenced frames can navigate but their history is not part of the browser back/forward list as that could be a communication channel from the fenced frame to the embedding page. Additionally, fenced frames will always have a replacement-only navigation (back/forward list of length 1) which is a simpler model since it doesn't imply that there's a hidden list of back/forward entries for the nested page, only accessible via history APIs and not via the back/forward buttons, etc. This is also consistent with the iframes new opt-in mode for disjoint session history as discussed in https://github.com/whatwg/html/issues/6501.

## Content Security Policy
Fenced frames ineractions with CSP are detailed [here](https://github.com/shivanigithub/fenced-frame/blob/master/explainer/interaction_with_content_security_policy.md).

## Permissions and policies
This is discussed in more detail [here](https://docs.google.com/document/u/1/d/16PNR2hvO2oo93Mh5pGoHuXbvcwicNE3ieJVoejrjW_w/edit).

## COOP and COEP
Fenced frames will by design be in a separate browsing context group from its embedding page, so this implicitly implies they have strict [COOP value](https://docs.google.com/document/d/1zDlfvfTJ_9e8Jdc8ehuV4zMEu9ySMCiTGMS9y0GU92k/edit) of "same-origin". In fact even if both the embedding page and fenced frame are same-origin, fenced frames are placed in separate browsing context groups for consistency. Note that even though fenced frames are nested documents, they are treated as top-level browsing contexts and it is therefore important to understand their interaction with these headers.
If the fenced frame’s embedding page enables COEP then the fenced frame document should allow itself to be embedded as described [here](https://docs.google.com/document/d/1zDlfvfTJ_9e8Jdc8ehuV4zMEu9ySMCiTGMS9y0GU92k/edit#bookmark=id.kaco6v4zwnx2).

## Opt-in header
Since fenced frames allow a document to have many constraints in place, an opt-in mechanism is a good way for the document to accept those restrictions. The opt-in will make use of the Supports-Loading-Mode proposed [here](https://github.com/WICG/nav-speculation/blob/main/opt-in.md).

## Browser features
There are many chrome browser features e.g. autofill, translate etc., that will need to be considered on a case-by-case basis as to how they would interact with a fenced frame with possible approaches being:
* Treat the feature as if it would behave in a cross-origin iframe.
* Treat the feature as if fenced frame was a separate top-level page.
* Do not support the feature at all.

## Extensions
It is important for fenced frames to be visible to extensions especially content blockers since many of them block advertisements. 
In terms of non-content blocker extensions, they should also be able to interact with fenced frames as they would with another page. Note that malicious extensions might be able to override the privacy protections here by creating cross-site identifiers but that threat is larger than just fenced frames and would need to be handled as a separate effort.

## Developer tools
Fenced frames should have developer tools access as an iframe would.




