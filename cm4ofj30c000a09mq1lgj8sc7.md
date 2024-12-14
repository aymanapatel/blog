---
title: "Speed up your site using Web worker"
seoTitle: "Speed up your site using Web worker"
seoDescription: "How Web workers speed up computationally heavy scripts and tasks"
datePublished: Sat Dec 14 2024 17:06:52 GMT+0000 (Coordinated Universal Time)
cuid: cm4ofj30c000a09mq1lgj8sc7
slug: speed-up-your-site-using-web-worker
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1734195964041/ebacaa02-217d-45cf-a743-caa3da0280d3.jpeg
tags: javascript, web-development, webdev, event-loop

---

Javascript is traditional a single-threaded application. It uses a concept of Non-Blocking IO using Event Loops. Event loops work with Queues, Queues, Web environment and Heap to orchestrate execution on the browser. But there are certain situations when this architecture becomes a bottleneck. For example, loading huge javascript scripts that have nothing to do with user experience or doing a computationally heavy task client-side.  
We will go into **Web Workers**. A simple API provided by API that helps offload some tasks to a non-render blocking thread.

# Event Loop revision

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734195322424/46f7befd-bfa9-47eb-a8d4-be1c2c9e8710.png align="center")

We will go briefly on how Event Loop executes.

For example for calling a `fetch` API using Promises. There are multiple things that are at play:

1. **Web API:** The code that runs in the browser. In this example it is of this format:
    

```javascript
const promise1 = Promise.resolve(3);
const promise2 = new Promise((resolve, reject) =>
  setTimeout(reject, 100, 'foo')
);

const promises = [promise1, promise2];

Promise.allSettled(promises).then(results => 
  results.forEach(result => console.log(result.status))
);
```

2. Call Stack: It contains all the function calls. In the above example it can be Promise methods or `console.log()`
    
3. Task Queue: This handles all sort of callbacks that are present.
    
    For example,
    
    ```javascript
    button.addEventListener("click",
    ()=> {...} // Task Queue Item
    )
    setTimeout(
    ()=> {...} // Task Queue Item
    , 100)
    setInterval(
    ()=> {...} // Task Queue item
    , 100)
    ```
    
4. Task Queue: These hold high priority task such as Promises, Mutation Observers etc
    
    ```javascript
    Promise.then(
    ()=> {...} // Microtask Queue Item
    }
    
    try {
    ...
    } catch (
    ()=> {...} // Microtask Queue Item
    ) finally (
    ()=> {...} // Microtask Queue Item
    )
    
    new MutationObserver(
    ()=> {...} // Microtask Queue Item
    )
    ```
    
5. Event Loop: This is the orchestration layer. It looks at the queues and puts them in the call stack for execution based on sequence of additions.
    

But there is a missing piece. There are times when there are items that can be render-blocking but should not be. Let us consider a scenario:

> Say you have a website, and your product manager says 1 day that they want to integrate a Product analytics tool such as Mixpanel and Amplitude. As a developer, you say “Sure”, thinking the UI is going to handle. Then the next day a marketing person enters and says they want to integrate with Google and Facebook ads for their marketing campaign. You say, “Well, I can add it but the site is going to be sluggish”. The marketing and product folks’ KPI are at crossroads. If the application does not have these integration then there is no way to gain insights from product and marketing’s Point of view. But of scripts are added then the app becomes sluggish. What do you do with this chicken and egg problem.

Sometimes at scale in terms of features the application starts having a lot of async operations that can cause the queue, and the call stack to be bloated. For example using analytics libraries such as Mixpanel, Google Tag Manager can cause the page to be unresponsive till it loads. Since these scripts are not important for the UI app to be functional; these become a useless bottleneck. Another example could be some file-heavy client side operations. If the file size is too large, then the file upload could cause the whole page to be unresponsive.

To mitigate this, there is an Worker API which can be used to run scripts that are not related to any sort of Web rendering.

This is the missing piece in the above diagram.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734195381675/8452ef62-13c7-4697-bae9-596868b7e522.png align="center")

# Web Workers

Web Workers allows to run scripts in the background thread without without causing any interference to the browser canvas. You can create a worker that does a specific task which can then return back after it is executed. There is no interference with the main thread. Hence, UI can not have any performance impact on loading analytic scripts or doing some client-heavy computation.

Let us look into an example using a library called Partytown which uses Webworker under the hood.

# Example: Enter Partytown and Webworker for script loading

Firstly, we have added scripts for Mixpanel and Amplitude. These are heavy scripts that are render-blocking and are 100s of KBs.

Without Partytown, Lighthouse score was 81. Once of the biggest bottlenecks was loading Amplitude.  
It was making site slower by ~1second.

```javascript
<script src="https://cdn.amplitude.com/script/id.js"></script>
<script>window.amplitude.init('<id>', {"fetchRemoteConfig":true,"autocapture":true});</script>
<script src="https://cdn.amplitude.com/libs/analytics-browser-2.11.1-min.js.gz"></script>
<script src="https://cdn.amplitude.com/libs/plugin-session-replay-browser-1.8.0-min.js.gz"></script>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734195421040/5a813983-069e-4c9b-b4d9-29351e9e333f.png align="center")

After adding Partytown like this:

```javascript
<script type="text/partytown" src="https://cdn.amplitude.com/script/id.js"></script>
<script>window.amplitude.init('<id>', {"fetchRemoteConfig":true,"autocapture":true});</script>
<script type="text/partytown" src="https://cdn.amplitude.com/libs/analytics-browser-2.11.1-min.js.gz"></script>
<script type="text/partytown" src="https://cdn.amplitude.com/libs/plugin-session-replay-browser-1.8.0-min.js.gz"></script>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1734195464359/76cf3049-8ae1-4655-83bd-3f57396fd411.png align="center")

We see the Lighthouse score increase to 88! And the Amplitude script is no longer a bottlneck

Web workers can also be used in heavy computation task file file decompression or background video/async loading. The possibilities are endless, and this blog is just a starting point.

# Closing thoughts

1. [Event Loop Visualized](https://www.youtube.com/watch?v=eiC58R16hb8&pp=ygULZXZlbnQgbG9vcHM%3D)
    
2. To learn more about Partytown, you can check the offical [docs](https://partytown.builder.io/).
    
3. More on [Web Workers](https://www.honeybadger.io/blog/javascript-web-workers-multithreading/)