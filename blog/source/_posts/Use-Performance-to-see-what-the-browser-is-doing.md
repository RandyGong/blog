---
title: Use Performance to see what the browser is doing
date: 2021-12-09 21:32:59
tags:
- performance
---


## Preface

The Performance panel of the Chrome browser provides us with the ability to detect page performance, but it provides far more than some performance data. This article will explore other information revealed to us by the performance panel from the perspective of the working principle, combined with the recording results of the actual project.

## Performance panel

Because the final standard architecture has not yet been decided, the implementation details of the major browsers are different. Here we take the architecture of Chrome as an example to compare the relationship between its architecture and the performance panel.

![](https://app.yinxiang.com/shard/s57/res/d26e9040-f8ba-46f8-bfe8-8097ab05704b/92eef10c915246868aad7d1f0c74f474%7Etplv-k3u1fbpfcp-watermark.image.png)

![](https://app.yinxiang.com/shard/s57/res/8baece67-d724-42fe-844f-64f27012ac47/d157e2a1c3454e7e96375172fcf51fe4%7Etplv-k3u1fbpfcp-watermark.image.png)

From the figure below we can see the main threads presented by the performance panel. The performance panel does not include all the threads in the architecture, but mainly related to the page rendering process.

![](https://app.yinxiang.com/shard/s57/res/ee80c3ff-412e-4940-8318-cc908984f795/9c06dde30d994fe1a9931504d3c284be%7Etplv-k3u1fbpfcp-watermark.image.png)

## Network
Network represents the network thread in the browser process. We can see that the timeline contains all network requests and file download call information, and different types of resources are identified in different colors.

## Main
Main represents the main thread in the rendering process. It basically does everything related to rendering, such as script execution, style calculation, layout calculation, drawing, and so on.

## Compositor & Raster
Compositor represents the compositing thread in the rendering process, and Raster represents the raster thread in the rendering process. Nowadays, when a browser draws a page, it can be divided into the following steps:

 - The main thread divides the page into several layers (Update Layer Tree will be mentioned later)
 - The raster thread rasterizes each layer separately
 - The compositing thread merges the rasterized tiles into one page

![](https://app.yinxiang.com/shard/s57/res/88459995-870a-4cf8-be46-c76a4f9a53af/fe55b08bf41a4457a897b7c3f3453fe2%7Etplv-k3u1fbpfcp-watermark.image.png)

We can see that in the performance panel, the main thread calls the grid thread to do the actual rendering at the end.

![](https://app.yinxiang.com/shard/s57/res/50a7eec8-d45c-4380-9615-098fcb30f19f/e5a6bff99f3041ca963bd9afbd93a183%7Etplv-k3u1fbpfcp-watermark.image.png)

## GPU
Obviously, this part is the GPU thread in the GPU Process.

##Browser work report
Next, we will look at the "work report" recorded by the browser roughly from the time dimension.

## Document download & analysis
The starting point of our journey will start by clicking the Reload button (like a refresh) of the Chrome Performance Panel. The current page is first uninstalled. With several log reports, the browser starts downloading index.html.

![](https://app.yinxiang.com/shard/s57/res/4166cb70-5637-490d-850d-f0f99c0329c3/fb82837d4d3b4625b832b6c00d25e852%7Etplv-k3u1fbpfcp-watermark.image.png)

After the HTML document is downloaded, the browser starts to parse index.html according to the HTML standard, and parse the received text string into DOM in the main thread. We can notice that the HTML parsing process is not done in one go, because HTML usually also includes other external resources, such as pictures, CSS, JS, and so on. These files need to be obtained through network requests or caches. Among them, when the HTML parser parses the script tag, the parsing process of the HTML document will be suspended, and the script will be loaded, parsed, and executed instead. Therefore, it can be seen from the timeline of the main thread that the process of Parse HTML is intermittent.

![](https://app.yinxiang.com/shard/s57/res/f4453d72-871e-4424-910d-48c2838fc5cc/ec714326194848ac8244377f649b883a%7Etplv-k3u1fbpfcp-watermark.image.png)

## Disposal of different resources
What is the browser's processing strategy for different resources?

 - The browser downloads HTML and parses it. If it encounters resources such as external CSS, it will be downloaded by the network thread in the Browser process
 - When the CSS is downloaded, the HTML parsing process can continue
 - When parsing encounters an external Script tag (not including async, defer attributes), parsing stops until the script is downloaded and executed
 
In general, the browser's HTML parsing process will not be blocked by the downloading of resources such as CSS and IMG, but the loading and execution of scripts will terminate the HTML parsing. This is mainly because JS may change the structure of the DOM, or JS dynamically loads other JS and then changes the DOM and other potential problems.
Obviously, although the browser can concurrently download resources from several network threads, if it is only processed like the above strategy, when parsing to script, if the file is large or the delay is high, it may happen that "the script monopolizes the thread without other "Resources are downloading" the idle network (idle network). Therefore, the pre-loader (or preload scanner, etc.) will scan the remaining tags in addition to the main thread, and make full use of the network thread to download other resources. This mechanism can optimize the loading time by 19%.

## Script analysis and execution

For complex mid- and back-end applications that emphasize business logic, the performance overhead brought by scripts tends to dominate. We can see from the example in the figure below that removing the unloading before beforeunload, the script itself accounts for more than half of the time overhead. Parsing HTML is second. As for other style calculations, microtasks, garbage collection, etc., it is not the most painful place. Of course, the example project itself focuses on business logic, and the amount of JavaScript code determines its high cost.

![](https://app.yinxiang.com/shard/s57/res/1f539d76-0cd1-4595-9533-df70cdc8a835/ff8dd78f24d44e6e94fab67437395899%7Etplv-k3u1fbpfcp-watermark.image.png)

Sometimes we can consider using async or defer attributes to improve page performance, the difference between the two will not be repeated. What needs to be specifically explained is the case of dynamically adding scripts. As shown in the sample code below, the script will start to download after being appended to the document, and it has the same behavior as async by default, that is, "execute first when it is loaded first".

```Javascript
let script = document.createElement('script');
script.src = "/xxx/a.js";
document.body.append(script);
```

If the async attribute is specifically set, it will follow the behavior of defer, that is, "load first, execute first". 

```Javascript
function loadScript(src) {
  let script = document.createElement('script');
  script.src = src;
  script.async = false;
  document.body.append(script);
}
```

As you can see from the figure below, the appendChild method executed in the call stack dynamically adds the script script, and the download action starts soon after. After the dynamically loaded script is downloaded, the script execution starts immediately.

![](https://app.yinxiang.com/shard/s57/res/b0ff2344-e810-4afc-95e3-dae858f96b1b/bb8b75e15f3d4260beae93cfe5077b8f%7Etplv-k3u1fbpfcp-watermark.image.png)

## Lifecycle & paint timing
The following figure shows the page life cycle flowchart mentioned in the article. In this section, we combine Performance and compare it to the graph for observation.

![](https://app.yinxiang.com/shard/s57/res/18c07b4a-c0a8-4326-b5ec-d4852e99411c/aa05fa7700004fbea54ebf64cad0bae0%7Etplv-k3u1fbpfcp-watermark.image.png)

## Beforeunload
Because the recording of Performance is reloaded on an existing page, the life cycle of the recording starts from the unloading of the page. As shown in Main below, the beforeunload event is first triggered by the browser. It can be noticed that the yellow bar Event: beforeunload is an activity triggered by the browser itself, and we call it Root activities.

## pagehide
From the figure below, we can notice why the trigger sequence of events is inconsistent with the above life cycle flowchart, which is pagehide -> visibilitychange -> unload? In fact, in the previous design of the browser, if the page is visible during the uninstall phase, visibilitychange will be triggered after pagehide, as shown in the screenshot below. This makes the unloading of the page have inconsistent life cycles and sequence of events in different visible situations, which brings complexity to developers.

![](https://app.yinxiang.com/shard/s57/res/4e45de71-e616-4961-b4a7-036c0e5ff0fa/0d7728fd75ba4eb18d690e16c3407196%7Etplv-k3u1fbpfcp-watermark.image.png)

## first paint

First distinguish the following two points in time:

 - first paint: refers to the time when the first pixel starts to be drawn on the screen, such as the background color of a page
 - first contentful paint: refers to the time to start drawing content, such as text or pictures
 
![](https://app.yinxiang.com/shard/s57/res/cf227904-401b-4f47-8433-35d3df626428/1e524e147e6d4339b9e6367da2e7ac34%7Etplv-k3u1fbpfcp-watermark.image.png)

From Performance, we can see a series of actions drawn for the first time:

 - CSS loaded
 - Parse Stylesheet: Parse the stylesheet and build CSSOM
 - Recalculate Style: recalculate the style and determine the style rules
 - Layout: Layout according to the calculation results to determine the size and position of the elements
 - Update Layer Tree: Update the render layer tree
 - Paint: draw the page according to the Layer Tree (position, size, color, border, shadow, etc.)
 - Composite Layers: combined layers, the browser merges the layers and outputs them to the screen

![](https://app.yinxiang.com/shard/s57/res/cfad9ef6-0926-48ec-9917-fb62259b263c/6b51acc8341f4b47a1fa23f5979d4de9%7Etplv-k3u1fbpfcp-watermark.image.png)

## DOMContentLoaded
DOMContentLoaded means that HTML has been fully loaded and parsed. Of course, resources such as style sheets and images may not have been loaded yet. As you can see from the figure below, after multiple HTML parsing, there is no other Parse HTML after DCL.

![](https://app.yinxiang.com/shard/s57/res/876d3c44-0803-4837-9a22-3a68ca5366ab/9c874104bb42401287201da9b4888ea9%7Etplv-k3u1fbpfcp-watermark.image.png)

## pageshow/load
When the browser renders the document in the window due to navigation, the browser will trigger the pageshow event on the window. Not only that, when the page is loaded for the first time, the pageshow event will be triggered after the load event.
So back to the Performance timeline, we can see from the figure below that after the red dotted line (marking load), the browser triggered the pageshow event, which is the root activity mentioned above.

![](https://app.yinxiang.com/shard/s57/res/8b1c9ebc-7756-4ee8-8992-c4a235667bac/7c6b7b80b11c4487b3ec225499d2a7f3%7Etplv-k3u1fbpfcp-watermark.image.png)

## Tasks and performance issues
Unfortunately, Performance has not been able to clearly see the Event Loop. The gray Task in the figure below does not refer to a macro task, it represents "the current main thread is busy and cannot respond to user interaction"; Run Microtasks is indeed a micro task executed at the end of a task. When we click on the call stack to observe, we can see the callback function in the source code and the corresponding source code location.

![](https://app.yinxiang.com/shard/s57/res/85b650a5-8a6c-4375-a5d6-425a2d451165/7b0872f1fe0d4ea3bf76645ed709df8f%7Etplv-k3u1fbpfcp-watermark.image.png)

![](https://app.yinxiang.com/shard/s57/res/04b63400-6bbe-4382-8a9d-c668ee27ad99/d3a709049c9f42c88a9a31710a11ea33%7Etplv-k3u1fbpfcp-watermark.image.png)