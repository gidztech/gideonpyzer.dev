+++
title = "Lazy loading Zendesk saved Huddle 2.3MB of JavaScript"
description = "3rd party libraries and integrations are responsible for significant slowdowns in web app. We at Huddle were able to shed 2.3 MB of assets from the initial page load of our app by implementing lazy loading of the Zendesk Help widget"
tags = [
    "zendesk",
    "lazy loading",
    "performance",
    "third party"
]
date = "2019-07-13"
image = ""
highlight = "true"
+++

Third party libraries and integrations can have a significant impact on web application performance. [Third Party Web](https://github.com/patrickhulce/third-party-web#summary) did an extensive analysis of the web's top 4 million sites to highlight the problems we are facing today.

> Across top ~4 million sites, ~2700 origins account for ~57% of all script execution time with the top 50 entities already accounting for ~47%. Third party script execution is the majority chunk of the web today, and it's important to make informed choices.

It's very easy to get into a situation where you're adding several scripts into your application to integrate with great services, and begin to degrade the performance of your application.

Assessing the performance impact of these services early on is really important. You might discover that you need to pick a more lightweight alternative, or perhaps build something in-house. Having said that, sometimes this is not a possibility.

# Background

During my innovation time at work, I have spent the last couple of months auditing and trying to improve our web application in regards to performance. Last month, I noticed that on our brand new Single Page Application (SPA), 47% of the assets being on page load came from Zendesk. This confused me because all we're doing is rendering a simple Help button.

This post will discuss how [Huddle](https://www.huddle.com/) implemented lazy loading of the Zendesk help widget to remove 2.3 MB of unnecessary JavaScript from page load.

# Zendesk Help Widget

![Help](/img/blog/lazy-zendesk/help.png)

At [Huddle](https://www.huddle.com/), we use Zendesk for our knowledge articles, and for live chatting with our support team. We include a snippet onto the page, which injects the web widget in an iframe. It's vital we continue to support this integraion, so removing it is not an option.

But the problem is that the initial page load of the application includes 2.3 MB of Zendesk assets from multiple origins. When I started looking at this, it was more like 3.1 MB, but that has since been reduced. There are two things to note here before we start to panic 🙀:

1. Assets that do not change will be served from browser cache (except cache-busting updates)
2. Assets are served asynchronously, so it doesn't block the initial rendering of the application

However, in the end, we are still parsing 2.3 MB of (after it's uncompressed) JavaScript code, which is significant. The average impact, according to [Third Party Web](https://github.com/patrickhulce/third-party-web#third-parties-by-total-impact), is 667 ms.

There's a bandwidth, CPU and memory cost involved, and particularly on slower devices, the widget could begin to affect the responsiveness of the application itself.

## When is it needed?

Consider you visit Huddle, our cloud-based document collaboration application. Do you expect to click on the Help button straight away to read our articles or chat to one of our support team? The answer is most likely no. In the case of a new user, we push our onboarding flow to help the user get started. For existing users, they will most likely go straight to the features they already use.

So why do we need to pull in all of this JavaScript when most users won't engage with Help for a given session. It's a complete waste of resources!

## How could Zendesk improve this?

They could ship minimal code just to render the button, and then lazy load the remaining dependencies when the button is clicked. This should (in theory) be straightforward to implement, but may require a change to their code snippet.

# My initial solution

I created a Help button in React that looked just like the original Zendesk one, and fixed the positioning where the normal button appears. The `onClick` handler invoked the Zendesk initialization code.

However, I discovered a problem. By doing this, all I did was bring in their iframe button on top of mine, and not actually open the main dialog. 😔

### Opening the dialog

[Web Widget API v1](https://developer.zendesk.com/embeddables/docs/widget/api) provides an `activate` function to trigger the opening of the dialog. But this didn't work because the widget was not always ready yet.

### Waiting for Zendesk to become ready

The widget documentation didn't appear to have an event tell us when the widget was ready to be interacted with. This resulted in me improvising and coming up with a hacky solution.

I looked at what changes in the DOM when Zendesk is ready and noticed that a `--active` modifier is applied to an element. When the iframe's `onLoad` event is fired, I called a function that used [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) to wait for the selector to exist before disconnecting it. I was also able hide the iframe button by setting a CSS style `visibility: hidden`.

```javascript
const waitForWidgetInitialised = async () =>
  new Promise(resolve => {
    const MutationObserver =
      window.MutationObserver || window.WebKitMutationObserver;
    if (MutationObserver) {
      const widgetObserver = new MutationObserver(() => {
        const iframe = document.querySelector('.zEWidget-launcher--active');
        if (iframe) {
          iframe.style.visibility = 'hidden';
          widgetObserver.disconnect();
          resolve(true);
        }
      });
      widgetObserver.observe(document.body, {
        childList: true,
        subtree: true,
        attributes: true
      });
    } else {
      // just fail gracefully since we don't support old browsers
      resolve(false);
    }
  });
```

This actually worked well, but I was not happy with the robustness of it. If Zendesk decided to change their DOM structure, my solution would break. Even with end-to-end tests and the ability to switch this solution off quickly, it's somewhat risky.

# Final solution

It turns out there was an event that you can hook into when the widget becomes ready. It wasn't documented in the main API, but it was mentioned in a [help article](https://support.zendesk.com/hc/en-us/articles/115007912068-Using-the-Chat-widget-JavaScript-API). There's also a Zopim API to hide the chat button instead of using the DOM directly. 🙂

```javascript
const waitForWidgetInitialised = async () =>
  new Promise(resolve => {
    zE(() => {
      $zopim(() => {
        $zopim.livechat.hideAll();
        resolve(true);
      });
    });
  });
```

All I need to do now is call `activate` afterwards to show the dialog. Since we own the button, we can even apply customer branding to it now! 🎉

# Considerations

This solution works really well but there's a few things to consider:

1. If the user changes page during a live chat, the session will end, because the assets are not fetched automatically anymore. This is easily solved by setting a flag in Session Storage after the button is clicked. If the flag exists on page load, we automatically load Zendesk.

2. The Help button normally is localized because it's hosted by Zendesk. By owning the button ourselves, we must localize the text for the languages we support.

3. Some customers blacklist certain assets for security compliance reasons. Currently this results in no Help button appearing, but with the new change, we will see a Help button that doesn't function. Perhaps in this case, we could simply show a user friendly message when the iframe's `onError` event is fired.

4. Zendesk maintains a WebSocket connection as soon as it's loaded. It may be possible to push certain things to the client to engage with online users. By only opening this connection when the user interacts with the button, we lose out on this potential engagement. Our support team clarified that we don't make use of this and so the limitation was justified for us.
