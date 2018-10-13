+++
title = "Visual Regression Testing"
description = "Automation tests are usually concerned with the functional requirements of an application. This post will show you why this might not be enough to have peace of mind when deploying your web app UI projects to production."
tags = [
    "visual regression testing",
    "testing", 
    "jest",
    "mocha",
    "puppeteer",
    "docker",
    "phantomcss",
    "muppeteer",
    "jest-image-snapshot",
    "resemblejs",
    "pixelmatch",
    "huddle"
]
image = "visual-regression-testing/cat.png"
date = "2018-06-25"
highlight = "true"
+++

<img src="/img/blog/visual-regression-testing/cat.png" style="width: 600px; margin-top: 20px; border:0" />

Automation tests are usually concerned with the functional requirements of an application. This post will show you why this might not be enough to have peace of mind when deploying your web app UI projects to production.

# Introduction

There are several layers to web application testing, and they're applicable to both the front-end and the back-end. You can generally organize them into three major categories: Unit tests, integration tests and end-to-end tests.

<div style="text-align: center; margin-bottom: 18px">
<img style="width: 40%" src="/img/blog/visual-regression-testing/pyramid.png" alt="Testing pyramid" />
</div>

Unit tests can be a little misunderstood sometimes. It's quite common to see tests that look at a particular function or class. One might test that a service generates the correct output, given a particular set of inputs. However, a unit test doesn't have to be so low level. A unit is really just a particular behaviour.

Martin Fowler wrote an [interesting post](https://martinfowler.com/bliki/UnitTest.html) on this, and highlights the fact that a unit is a more broad term, and it's really up to the team to decide how to define it.

The reason I highlight this is because UI components can be tested as units. They provide a single behaviour to the system. Sure, internally they might do a few different functions, but I'm going to black-box these and instead test the component as a single entity.

Integration testing is more about looking at the interactions and data flow between multiple units. This might be two or more UI components performing a larger overall behaviour, such as an upload component updating a file list.

End-to-end tests look at the full user journey through the application. It's a way of testing that the requirements have been met from the end user's perspective. This is most often where people traditionally make use of browser automation technology, such as [Selenium](https://www.seleniumhq.org/) or [PhantomJS](http://phantomjs.org/).

However, browser automation can be used in **all** of these layers.

# Does the UI look right?

- You can test that particular DOM elements exist on the page when certain criteria are met, such as clicking a button and performing an action.
- You also might test that the router is correctly initializing the right page or component when a particular end-point is hit.

**Question: How do you test if the application actually looks right?**

**Answer:** One way is to manually look. We're usually quite good at spotting large regressions, such as colour changes or content overflowing on the page.

**Problems:**

- The application is large. There may be many different screens, and some will show under very specific, rare conditions (e.g. permission based)
- What if the application is responsive, and one needs to test multiple breakpoints for different devices and screen sizes?
- What if the tester simply misses something? They could miss both small and larger regressions, due to [Change Blindness](https://www.verywellmind.com/what-is-change-blindness-2795010).
- Time is a factor too. With all these different scenarios, it might simply be too time consuming for developers and QA to do a thorough job.

An automated mechanism for testing how the UI looks is needed. What if you could take a photo of the UI component before and after a breakage and compare the difference in a computery kind of way? _(Puts nerd hat on)_

<div style="text-align: center; margin-bottom: 18px">
<img style="border: 0; border-radius: 0" src="/img/blog/visual-regression-testing/nerd-face.png" alt="Nerd face" />
</div>

# Pixel Comparisons

Every image is made up of pixels, which are assigned a red, green and blue value. In a simple model, you could take screenshots of the UI and compare every pixel's colour in screenshot one with the pixel colour in the corresponding location of screenshot two. If any pixels are different, then you know the images are not identical.

Maybe a single pixel difference is quite a high threshold, so let's say that there's probably a problem if 1% or more of pixels are different.

It's not quite as simple as that. There are other factors that might give false negatives. Aliasing occurs when an image's resolution is too low for the processor to accurately render smooth lines. You end up with jagged edges, often in text. Operating systems will apply **anti-aliasing** techniques to give a smoother appearance, but they all do it slightly differently.

**Problem:** You can get different comparison results per environment.

<div style="text-align: center; margin-bottom: 18px">
<img style="width: 40%" src="/img/blog/visual-regression-testing/aliasing.png" alt="Anti aliasing" />
<div style="font-size: small; font-style: italic">Source: <a href="https://helpx.adobe.com/photoshop/key-concepts/aliasing-anti-aliasing.html">https://helpx.adobe.com/photoshop/key-concepts/aliasing-anti-aliasing.html</a></div>
</div>

Image comparison libraries can deal with anti-aliasing problems by ignoring some of the pixels where anti-aliasing is likely to have occurred. Examples of libraries include [ResembleJS](https://github.com/HuddleEng/Resemble.js) and [Pixelmatch](https://github.com/mapbox/pixelmatch).

# Visual Regression Testing Process

## 1. Take screenshot of the UI in good state

The visual regression library or framework of your choice will provide a function to do a visual comparison. When it's invoked on a particular UI element, a screenshot is taken of it. If one doesn't already exist, the test will automatically pass and output the screenshot to disk.

## 2. Store the screenshot in source control

After running the tests, you version the test suite and associated screenshot in source control, e.g. Git. This will become the baseline image.

## 3. Run the tests again after a code change

After making a code change, the test suite can be run again. Code changes, once committed, can also trigger tests configured in a CI environment. The visual regression library or framework will see that a baseline exists and will take a second screenshot of the UI element. It will be held in memory instead of on disk.

## 4. Compares screenshots

The new screenshot is automatically compared with the baseline image to see if there are any visual differences.

## 5. Pass or fail test based on visual threshold

The test will be sent a pass or fail signal depending on how different the screenshot is. This is based on the threshold configured.

## 6. (Optional) Rebase visual if valid change

If the visual test has failed due to a intended change, you can rebase the visual so that subsequent test runs will pass. This is essentially done by removing the old image and running the test again to generate a new baseline to be committed.

# Benefits

## Extending components

Say you have a Panel component which displays a title and a body. You use it in a few places and release it to production.

You later decide to add an icon to the panel in one mode, so you give it a property to enable the icon. You conditionally put the icon next to the title if it exists.

You're happy with how that looks and you integrate it into your page and release it.

You later find out from a customer that the panel is misaligned. **What happened?**

Well, when you added the icon next to the title, you added a margin. But what you didn't realise is that you applied the margin to the title element, which is not scoped to the new property. There's now a margin in both modes, leaving a large gap.

<div style="text-align: center; margin-bottom: 18px">
<img src="/img/blog/visual-regression-testing/baseline.png" alt="Original" />
<div style="font-size: small; font-style: italic">Original</div>
</div>

<div style="text-align: center; margin-bottom: 18px">
<img src="/img/blog/visual-regression-testing/broken.png" alt="Broken" />
<div style="font-size: small; font-style: italic">Broken</div>
</div>

## Large refactors

You may be doing a large scale architectural refactor. For example, you have a CSS file that you need to split up.

In CSS, the order of imports is important for specificity. So if you get the wrong order, your application could look different. Sometimes this will be quite obvious, but perhaps not. You might subtly break your hover states.

## Theming

You might provide theming capabilities to your application, where you set theme styles for each customer. It's not ideal to test the whole application again for every single theme because functionally it will work the same. Instead, you can just use visual comparisons to ensure styles are correct.

# Limitations and solutions

## Dynamic content

If you decide to take screenshots of dynamic content, you're in for some trouble. Say for instance you take a screenshot of the BBC News website. Taking multiple screenshots even within a 5 minute window could yield different results. It's important to remember to stub dynamic content within the system under test.

## Animations

<img src="/img/blog/visual-regression-testing/spinner.gif" style="width: 150px; border: 0; border-radius: 0" alt="Spinner" />

<span style="font-size: small; text-style: italic; text-align: center; display: block">This is just an image ^</a></span>

Animations are a lovely enhancement to your application, but they can play havoc with visual regression testing. Imagine a spinner component. If you take a screenshot of it twice, there's a fair chance that the visual will be different. Animation speeds can vary by milliseconds, so you might get a screenshot of the spinner at its starting position in one case and the middle position in the second case.

The solution here is to disable animations in your tests.

## Browsers

<img src="/img/blog/visual-regression-testing/browsers.png" style="width: 300px; border: 0" alt="Browsers" />

People use different browsers, so you should be testing that the UI looks right in a handful of them. Visual regression testing isn't really about checking cross browser rendering. It's designed to look at regressions, and you usually find most dev regressions in one particular browser. It's an expensive job to run all your tests against every browser, so it's usually sufficient enough to manually smoke test or use something like [BrowserStack](https://www.browserstack.com/).

## Operating systems

Operating systems will render content differently. If you look at your app in Windows and compare it to MacOS, it will likely look different. This is a problem for visual regression testing where you have teams on different environments. It's important to make sure that screenshots are taken under the same environmental conditions every time, otherwise it defeats to purpose of automation testing.

### Docker

<div style="text-align: center; margin-bottom: 18px; overflow: hidden">
<img style="display: inline-block; height: 300px;  border: 0" src="/img/blog/visual-regression-testing/docker.png" alt="Docker" />
<img style="display: inline-block; margin-left: 18px; height: 300px; border: 0" src="/img/blog/visual-regression-testing/win.png" alt="So much win!" />
</div>

Docker allows you to host a different environment within the current OS. It's like a virtual machine, but it works a little differently. The important part here is that you declaratively provide a recipe for how the environment should be configured, and it's followed every single time a container is started.

# Libraries and frameworks

Let's now take a look at some of the visual regression solutions available.

## PhantomCSS

<img src="/img/blog/visual-regression-testing/phantomcss.png" style="width: 200px; border: 0; border-radius: 0" alt="PhantomCSS" />

[PhantomCSS](https://github.com/HuddleEng/PhantomCSS) was one of the first open source front-end visual regression testing libraries around. It was released by [James Cryer](https://twitter.com/jamescryer) at [Huddle](https://www.huddle.com/) back in 2013 and has become quite a popular tool.

It uses [PhantomJS](http://phantomjs.org/), a headless browser (no GUI), to render web applications. On top of that, [CasperJS](http://casperjs.org/) provides a comprehensive API for interacting with the browser and the DOM. For example, you can click on buttons, navigate to pages, get and set properties on page elements, etc. James also created the image comparison library, [ResembleJS](https://github.com/HuddleEng/Resemble.js), to compare the screenshots.

The main problem is that PhantomJS is no longer being maintained as a project. It's based on an old fork of Webkit, which lacks support for modern browser features, such as [CSS Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout). It's possible to use [SlimerJS](https://slimerjs.org/) instead, which is based on Gecko, the rendering engine behind FireFox. However, the technology for communicating with the browser is not standardized.

## Muppeteer

<img src="/img/blog/visual-regression-testing/muppeteer.png" style="border: 0" alt="Muppeteer" />
<span style="font-size: small; text-style: italic; text-align: center; display: block">Logo by: <a href="https://twitter.com/hsincyeh">Hsin-chieh Yeh</a></span>

Due to the problems with PhantomJS, I decided to create a framework that would use the headless mode of Chrome using the [Chrome Debugging Protocol](https://chromedevtools.github.io/devtools-protocol/). Originally, I was writing a lower level library to work with that protocol, but Google released [Puppeteer](https://github.com/GoogleChrome/puppeteer), which did exactly that. Puppeteer allows you to interact with the browser and pages in same way CasperJS does with PhantomJS. The difference is you get a lot more control, and a standardized protocol is used.

Muppeteer uses Mocha and Puppeteer, hence the name. The image comparison library used is [Pixelmatch](https://github.com/mapbox/pixelmatch), a lighter version of ResembleJS.

One of the benefits of the framework is the abstractions to set up and configure the tests. In Muppeteer, many of the abstractions occur using a wrapper `describeComponent` block. You can configure the URL to navigate to here, or you can do that in a more central place, like in the example below.

The important part is the `assert.visual()` function. This is what performs the visual comparison. All you provide is a DOM selector for the component you want to take a screenshot of and the framework will do the rest.

```javascript
describeComponent({ name: 'Panel' }, () => {
    describe('Simple mode', async () => {
        it('title and body exist', async () => {
            ... omitted functional tests ...
        });
        it('title and body appear correctly', async () => {
            await assert.visual(panelContainer);
        });
    });
    describe('Icon mode', async () => {
        it('title, body and icon exist', async () => {
           ... omitted functional tests ...
        });
        it('title, body and icon appear correctly', async () => {
            await assert.visual(panelContainer);
        });
    });
});
```

To configure your test suites, you just have to invoke a configuration function in a Node script as your entry point for running the tests.

```javascript
const ConfigureLauncher = require('muppeteer');

const launcher = await ConfigureLauncher({
        testPathPattern: 'tests/**/*.test.js`
        reportDir: 'tests/report',
        componentTestUrlFactory:
            ({name}) => `http://${IP}:${PORT}/${name}`,
        visualThreshold: 0.05,
        useDocker: true,
        dockerChromeVersion: '67.0.3396.79',
        onFinish: () => {}
    }
);

await launcher.launch();
```

The first interesting property is `componentTestUrlFactory`. You can give it a function that will return a URL for the tests to navigate to. This is instead of providing the `url` property directly in the test file. The benefit of this is that you can reduce boilerplate in your tests and just concentrate on the important stuff. The function will receive the object you provide to the `describeComponent` function, which means you can customize the URL on a per component basis.

The other interesting configuration is setting up Docker. Normally, you'd have to create a Dockerfile for setting up all the dependencies for the image. With Muppeteer, the only dependency you have is having it installed on the computer. By setting the `dockerChromeVersion` property, when you run the tests, Muppeteer will automatically pull a Docker image with that version of Chrome installed. The test runner will then start a container and run the tests against the Chrome instance running inside.

There's literally nothing for you to do.

![Screen-Shot-2018-06-14-at-14.13.16](/img/blog/visual-regression-testing/output.png)

When running your test suites, you will see a percentage mismatch in the terminal when a test fails. You can then go to the screenshot path to view the differences.

<div style="text-align: center; margin-bottom: 18px">
<img src="/img/blog/visual-regression-testing/baseline.png" alt="Original" />
<div style="font-size: small; font-style: italic">Original</div>
</div>

<div style="text-align: center; margin-bottom: 18px">
<img src="/img/blog/visual-regression-testing/broken.png" alt="Broken" />
<div style="font-size: small; font-style: italic">Broken</div>
</div>

<div style="text-align: center; margin-bottom: 18px">
<img src="/img/blog/visual-regression-testing/diff.png" alt="Diff" />
<div style="font-size: small; font-style: italic">Diff</div>
</div>

You can take a look at [muppeteer-example](https://github.com/gidztech/vrt-examples/tree/master/examples/muppeteer-example) for an example repository including both a unit test and an end-to-end test suite.

## Jest-image-snapshot

While I was creating Muppeteer, American Express were also working on a Jest plugin called `jest-image-snapshot`. This is another great visual regression utility. The main difference is that it's not a full framework, it's a just a library. There are pros and cons of both approaches.

You can easily integrate this into your existing Jest-based test suites, whereas Muppeteer forces you to use Mocha and in a particular way (TDD syntax).

However, you have to do a lot more configuration yourself in every test. It's up to you to set up Puppeteer in your Jest config file, connect to Chrome and tear it down. Having said that, I recently found a new project called [jest-puppeteer](https://github.com/smooth-code/jest-puppeteer) which does help to reduce some of this boilerplate.

You would still need to configure Docker yourself in any case.

```javascript
it("renders correctly", async () => {
  const page = await browser.newPage();
  await page.goto("https://localhost:3000");
  const image = await page.screenshot();

  expect(image).toMatchImageSnapshot();
});
```

You can take a look at [jest-image-snapshot-example](https://github.com/gidztech/vrt-examples/tree/master/examples/jest-image-snapshot-example) for an example repository including both a unit test and an end-to-end test suite.

## More

There are several other libraries and frameworks available that implement visual regression techniques. Some use more legacy technology, such as PhantomJS, but others use more modern techniques.

Check out [awesome-regression-testing](https://github.com/mojoaxel/awesome-regression-testing) for an extensive list.
