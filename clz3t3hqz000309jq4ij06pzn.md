---
title: "Node 22 Features: Learning from Peers"
seoTitle: "Node js 22 features"
datePublished: Sat Jul 27 2024 07:24:58 GMT+0000 (Coordinated Universal Time)
cuid: clz3t3hqz000309jq4ij06pzn
slug: node-22-features-learning-from-peers
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1722065061075/284485ea-1d50-4497-8594-be53ede7ff75.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1722065071408/d403c5b4-f303-4b73-8a9c-7f6bdb3d7dbc.jpeg
tags: nodejs

---

Node 22 has been listening to its competition (Bun and Deno), and have added features to level the playing field. It is exciting to see Node go in this direction, given that it is one of the most popular runtimes on the web.

# Bun and Deno

So, for the uninitiated; what is Bun and Deno.

Both of these are alternatives to the Node/NPM ecosystem. In spite of Node/NPM being popular, its historic context makes it a little bit of a brittle system.  
Like any popular system, Node had to undergo a lot of social challenges such as changed of hands with Joyent and back to the community, the `io.js` fork drama (just to name a few). Not to mention the fast pace of changes on the Web. Typescript becoming the de-facto tool for enterprises (due to code maintainability in large software projects), people ditching CommonJS (initially part of Node.js) in favour of ESM (browser-based).

So Bun and Deno came up into the Node.js scene as an alternative to help alleviate a lot of developer's frustrations and feature requests.

## Bun

Bun is created by Jarred Sumner whose motivation is to make Javascript faster and simpler to write.

Bun is bringing all Node.js ecosystem in 1 place:

It is a :

1. Javascript Bundler
    
2. Javascript Package Manager
    
3. Test runner
    
4. Can als
    

It also has SQLite and implements Websocket natively. It can also [run Shell scripts](https://bun.sh/docs/runtime/shell) with JS/TS

## Deno

%[https://youtu.be/M3BM9TB-8yA?si=FWxPnWIhRERpf8a4] 

Created by the original creator of Node.js; Ryan Dahl. Deno aims to resolve a lot of regrets by Ryan Dahl.

You can see this talk for more context on the regrets as well as initial design goals for Deno

# Node 22 features

All code can be found here:[  
Code sandbox link](https://codesandbox.io/p/devbox/node-22-blog-3gkgvy)

## Typescript

> Inspired by Deno's [native Typescript support](https://docs.deno.com/runtime/manual/advanced/typescript/overview/)

You can now run `.ts` files in Node, without need of `tsc` or any compiler.

Right now, the feature is very minimal. It only strips way types and does not support a lot of Typescript features. Nonetheless, it is a step in the right direction.

```typescript
let typedVariable : string = "Hello typed Node.js";

console.log(typedVariable)
```

This just works and gives the following output:

```typescript
> nodejs-sandbox@1.0.0 node22-typescript
> NODE_OPTIONS=--disable-warning=DeprecationWarning npx node-nightly --no-warnings --experimental-strip-types features/node-typescript.ts

 New nightly available. To upgrade: `node-nightly --upgrade` 
Hello typed Node.js
```

Code example can be found here:

%[https://codesandbox.io/p/devbox/node-22-blog-3gkgvy?file=%2Ffeatures%2Fnode-typescript.ts%3A1%2C1-3%2C27] 

> You need to enable experimental flag under Node's nightly build:  
> `npx node-nightly --experimental-strip-types features/node-typescript.js`

## ESM and CJS

> Inspired by Bun [Module resolution](https://bun.sh/docs/runtime/modules#using-require)

One of main issues in Node.js packages are the module resolution.

Using both Common.js (Node.js based and legacy) and native ES Modules (newer, browser-based) was a pain.

Bun provided a way to do this when it first released.

In current Node, when you run this code

```javascript
const { esmExample } = require('./esm')
const { cjsExample } = require('./commonJs')

console.log(esmExample('(New)   Hello ESM!'))
console.log(cjsExample('(Legacy)Hello CJS!'))

// esm.js
export function esmExample(str) {
    return str;
}

// commonJs.js
function cjsExample(str) {
    return str;
}

module.exports = { cjsExample }
```

You would get this error:

```bash
export function esmExample(str) {
^^^^^^

SyntaxError: Unexpected token 'export'
```

With Node 22's `--experimental-require-module` , this is a thing of the past.

Running this

%[https://codesandbox.io/p/devbox/node-22-blog-3gkgvy?file=%2Ffeatures%2Fesm-cjs%2Fnode-native-esm.js%3A5%2C46] 

Give this output:

```bash
> NODE_OPTIONS=--disable-warning=DeprecationWarning npx node-nightly --no-warnings --experimental-require-module features/esm-cjs/node-native-esm.js

 New nightly available. To upgrade: `node-nightly --upgrade` 
(New)   Hello ESM!
(Legacy)Hello CJS!
```

Voila, all the pain of using ESM-only packages just goes away! (Hopefully)

## SQLite

> Inspired by Bun's [SQLite Driver](https://bun.sh/docs/api/sqlite)

Can create a database table by importing `node:sqlite` package

```javascript
import { DatabaseSync } from 'node:sqlite'
const database = new DatabaseSync(':memory:')
database.exec(`
 CREATE TABLE node_db (
        id integer primary key,
        name text
    )
`)
```

Insert Scripts can be added using the `prepare` method:

```javascript
const insertScript = database.prepare(`insert into node_db (id, name) values (:id, :name)`)

console.log('----- Inserting data into Node SQLite; John and Mae')
insertScript.run({id: 1, name: 'John'})
insertScript.run({id: 2, name: 'Mae'})
```

If you want to be insecure, you can directly use the `SELECT` and get the data like this:

```javascript
const getDataSQLInjection = database.prepare(`select * from node_db where name = 'John' OR 1=1`)
console.log('----- Getting `John` User from Node SQLite')
console.log('----- 🚩 SQL Injection!!!!')
console.log(getDataSQLInjection.all())
```

And get all user details:

```bash
----- Getting `John` User from Node SQLite
----- 🚩 SQL Injection!!!!
[ { id: 1, name: 'John' }, { id: 2, name: 'Mae' } ]
```

But if you are good developer, you need to parameterise your queries which can be done like this:

```javascript
const getDetailWithoutSQLInjection = database.prepare(`SELECT * FROM node_db WHERE name = :nameParam`);
console.log('----- Getting `John` User from Node SQLite')

console.log('----- ✅ No naive SQL Injection!!!!')
console.log(getDetailWithoutSQLInjection.all({'nameParam': 'John'}))

// Below will not return empty as prepared statement will sanitize 
// console.log(getDetailWithoutSQLInjection.all({'nameParam': 'John OR 1=1'}))
```

```bash
----- Getting `John` User from Node SQLite
----- ✅ No naive SQL Injection!!!!
[ { id: 1, name: 'John' } ]
```

Full code snippet can be found and run here by running `npm run node22-esm`:

> You need to enable experimental flag under Node's nightly build:  
> `npx node-nightly --experimental-sqlite features/node-sqlite.js`

%[https://codesandbox.io/p/devbox/node-22-blog-3gkgvy?file=%2Ffeatures%2Fnode-sqlite.js%3A24%2C29] 

# Closing thoughts

Node has been listening to competition and embracing the features. Let's wait and see how these features evolve.