+++
title = "Docker-in-JS"
description = "Recently I wrote a library called `jest-puppeteer-docker`, which provides a Docker setup for running your Jest Puppeteer tests. Here are the interesting decisions I made regarding the internals of it."
tags = [
    "javascript",
    "docker",
    "networking",
    "jest",
    "puppeteer",
    "visual regression testing",
    "bash"
]
image = "docker-in-js/heading.png"
date = "2018-12-09"
highlight = "true"
+++

<img src="/img/blog/docker-in-js/heading.png" style="width: 400px; margin-top: 20px; border:0" />

Recently I wrote a library called [jest-puppeteer-docker](https://github.com/gidztech/jest-puppeteer-docker), which provides a Docker setup for running your Jest Puppeteer tests. Here are the interesting decisions I made regarding the internals of it.

# 🏃‍♂️ Motivation

[jest-puppeteer](https://github.com/smooth-code/jest-puppeteer) is a library for Jest that allows you to run browser-based UI tests using the Puppeteer API. It launches Chromium and handles the communication between the two.  

I use [Visual Regression Testing](https://gideonpyzer.com/blog/visual-regression-testing/) to capture CSS regressions. I wrote an [article](https://gideonpyzer.com/blog/visual-regression-testing/) about it if you are not familiar with it. The main problem with using "jest-puppeteer" directly for this particular case is environmental differences in the rendering of the pages. Docker is a solution to that problem. 

## 😐 Easy (slow) solution
One way to solve this is to launch a Docker container, `npm install`, copy the app over, and run `npm run test`. We can create a mount point to capture artefacts (e.g. test reports, failed screenshots), so that CI can report on these stats. 

This is a perfectly valid solution, but this can be quite slow. Your CI is going to set up a clean environment with Docker installed, and then build a Docker image inside of that and start it up. This is going use a lot of resources and slow your app and tests down. 

## 🚀 Better solution (maybe)
Another solution would be to run your app and tests directly in the CI environment, but run the browser itself in a container, and then communicate between the two. We can do that with [Remote debugging](https://chromedevtools.github.io/devtools-protocol/), connecting via a web socket. In order to achieve this, I created [jest-puppeteer-docker](https://github.com/gidztech/jest-puppeteer-docker).

# 🔎 How jest-puppeteer-docker works

<div style="display: inline-block">
<img src="/img/blog/docker-in-js/jest.png" style="height: 250px; border:0" />
<img src="/img/blog/docker-in-js/puppeteer.png" style="height: 250px; border:0" />
<img src="/img/blog/docker-in-js/docker.png" style="height: 250px; border:0" />
</div>

The main goal of the library is for it to automagically set up a Docker container and run your tests within the Chromium instance inside it. The end-user shouldn't need to do anything themselves regarding the container configuration. 

Normally, you run your `docker-compose` command with some static config, but in this case, the config needs to be determined dynamically, using JavaScript!


In order to use "jest-puppeteer", you need to have the peer dependency "puppeteer" installed. Puppeteer ships with a Chomium binary that is guaranteed to work with their API. The version of Chromium is referenced in the `package.json`. We need to use that version in our Docker image.

```javascript
"puppeteer": {
    "chromium_revision": "609904"
},
```

## Building Docker image
We could create a Dockerfile and `apt-get` all the dependencies and pull the Chromium binary. However, building an image from scratch takes a while, so instead, I found [chrome-headless-trunk](https://hub.docker.com/r/alpeware/chrome-headless-trunk/) on Docker Hub. This provides pre-built versions of Chromium, tagged by revision. We can simply pull an image with a particular tag and we're ready.

Now, to work out which revision to retrieve, We need to parse the `package.json` file. 

```javascript
const revision = require(path.resolve(puppeteerConfigPath)).puppeteer.chromium_revision;
```

Finally we need to patch the internal Dockerfile to reference the tag associated with the revision.

```javacsript
const data = readFileSync(dockerFilePath, { encoding: 'utf-8' });
const previousTag = data.match(/:(.*)/)[1]; // get everything after : on same line
const newData = data.replace(previousTag, latestTag);
writeFileSync(dockerFilePath, newData, { encoding: 'utf-8' });
```

### Result
```docker
FROM alpeware/chrome-headless-trunk:rev-609904
```


## Pulling Docker image
The next thing we do is to build and run the container from JavaScript using [`exec`](https://nodejs.org/api/child_process.html#child_process_child_process_exec_command_options_callback).

`await exec('docker-compose -f docker-compose.yml build --pull chromium');` \
`await exec('docker-compose -f docker-compose.yml build up -d');`

Once we're up, we need to connect to the Chromium instance and obtain a web socket.

```javascript
const res = await request({
    uri: `http://localhost:9222/json/version`,
    json: true,
    resolveWithFullResponse: true
});

const webSocketUri = res.body.webSocketDebuggerUrl;
```

*Note: This is not the exact code I'm using.*

Finally, we just need to pass the web socket we obtained to "jest-puppeteer", which will then hand over all the remaining work to it.

## 🌍 Accessing host from Docker container
This was a nightmare. If you use the default bridge networking on Docker, you can access a server running on your host by IP. But when you have multiple network interfaces, things get complicated. 

Docker for Mac and Windows exposes the host IP with a friendly hostname `host.docker.internal`, but this is [not supported in Linux currently](https://github.com/docker/for-linux/issues/264). 

I spent a long time hacking around, and managed to create an entrypoint bash script that provides a workaround. At this point, there's a chance some of this is unnecessary, but as soon as it started working, I decided not to touch it again. It's probably terrible.

```bash
# Make sure there's a host entry
HOST_DOMAIN="host.docker.internal"
DOCKER_IP="$(getent hosts host.docker.internal | awk '{ print $1 }')"
echo $DOCKER_IP " " $HOST_DOMAIN >> /etc/hosts

ping -q -c1 $HOST_DOMAIN > /dev/null 2>&1
if [ $? -ne 0 ]; then
  # Try using default interface
  DOCKER_IP="$(ip -4 route show default | cut -d' ' -f3)"
  ping -q -c1 $DOCKER_IP > /dev/null 2>&1
  if [ $? -eq 0 ]; then
      # Default interface was good so patch hosts
      echo $DOCKER_IP " " $HOST_DOMAIN >> /etc/hosts
  else 
      # Try eth0 instead and then patch hosts
      DOCKER_IP="$(ip addr show eth0 | grep 'inet ' | awk '{ print $2}' | cut -d'/' -f1)"
      echo $DOCKER_IP " " $HOST_DOMAIN >> /etc/hosts
  fi  
fi
```

Now if you run a local server on your host, you can access it via http://host.docker.internal:3000.

## 🛠 Launching Chromium with custom flags
The pre-build Docker image contains a startup script for launching Chromium with some default flags. The consumer of this library may wish to provide additional flags via a config file. 

We need to find a way to get those flags from JavaScript running on the host, to a bash script running inside the Docker container. This is fun!

Our config may look like this:

```javascript
config.chromeArgs [ '–ignore-certificate-errors' ];
```

The first thing we can do is to read the config from the JS config file, and then create an environment variable containing that config.

```javascript
const { chromiumArgs } = require(path.resolve(process.env.JEST_PUPPETEER_CONFIG));

if (chromiumArgs) {
    process.env.CHROMIUM_ADDITIONAL_ARGS = chromiumArgs;
}
```
A problem I ran into later was the fact I needed to read the config file at two points in time. The first time is to read the Chromium arguments, which needs to be done before launching the container. 

The second time, "jest-puppeteer" will require it in order to read the web socket from file. In Node, when you `require` something, it gets added to a cache. The next time you `require` the same file, it will fetch it from the cache instead. 

This is a problem because the web socket is not available the first time we `require` it and we need to the web socket to be read during the second time. The solution here is to delete the cache.

```javascript
delete require.cache[path.resolve(process.env.JEST_PUPPETEER_CONFIG)];
```

In order to pass the `process.env.CHROMIUM_ADDITIONAL_ARGS` environment variable to the container, we need to use `--build-arg` in our `docker-compose build` command.

At this point, the environment variable is available during the build stage, but it won't be accessible inside the container. To solve this, we need to add the following to our `Dockerfile`.

```docker
ARG CHROMIUM_ADDITIONAL_ARGS
ENV CHROMIUM_ADDITIONAL_ARGS=${CHROMIUM_ADDITIONAL_ARGS}
```

We can finally append these args in our bash script running inside the container.

```bash
CHROMIUM_ADDITIONAL_ARGS=$(echo $CHROMIUM_ADDITIONAL_ARGS | tr ',' ' ')
sh -c "/usr/bin/google-chrome-unstable $CHROME_ARGS $CHROMIUM_ADDITIONAL_ARGS"
```


## 🎉 End result
We're finally there. After a lot of steps and potentially dodgy hacks, we've provided a library that will automagically configure Docker images and containers within a Jest Puppeteer setup.

<img src="/img/blog/docker-in-js/example.png" alt="Example" />