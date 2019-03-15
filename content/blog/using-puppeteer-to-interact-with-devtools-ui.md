+++
title = "Using Puppeteer to interact with the Devtools UI (Hack)"
description = ""
tags = [
    "javascript",
    "puppeteer",
    "devtools",
    "chromium",
    "extension",
    "hack",
]
date = "2018-12-30"
highlight = "true"
aliases = [
    "/blog/using-puppeteer-to-interact-with-devtools-ui"
]
+++

# 📄 Context
Someone recently asked a question in a Puppeteer issue ["Can we use Puppeteer to interact with a Devtools panel?"](https://github.com/GoogleChrome/puppeteer/issues/3699). The user wanted to be able to test a Chrome Extension in the DevTools UI. There is already a way of testing the background page, by grabbing the page associated with the `background_page` target. See [Working with Chrome Extensions](https://pptr.dev/#?product=Puppeteer&version=v1.11.0&show=api-working-with-chrome-extensions) for an example.

However, there is no API for interacting with the DevTools UI. Puppeteer uses the low level [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) to interact with the browser and pages.

# 🔧 DevTools UI vs DevTools Protocol

> The Chrome DevTools Protocol allows for tools to instrument, inspect, debug and profile Chromium, Chrome and other Blink-based browsers. Many existing projects currently use the protocol. The Chrome DevTools uses this protocol and the team maintains its API.

**Source:** https://chromedevtools.github.io/devtools-protocol/

The UI part of DevTools is a separate project, and so there are no low level APIs exposed for interacting with it. The important thing to understand is that the DevTools UI is just like any other web app, which means we can load it up in a browser tab and interact with it with Puppeteer. 

## Loading up static DevTools UI with Puppeteer
To start with, we will use Puppeteer to load the DevTools UI in a new tab.

```javascript
const puppeteer = require('puppeteer');

(async() => {
    // launch with DevTools, which enables debugging capabilities
    const browser = await puppeteer.launch({ devtools: true });
    const targets = await browser.targets();

    // find DevTools target URL
    const devtoolsUrl = targets
        .map(({ _targetInfo }) => _targetInfo.url)
        .find((url) => url.indexOf('chrome-devtools://') !== -1);
    
    // load the DevTools page in a new tab
    const page = await browser.newPage();
    await page.goto(devtoolsUrl);
})();
```

<img alt="Static DevTools UI" src="/img/blog/devtools/static-ui.png" />

This might look confusing at first. On the left-hand side is the DevTools page we loaded, and on the right-hand side is the attached DevTools. This means we can use the DevTools on the right to inspect the DevTools UI on the left. Crazy stuff 😲.

You can do this in a normal browser by undocking DevTools and then pressing <kbd>Cmd</kbd> + <kbd>I</kbd>.

This is all cool and it's useful for debugging problems with the UI itself, but we have no data. What we need is to connect the UI up to the page via the DevTools Protocol. The way this is done is to connect a WebSocket to the backend. The UI and the backend will send and receive messages using JSON, and it's these messages that allow DevTools UI to interact with a page.

## Attaching DevTools UI to inspected page with Puppeteer
```javascript
const puppeteer = require('puppeteer');

(async() => {
    // launch with DevTools, which enables debugging capabilities
    const browser = await puppeteer.launch({ devtools: true });

    const wsEndpoint = browser.wsEndpoint();

    // the page I want to debug
    const myPage = await browser.newPage();
    const pageId = myPage.target()._targetId;

    // use the host:port that Chromium provided, but replace the browser endpoint with the page to inspect
    const pageTargeUrl = `${wsEndpoint.replace('ws://', '').match(/.*(?=\/browser)/)[0]}/page/${pageId}`;
                
    // build the full debugging url for the page I want to inspect
    const pageDebuggingUrl = `chrome-devtools://devtools/bundled/devtools_app.html?ws=${pageTargeUrl}`;

    // open the debugging UI in a new tab that Puppeteer can interact with
    const devtoolsPage = await browser.newPage();
    await devtoolsPage.goto(pageDebuggingUrl);

    // navigate to the page now so that we start capturing data in the debugger UI
    await myPage.goto('http://news.bbc.co.uk');
    
    // the installed extension may open a new tab so make sure we select the debugger UI tab
    await devtoolsPage.bringToFront();
})();
```

### 🔎 How it works
Firstly we grabbed the WebSocket endpoint that Chromium created when we launched the browser. This will output something like `ws://127.0.0.1:50037/devtools/browser/cd4a63c8-c18e-4a4d-be66-552ce629236a`. 

This is the `browser` endpoint, but there is one exposed for the `page` we want to inspect. We obtain the identifier by getting the `_targetId` from our Puppeteer page.

Finally, we build the full debugging URL by providing the page WebSocket endpoint as an argument to the bundled DevTools app.

If we then navigate to the BBC News website, the DevTools UI will receive data about the page.

<img alt="Dynamic DevTools UI" src="/img/blog/devtools/dynamic-ui.png" />

# Interacting with a Chrome Extension panel
By modifying the code above, we can interact with a Chrome Extension panel. This requires just a couple of changes:

1. Modify the launch arguments reference the unpacked extension code. 
2. Select the Adblock Plus tab in the UI. 

This is somewhat hacky, but what this does is continuously press <kbd>Cmd</kbd> + <kbd>]</kbd> until the the right panel is selected. Note the `[aria-selected="true"]` attribute selector on the tab. 

```javascript
const puppeteer = require('puppeteer');

(async() => {
    // launch with DevTools, which enables debugging capabilities
    // and load the Adblock extension (assumes the unpacked extension is in a folder called `adblock-unpacked`)
    const browser = await puppeteer.launch({ 
        devtools: true,
        args: [
            '--disable-extensions-except=adblock-unpacked',
            '--load-extension=adblock-unpacked/',
          ]
    });

    // ... ommitted code

    // use Cmd + ] shortut to move across tabs until we're on the the Adblock Plus tab
    let selectedAdblockTab = null;

    while (!selectedAdblockTab) {
        await devtoolsPage.keyboard.down('MetaLeft');
        await devtoolsPage.keyboard.press(']');
        await devtoolsPage.keyboard.up('MetaLeft');

        selectedAdblockTab = await devtoolsPage.evaluate(() => {
            return document.querySelector('#-blink-dev-tools > div.widget.vbox.root-view > div > div > div').shadowRoot.querySelector('#tab-chrome-extension\\\:\\\/\\\/cfhdojbkjhnklbpkdaibdccddilifddbAdblockPlus[aria-selected="true"]');
        })
    }
})();
```

If you are wondering why the selector is really long and complicated, it's because DevTools UI uses Shadow DOM. It does make it difficult selecting elements from an outer scope (for good reasons). However, Chromium 72 added the 'Copy JS path' option to generate a selector statement for selected element. This makes this simpler.

<img alt="Copy JS path" width="500px"  src="/img/blog/devtools/copy-js-path.jpg" />

## 😞 Problem
Even though we are getting new network requests etc. in the main DevTools UI, we are not getting any blocked requests in the Adblock extension panel. Something has gone wrong.

I think the problem is that Adblock attaches to a particular tab, and since we are running the debugger in a new tab, it's not going to receive messages about the other tab. Whether this is a problem for all extensions or just some I don't know. It looks as though the communication mechanism with extensions is complex. See [Extending DevTools](https://developer.chrome.com/extensions/devtools) for details.

At this point, I gave up trying to get it to work, as it's a whole new area for me. I will likely revisit this when I know more about Chrome Extensions - a new excuse to create my own.
