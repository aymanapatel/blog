---
title: "Microfrontends - Decoupling Frontends"
seoTitle: "Microfrontend: Learning from Microservices and learning to do it right"
seoDescription: "Microfrontend: Learning from Microservices and learning to do it right"
datePublished: Sun Feb 11 2024 15:26:18 GMT+0000 (Coordinated Universal Time)
cuid: clshnr8nn000709l075vdbj4a
slug: microfrontends-decoupling-frontends
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1707665148017/785b3f7e-b0d9-46fc-aa8b-378ee7daf4c4.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1707665168537/d9ffa773-7677-4ae8-b833-03397cd223e3.jpeg
tags: microservices, microfrontend, module-federation

---

# Preface: Microservices

We are all widely aware of Microservices. For the uninitiated, it is a way of decomposing your applications to independently deployable units.  
This allows to structure teams so that they can work independently with different release cycles. But let us define the goals more explicitly:

## **Goal #1: Loose coupling**

In layman terms, these are services that are required to work where changes in any service does not impact the other. One way to find whether to see there is tight-coupling is to see the commits of each service; [{ Reference: microservices'io blog on doing microservice correctly](https://microservices.io/post/architecture/2023/07/06/msa-is-meant-to-simplify-development.md.html#analyze-git-commits-to-detect-tight-design-time-coupling) }. If you find commits for both services simultaneosly; then you are writing a highly-coupled service.

## Antipattern #1: More the merrier aka Too many Services

One aspect of microservices that creates a lot of issues is to have too fine-grained service. In the age of serverless, this is an acute problem. Having 1 line functions that are discarded is a good design for only certain use-cases. For traditional applications; having both loose-coupling and high cohesion is key! Always start out as a monolith; and break into microservices when monolith becomes too big.

## Goal #2 : Observability and Service Discovery

Having a ton of microservices, means a number of failure points. If observability is not upto par then S1 outage will be less about solving customer issues and more like finding a needle in a haystack.

Good observability includes logs, traces and metrics which is the holy trinity triangle. If any one is not present or does not work as intended, then good luck finding RCAs and fixes!

## Anti Pattern #2: Microservice is magic

Read: [**Microservices adoption anti-patterns: microservices are a magic pixie dust**](https://microservices.io/post/antipatterns/2019/01/07/microservices-are-a-magic-pixie-dust.html)

Also read and ponder: [Fred Brooks "No Silver Bullet"](https://en.wikipedia.org/wiki/No_Silver_Bullet)

## Goal #3 : Tests

Write tests that test the whole system. Integration test is key!

# Microfrontends

Micro-frontends is a term to put microservices in the frontend world. Whilst the **Goals** (Loose-coupling, Observability etc) and **Anti-patterns** remain as it is; there are more caveats and considerations for microfrontends to succeed. Saying we need to work independently sounds great on paper, but it will lead to a lot of issues such as [feature envy](https://softwareengineering.stackexchange.com/a/212130), ownership, constant breaking changes for downstream dependencies due to change in micro-service/frontend, lack of tests etc

One think to keep in mind about Microservices and Microfrontends that happens in the real world is that these are not technologies which automagically solves your technology or scaling issues. There are to an extend to a way to solve **Organization issues and enhance collaboration between teams!**

## Matrix of compatibilities and Considerations

### Matrix #1: Polyglot Framework (and Version)

There are a lot of options for framework. Goal would be to build a setup which is framework agnostic. Otherwise it will lead to *framework lock-in*

| **Frameworks** |
| --- |
| React (And Metaframeowrks such as Next.js) |
| Vue (And Meta frameworks such as Nuxt.js) |
| Angular |
| Native JS |
| Svelte |
| Solid |
| ... JS Framework of the month |

### Matrix #2: Bundlers

There has been a lot of ways of building frontends

* In the 2000s; simple `<script>` tag which pointed to a CDN
    
* In the early to mid 2010s; there were ***task runners*** such as **Gulp, Grunt** and early ***module bundler*** such as **Browserify**
    
* In the mid to late 2010s; ***module bundlers*** such as Webpack, Parcel and Rollup became popular
    
* In the 2020s; Evan from Vue created **Vite** which is basically as combination of next-gen/existing JS runtime and technologies which include **Rollup (prod builds), ES-Modules (For instanct code load during development), ESBuild (Transpilation and minification)**
    

| Bundlers |
| --- |
| Webpack |
| Rollup |
| Parcel |
| Vite |
| `<script>` from a CDN |
| (Legacy) Grunt |
| (Legacy) Browserify |
| (Legacy) Bower |
| (Legacy) Gulp |

### Consideration #1: Bundle size and Depedency Management

What is worse than using a resuable component is to decrease the performance of the application. In the micro-service world, every microservice adds latency. Connection establishment, TCP connection, TCP Wait time, Database call; all these add up. In the Microfrontends world; it is simpler to explain the main bottleneck which is **Bundle-size and Dependency management**. But this is non-trivial to solve. Any framework you choose to use (or Build yourself) you need to consider how the UI application will load the duplicate bundle. Usually this is the runtime like React, Vue, Angular etc.

Webpack's Module Federation provides a feature [singleton](https://webpack.js.org/plugins/module-federation-plugin/#singleton) which provides a way to resolve the versions at runtime. But this can still be an issue when Microfrontends evolve to use many inter-dependant libraries.  
Hence, be mindful of this problem during initial architecture phase.

### Consideration #2: Styling Conflicts

One of the most frustrating mistakes that happens in web apps is Style conflicts. Usually it is a good practice to use Design systems, Liniting styles and CSS Variables which can reduce class overriding for styles and CSS collisions; but it may still creep in when using multiple UI projects where both code and devs have a barrier.

Say, team A has created a style for a `<p>` tage like `p { color: purple;}` and team B has created a style for a `<p>` tage like `p { color: red;}`

These need to be solved when doing the architecture of how you are going to build you microfrontends.

Some solutions include:

1. [Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM) to encapsulating the components. Usually used in Web Component Libraries such as Lit
    
2. CSS Modules: These are native browser feature which can work even when there are CSS name collision
    

```css
/* my-style.module.css */
myStyle {
   color: blue;
}
```

```javascript
import * paraStyle from './my-style.module.css'
export default function Para(){
    return (<p className="paraStyle.myStyle">Hello </p>)  
}
```

1. CSS-In-JS: Writing CSS inside JS. This can reduce chance of CSS collisions as the style is tied to component itself. Libraries such as styled-components and emotion are examples of CSS-In-JS.  
    But note the performance gets impacted as CSS-in-JS libraries inserts styles during component rendering which is too late and wasteful for rendering purposes. For more info, read [Sam's article here](https://dev.to/srmagura/why-were-breaking-up-wiht-css-in-js-4g9b)
    

```javascript
import styled from "styled-components";
const ParaGraph = styled.h1`
  color: #e222211;
  font-size: 18px;
`;
const App = () => <ParaGraph>Hello World!</ParaGraph>;
```

1. Adopt [BEM methodology](https://getbem.com/) for clean class names for CSS as well as avoiding style collisions in large projects
    

### Consideration #3: Sharing State/Information between Microfrontends

Even though microservices/microfrontends are decoupled, they need to talk to each other. Traditional methods such as interfaces or application cache/store/DB do not work in this world.

There are couple of approaches

1. State Management Library
    
    Can use either Redux, Ngrx or Zustand. As Redux can be a bigger library which includes a lot of library such as Think, RTK Query (if using newer versions) it might lead to unnecessary bloat. Zustand is a light-weight library which can be included to manage state. State management library can be an anti-pattern; similar to using a single DB for multiple microservices. Also, multiple frameworks have their own way of managing state, hence there are better alternatives than using a state library.
    
2. Browser Native `eventListeners` and `CustomEvent`
    
    Similar to PubSub pattern, you can use a Event Consumer is the App Shell and Event Publisher in the App Triggering the Event. Since this is a Browser-level API; there won't be any hidden suprises with respect to compatibility between frameworks
    
    ```javascript
    // 1. Consumer
    useEffect(() => {
        window.addEventListener("my-state-event", (event) => {
            console.log(event.detail)
        }
    },[])
    // 2. Publisher
    const customEvent = new CustomEvent("my-state-event", {
        detail: { message: "Hello CONSUMER!"},
    });
    customEvent.dispatchEvent(event)
    ```
    
    SHORT AND SIMPLE!
    

### Consideration #4: Cache Bursting

One of the hardest problem in computer science is to cache invalidation. In Backend it is difficult to know when to remove a cache; as cache is not an infinite resource. Similarly, UI also has similar issue but at the browser level. Usually when you use a module bundler, you will have a `contenthash` which will make sure that the hash will change if there is any file change

Webpack provides this as giving a filename with hash

For example:

```javascript
// webpack.config.js
module.exports = {
  output: {
    filename: '[name].[contenthash].js',
    publicPath: '/build/',
  },
  // ...
};
```

There are other mechanisms also such as adding a `Cache-Control` header in your Nginx or Apache configuration

Nginx Example:

```nginx
location /microFE1 {
  # Route url to initial `index.html`
  try_files $uri $uri/ /index.html;
  # 1.`no-store`:Do not store any response
  # 2.`no-cache`:Ask Origin Server before using cache-content
  # 3.`must-revalidate`: If stale; Go to Origin Server. If fresh; Reuse
  add_header Cache-Control "no-store, no-cache, must-revalidate";
}
```

### Consideration #4: Testing

Writing tests is more important than ever in the micro-fronend world. As Murphy's law would suggest "Anything that will go wrong, will go wrong"

So it is important to include all type of testing to get instant feedback loop. It is also important that the consumers of application do not get breaking changes.

Testing types include

| Type | What |
| --- | --- |
| Static Types | This is not testing perse, but writing typescript types is going to establish the API for consumers and also see if consumers are using the methods and params with the correct types |
| Unit test | Writing unit test is beneficial for a complicated business logic. But any sort of UI rendering is not useful as it runs on JSDOM. Better to write e2e tests for UI rendering scenarios |
| Integration test | Integrating between microfrontends is highly probable so it is better to run test suite for consumers in a staging environemnt before publishing the artifact |
| e2e test | Can be run after publishing, just to make sure things are running smoothly and there is not performance |
| Snapshot testing | This is a different type of testing that is useful in this context. Here the initial test for a component saves a snapshot; and after things are integrated, another test can be run to see any visual changes are there. If unexpected visual changes occur then the test case fails. |

Usually there is a testing pyramid that is followed, but from my experience, testing trophy (Kent C Odd's idea) is more suitable wherein you write good amount of good Typescript Static Types and Integration test. e2e and unit test are written but there are lower in number.

### Consideration #5: Deployment

Deployment is the key to make the Microfrontend discoverable and available.  
The deployment strategy should include

1. Fast build times: Small things should run fast, so adopt technologies that improve you CI builds.
    
    * Module bundler (Vite, Rspack) improve by leveraging modern runtime such as Esbuild (Vite) and Rust (Rspack is Webpack's Rust port by the person who owns Webpack's Module Federation)
        
    * Build systems (Nx, Turbopack) improve build times by caching CI builfs
        
2. Storing in a Highly-available storage such as Amazon S3 or any other HA storage artifactory
    
3. Serving the bundles via CDN so that JS is loaded ***blazingly fast!***
    

### Consideration #6: Observability

We need to see how our application performs. One of the major pitfalls in microservice was that observailbility was an after thought. How to fix something when the service/frontend that is causing an S1 outage is not known.

Observability in the frontedn space is tricky and expensive.

Tricky because you need much more logs which can be hard for a SPA as well as data privacy laws might dictate to make masking necessary

Expensive as the volume of data (network requests, screenshots) all add up storage and requires new querying techniques for postmortems

Some libraries that can add you in enhancing your frontend observability:

| What | Considerations | Cost |
| --- | --- | --- |
| [Dynatrace RUM](https://www.dynatrace.com/platform/real-user-monitoring/) | \- Dynatrace requires an Agent. So need a server for the agent to be attached. If serving via CDN; need to inject at CDN level | $$$$$ |
| [Grafana Faro SDK](https://github.com/grafana/faro-web-sdk) | \- Open Source SDK  
\- Need to setup other Grafana stack such as Loki, Tempo as well as Grafana Agent | $$ (Cost of server) |
| [Sentry](https://sentry.io/welcome/) | \- Inject SDK during JS load  
\- Can degrade performance due to JS bundle | $$$$ |
| [Logrocket](https://logrocket.com/) | \- Inject SDK during JS load  
\- Can degrade performance due to JS bundle | $$$$ |

## Miscellaneous Considerations

> Non technical but definitely important for having a succeeful micro-frontend adoption

1. Documentation: Having auto-generated and upto documentation so that consuming teams know exactly what to do and who to ask for help when things do not work (which it won't initially)
    
2. Governance: How the ownership of frontends should occur. How to manage improvements and feature requests amongst other things.
    

# Implementation

## Webpack 5's Module Federation

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1707663907911/882202f8-dd53-41f2-87bf-b597c1315a70.jpeg align="center")

Webpack 5 has introduced concept of [Module federation](https://module-federation.io/docs/en/mf-docs/0.2/getting-started/)

It has the concepts of exposing module and consuming applciation to consume these remote modules.

`remoteEntry.js` is the file where all the code resides in the library producer which can be consumed by the consumer

Module Federation is a Plugin which can be configred in the `webpack.config.js` file which has several configs.

| Config | What | Link |
| --- | --- | --- |
| `exposes` | expose `modules` from 1 app to other | [Component Ownership Doc](https://module-federation.io/docs/en/mf-docs/0.2/component-level-ownership/) |
| `remote` | Consume `module` from another app via the producers'`remoteEntry.js` | [Doc Link](https://module-federation.io/docs/en/mf-docs/0.2/dynamic-remotes/) |
| `singleton` | Pin Shared Library to a version or a range of version | [Doc Link](https://webpack.js.org/plugins/module-federation-plugin/#singleton) |
| `shared` | Shared resources such as runtime (Angular, React) & Libraries (Lodash, Axios etc) | [Doc Link](https://module-federation.io/docs/en/mf-docs/0.2/shared-api/) |

## Web components

> [MDN Link](https://developer.mozilla.org/en-US/docs/Web/API/Web_components)

These are browser APIs which allow to create custom HTML elements. There are 3 building blocks

1. **Custom Elements:** Defining your custom element such as `<my-custom-element>`
    
2. **Shadow DOM:** This is for "encapsulating" the DOM to the custom element. This makes the element private and ensures for CSS style collisions occur.
    
3. **HTML Templates and Slot:**
    
    Templates provide structure to custom element while Slot provide placeholder
    

### Library 1: [Stencil.js](https://stenciljs.com/docs/introduction)

It is a framework to write React-like code as Input and as an Output provides a Framework agnsotic Web Component

Some downsides with Stencil is that the library size might get bloated with high number of components. Also, it is not natively compatible wit ES Modules so Vite cannot work with Stencil ([Source](https://stenciljs.com/docs/config-extras#enableimportinjection))

### Library 2 :[Lit](https://lit.dev/)

A Web component library built by Google. It is lightweight (5KB only)

## [Single-spa](https://single-spa.js.org/docs/building-applications)

A framework that allows all possible permutation of mainstream frameworks (`<script>` , Angular, Vue, React), Build systems (Gult, Grunt, Webpack, Bower), SystemJS loader (Optional but recommended).

### Configuring

1. Root HTML which is shared by all single-spa applciatopns
    
2. Register applications and implementing `bootstrap`, `mount` and `unmount`
    
3. Calling SingleSPA's `singleSpa.start()` to initialise application.
    

## Other Implementation

1. Bit.dev
    
2. Luigi
    

# Conclusion

This blog was written with the intention of what it takes to enable a successful Microfrontend architecture.  
Little less on the demo-side due to a lot of PoCs covering all frameworks would be a lot of time-consuming