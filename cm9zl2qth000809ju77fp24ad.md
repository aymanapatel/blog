---
title: "Tanstack is All You Need!"
seoTitle: "Tanstack is All You Need!"
seoDescription: "Forget Next.js; Tanstack is the ultimate React framework"
datePublished: Sun Apr 27 2025 11:46:07 GMT+0000 (Coordinated Universal Time)
cuid: cm9zl2qth000809ju77fp24ad
slug: tanstack-is-all-you-need
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1745754330606/043871b7-25d6-447c-b3b8-b38e209c913c.png
tags: reactjs, tanstack

---

We know React is a lightweight powerful runtime. There are a plethora of libraries that are present that alleviate React to make it dead-simple in making beautiful apps.

Want to make amazing animations, you have [Framer Motion](https://www.npmjs.com/package/framer-motion). Want to make and edit videos programmatically, [Remotion](https://www.remotion.dev/) has your back. Want a beautiful UI, [Radix UI](https://www.radix-ui.com/) has made it simple with following web standards and great accessible components.

But the issue is the cost of maintenance and bundling of it in 1 ecosystem. Like Angular has [Interceptors](https://angular.dev/guide/http/interceptors) for Fetch requests; you need a non-React library like Axios. This leads to fragmentation where libraries might not work well with each other.

The examples I had shown in 2nd paragraph were for animation which might be optional. In normal complex UI apps, there are a suite of libraries that are typically used. These are:

1. **Data fetching, state management**: These might be different things (Redux vs React Query vs Rxjs) but in essence, it is a way to sync data from server to client and viceversa.
    
2. **Form libraries**: Form validation is one of the most complicated parts. Making forms that are very big, do not have re-rendering issues and provide good Developer experience for complicated validations, a form validation library comes very handy. Some examples include React Hook Form, Formik etc.
    
3. **Tables**: Whilst HTML has tables, in the real world you have complicated tables. You need complex filtering, nesting of rows, server-side sorting/pagination, column resizing, sticky header etc. And all these features need to be accessibility compliant. Libraries such as Aggrid, Material UI's Data Grid are typically used for providing all the aforementioned features.
    
4. **Router**: This is one of the most changing paradigms. In MPA age, it used to be server driven; then in the 2010s, SPAs became the norm, which caused the shift of routing to client. In 2020s, with the advent of Server components (React, Vue etc), the paradigm has swung back again to have server in the mix. React Router's pre v5 and v5 -&gt; v6 -&gt; v7 -&gt; and beyond can provide the glimpse the shift back to server.
    
5. **Meta frameworks**: This ties quite a bit to Router. The shift to server-based paradigms has brought in need for Meta frameworks. In the React ecosystem, it is Next.js while in Vue it is Nuxt.js.(Angular also has Analog but it is not too popular.
    

Now, let's circle back to the heading which is **Tanstack**.

Tanstack has it all the 5 mentioned libraries with amazing features that help both developers and their ability to create feature-rich apps.

Look for Adam's link on his Tanstack Start tutorial on Frontend Masters:

[https://x.com/AdamRackis/status/1869444556586143837](https://x.com/AdamRackis/status/1869444556586143837)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745754208301/67d419bd-910c-4372-830d-eac0dcd307f7.png align="center")

  

So let's come to the features:

1. **Data fetching -** [**Tanstack Query**](https://tanstack.com/query/latest/docs/framework/react/overview)
    

It is amazing at providing amazing primitives of fetching data, caching as well as maintaining it with server.

Can't go into individual details as [Tkdoto's blog](https://tkdodo.eu/blog/practical-react-query) has it all covered

2. Form library - [Tanstack Form](https://tanstack.com/form/latest):
    

Alternative to React Hook form. Tanstack is amazing at bringing type-safety and Tanstack form is no exception.

And 1 amazing thing is it works with Next.js or SSR based apps; which is typically hard to achieve in the traditional form validation libraries.

3. Tables - [Tanstack Tables](https://tanstack.com/table/latest):
    

Provides a headless table with all the complex table features. Still not as feature-rich as Aggrid, but it gets the job done for most apps.

4. Router - [Tanstack Router](https://tanstack.com/router/latest/docs/framework/react/overview):
    

Tanstack router is type-safe, through and through; without you needing to write any types yourself! Just look at this screenshot and say you are not impressed. Type-safety with auto-complete at the Router level is just amazing!

![Link](https://pbs.twimg.com/media/GDDhnLRacAA0_ou?format=jpg&name=medium align="left")

Apart from that it has so many features:

a. Support for SSR based apps.

b. Support for View transitions API for smooth transitions 🤌🏻

c. Navigation blocking

5. Metaframework - [Tanstack Start](https://tanstack.com/start/latest/docs/framework/react/overview):
    

Tanstack Start is the culmination of the libaries that Tanstack has built. It works beautifully with Tanstack Query for great data fetching, Tanstack Router for Type-safe routes.

What Tanstack Start provides is:

1. Server functions
    
2. Full-stack typesafety (If you want to use Server Actions, Middlewares and [tRPC](https://trpc.io/) for RPC-like API function calls)
    
3. Full Document SSR
    

[People](https://x.com/nikitavoloboev/status/1843239102814339363) are legit trying to move away from Next.js!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1745754271801/f5cd0670-4983-4e0f-9f3c-1a9d3a10d650.png align="center")

Tanstack start deploys in any runtime; be it Netlify, Vercel, Cloudflare pages, Bun runtime, Node.js runtime!

So that's it. Tanstack makes React better. And has made developers happy. Thank you Tanner and the rest of team.

PS: Dcy oum fmxo ymck im cdczvft, uchhy cfe bgqvft, Cdmfc