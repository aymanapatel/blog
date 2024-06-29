---
title: "Ditch Javascript Frameworks and Embrace HTMX"
seoTitle: "Ditch Javascript,  Embrace HTMX"
seoDescription: "Let their be light at the end of the frontend tunnel"
datePublished: Sat Jun 29 2024 20:31:42 GMT+0000 (Coordinated Universal Time)
cuid: cly0kve7x000009mq8flgaulw
slug: ditch-javascript-frameworks-and-embrace-htmx
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719693031151/596a8815-161f-4e3a-80bb-cbcd36f034c1.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1719693059133/03824ade-352e-4d23-b4b2-0e800c1788a3.png
tags: frontend-development

---

# Pendulum swings in UI

First in the 2000s they said, serve Web pages from server. The common tech stack would be PHP/.NET/Java based servers returning HTML. Then in 2010s they said, serve JS and hydrate HTML on client. Common tech stack would be a decoupled frontend framework such as React, Angular, Vue. Now in 2020s they are saying to use a "full-stack" Javascript-based frameworks, have some bits written for the client and some on server. There is another kid in the block that most backend engineers will like to use.

## The world of SSR, ISR, SSG, CSR, SC

| What | Fullform | What is does | Frameworks |
| --- | --- | --- | --- |
| CSR | Client-side rendering | Server serves full JS, CSS and serves that to client. | React, Vue, Angular |
| SSR | Server-side rendering | Server renders initial data in HTML. | React with `renderToString` & `renderToStream`.  
Meta frameworks such as Next.js (React), Nuxt.js (Vue), React Router v7/Remix, SolidStart, Sveltekit |
| SSG | Static Site Generation | All static content are generated at build time. No room for dynamic content | Documentation site built using Gatsby, Astro |
| ISR | Incremental Static Regeneration | Mix of SSG with cache update using `stale-while-revalidate` | Next.js Page Router with `cache` |
| SC or RSC | Server Components or Re |  | Next.js App router, Nuxt.js Islands |

### SSG aka Static Site Generation

This pattern is to build your entire applications at build time with rendered HTML and styling. And then, you send the HTML and styles to client.

This pattern is seen in blog sites such as [Hugo](https://gohugo.io/), [Jekyll](https://jekyllrb.com/), [Gatsby](https://www.gatsbyjs.com/) and other JAMStack applications.

Major disadvantage is it cannot be used for interactive sites. Best use-case for these are limited to documentation or marketing sites whose data do not change frequently.

### ISR aka Incremental Static Regeneration

It is a enhancement over SSG. It allows changing site after it has been built. It uses the `stale-while-revalidate` cache header to make a decision to either take data thst is cache or refresh the cache and fetch form network.

Angular and Meta framework such as [Next.js](https://nextjs.org/docs/pages/building-your-application/data-fetching/incremental-static-regeneration) for React, [Nuxt.js](https://dev.to/jacobandrewsky/incremental-static-regeneration-in-nuxt-3255) for Vue provide methods to use ISR.

### CSR aka Client Side Rednering

Here, you serve the minimal Root HTML and download the Javascript and CSS. The hydration step where the framework builds the App using the root HTML

1. React attaches to the `id` defined in `index.html` during initial render
    

```javascript
// index.ts
// 👇 This is React 17 Client code
import React from 'react';

ReactDOM.render(
    <App />,  document.getElementById('root')
);
```

Corresponding HTML sent to client:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>React App</title>
</head>
<body>
  <!-- This is same id in index.ts -->
  <div id="root"></div>
</body>
</html>
```

After this, the application is rendered. Additional API calls happen after this rendering/hydration is completed.

2. Angular initializes with Apache Ivy out and Zone.js for monkey for change detection.
    

After all this, you need to think about fetching data and storing that data. State management libraries (Redux, Zustand, Ngrx) and data fetching libraries (Axios, Tanstack Query, RTK Query). Since client is decoupled from server, their is always a data impedence mismatch which needs to be thought through so that there is no stale data/state.

Server Components(mentioned below), solve the data impedence issue by using Server as driver of data and clientbasdrover of UI interactions.

### SSR aka Server Side Rendering

This is the most common paradigm which is king of what the 2000s way of doing things was. Here the Server drives the initial rendered HTML along with CSS which is the shown to the client. So the user can see the page loaded. The Javascript which includes the Framework rendering (React VDOM, Angular Ivy & zone.js, Vue VDOM) is then loaded on the page. The main advantage of SSR is that the initial rendering is almost-instant (if using a fast connection and CDN). But you still need JS for the client to be interactive.

TODO:

### Server Components

> This is still a very new concept which not all frameworks support. Even if they do, they vary slightly in implementation

The server part can do server-like things like calling database, logging, access file system, write Middleware etc. And client can be used for creating UX-rich applications. Server components can do API fetching at server-side which access to database.

Server components is typically used with a meta-framework like Next.js for React and Nuxt.js for Vue.

In [React](https://react.dev/reference/rsc/server-components), Next.js is typically used for RSC. [App router](https://nextjs.org/docs/app/building-your-application/rendering/server-components) is used for creating full stack apps.

In Vue, [Nuxt.js](https://nuxt.com/docs/guide/directory-structure/components#server-components) is the de-facto method to use Server components. Firstly, you need to enable the `componentIslands` flag.

```typescript
export default defineNuxtConfig({
  experimental: {
    componentIslands: true
  }
})
```

Secondly, you need to add `<file_name>.server.vue` and `<file_name<.client.vue` for Server and Client component s respectively.

# Enter HTMX: Thinking UI State differtly

Looking the the frontend landscape, it becomes daunting on different ways to render a UI application. It seems like we are back where we started. Creating document with data on Server and sprinkling interactivity on client-side.

There are tools such as Alpine.js, Marko and HTMX that treat H

## Richardson Maturity model and HATEOAS

| Level | Type | Behaviour | Troubles |
| --- | --- | --- | --- |
| Level 0 | RPCs | Have only RPCs with XML, SOAP etc | No structure or intention of the API |
| Level 1 | Resources | Have URL resources which provides identification to the action to be done | No differentiation of the intention (Add, Delete, Update) |
| Level 2 | HTTP Verbs | Addition to resource, pass your intention with GET (safe) and PUT, DELETE, POST (non-safe operations) | How to know the next step |
| Level 3 | HATEOAS | Hypermedia as The Engine Of Application State | No one uses this for some reason |

### HATEOAS

We have gone from Level0 to Level 2; which is fairly simple. You can read about Richardson Maturity Model in Fowler's blog [here](https://martinfowler.com/articles/richardsonMaturityModel.html).

We'll cover about HATEOAS again in this article (if you've read the above blog, it will be repetition). Basically, HATEOAS provides a good way to provide a flow to your APIs. For example, you have a payment flow. You know that you are going to sell them your product in a standard flow: Get the user information -&gt; Authenticate them -&gt; Provide Pricing Page with different tiers -&gt; Payment -&gt; Confirmation.

You don't need create API that links all these flows one after the other. From the UI perspective, you don't need to manage state; you can get all the data from the server. There is less chance of stale data on UI as well as better discoverability of the APIs with no need to send your Postman collection.

HTMX allows for this decoupling. Every backend framework has support of HATEOAS (unlike Server components which is meta-framework specific). And every backend framework has a templating engine.

Hence, embracing HTMX leads to following the KISS principle - Keep It Simple Stupid

## Keep It Simple Stupid - Enter HTMX

> Their is one level of software maturity where tools and libraries evolve to improve performance, enhance usecases. Then there are pendulum swings where the spaghetti code resides.

## HTMX philosophy

Philosophy of HTMX is simple. Firstly, you use your favourite templating library in your sane backend language ([Thymeleaf](https://www.thymeleaf.org/) or [FreeMarker](https://freemarker.apache.org/index.html) in Java, [Templ](https://github.com/a-h/templ) or `html/template` in Golang, [Jinja](https://jinja.palletsprojects.com/en/3.1.x/) in Python, Embedded Ruby (erb) for Ruby, [Blade](https://laravel.com/docs/11.x/blade) templates in Laravel/PHP). Then you add the HTMX methods that make it feel like a SPA. Third, enjoy saving time not using Node with random deprecations, working with CJS and ESMs, deleting `node_modules` and `package.lock.json` or working with Webpack config.

## HTMX attributes

This is the meat of all the rage. HTMX are extensions to HTML that allow with client-side interactions.

### 1\. Form attributes

For example, native HTML `<form>` has only to method attributes: `GET` and `POST`. This makes it harder to adhere Level 2 of RMM .i.e. use **HTTP Verbs**.

HTMX has all the HTTP attributes covered!

| Attribute | Description |
| --- | --- |
| [hx-get](https://htmx.org/attributes/hx-get/) | Issue a `GET` request to the given URL |
| [hx-post](https://htmx.org/attributes/hx-post/) | Issue a `POST` request to the given URL |
| [hx-put](https://htmx.org/attributes/hx-put/) | Issue a `PUT` request to the given URL |
| [hx-patch](https://htmx.org/attributes/hx-patch/) | Issue a `PATCH` request to the given URL |
| [hx-delete](https://htmx.org/attributes/hx-delete/) | Issue a `DELETE` request to the given URL |

### Trigger attributes

These are trigger events that can trigger the form attributes.

These events include:

1. Polling: `<div hx-trigger="every 1s" hx-get="/getUser">Nothing Yet!</div>`
    
2. Keyboard Events: `<div hx-trigger="every 1s" hx-get="/getUser">Nothing Yet!</div>`
    
3. Click Events: `<div hx-trigger="click" hx-get="/getUser">Nothing Yet!</div>`
    
4. Changed `hx-trigger="input changed delay:500ms"`
    

### Target

You can place the result of the API call to another HTML DOM Element by targeting to its `id` attribute

```html
<input type="text" name="q"
    hx-get="/trigger_delay"
    hx-trigger="keyup delay:500ms changed"
    hx-target="#search-results"
    placeholder="Search..."
>
<div id="search-results"></div>
```

### Swapping

You can swap the content of the DOM around the calling DOM tag.

| Name | Description |
| --- | --- |
| `innerHTML` | the default, puts the content inside the target element |
| `outerHTML` | replaces the entire target element with the returned content |
| `afterbegin` | prepends the content before the first child inside the target |
| `beforebegin` | prepends the content before the target in the target’s parent element |
| `beforeend` | appends the content after the last child inside the target |
| `afterend` | appends the content after the target in the target’s parent element |
| `delete` | deletes the target element regardless of the response |
| `none` | does not append content from response ([Out of Band Swaps](https://htmx.org/docs/#oob_swaps) and [Response Headers](https://htmx.org/docs/#response-headers) will still be processed) |

### File uploading

You can do file upload by sending an `multipart/form-data` encoding

```html
 <form hx-encoding='multipart/form-data' hx-post='/upload'
          _='on htmx:xhr:progress(loaded, total) set #progress.value to (loaded/total)*100'>
        <input type='file' name='file'>
        <button>
            Upload
        </button>
        <progress id='progress' value='0' max='100'></progress>
    </form>
```

You can read more about:

1. [HTMX Examples](https://htmx.org/examples/) to get idea on the capabilities of UI components
    
2. [HTMX Essays](https://htmx.org/essays/) to get idea the why of HTMX
    

# Closing thoughts

HTMX is ridiculous. It is the frontend paradigm with simplicity and sanity.