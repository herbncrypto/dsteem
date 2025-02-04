
# [dsmoke](https://github.com/herbncrypto/dsmoke) [![Build Status](https://img.shields.io/circleci/project/github/herbncrypto/dsmoke.svg?style=flat-square)](https://circleci.com/gh/herbncrypto/workflows/dsmoke) [![Coverage Status](https://img.shields.io/coveralls/jnordberg/dsteem.svg?style=flat-square)](https://coveralls.io/github/jnordberg/dsteem?branch=master) [![Package Version](https://img.shields.io/npm/v/dsmoke.svg?style=flat-square)](https://www.npmjs.com/package/dsmoke)

Robust [smoke blockchain](https://smoke.io) client library that runs in both node.js and the browser.

* [Bug tracker](https://github.com/herbncrypto/dsmoke/issues)

---

**note** As of version 0.7.0 WebSocket support has been removed. The only transport provided now is HTTP(2). For most users the only change required is to swap `wss://` to `https://` in the address. If you run your own full node make sure to set the proper [CORS headers](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) if you plan to access it from a browser.

---


Browser compatibility
---------------------

[![Build Status](https://saucelabs.com/browser-matrix/jnordberg-dsteem.svg)](https://saucelabs.com/open_sauce/user/jnordberg-dsteem)


Installation
------------

### Via npm

For node.js or the browser with [browserify](https://github.com/substack/node-browserify) or [webpack](https://github.com/webpack/webpack).

```
npm install dsmoke
```

### From cdn or self-hosted script

Grab `dist/dsmoke.js` from a [release](https://github.com/jnordberg/dsmoke/releases) and include in your html:

```html
<script src="dsmoke.js"></script>
```

Or from the [unpkg](https://unpkg.com) cdn:

```html
<script src="https://unpkg.com/dsmoke@^0.8.0/dist/dsmoke.js"></script>
```

Make sure to set the version you want when including from the cdn, you can also use `dsmoke@latest` but that is not always desirable. See [unpkg.com](https://unpkg.com) for more information.


Usage
-----

### In the browser

```html
<script src="https://unpkg.com/dsmoke@latest/dist/dsmoke.js"></script>
<script>
    var client = new dsmoke.Client('https://rpc.smoke.io')
    client.database.getDiscussions('trending', {tag: 'smoking', limit: 1}).then(function(discussions){
        document.body.innerHTML += '<h1>' + discussions[0].title + '</h1>'
        document.body.innerHTML += '<h2>by ' + discussions[0].author + '</h2>'
        document.body.innerHTML += '<pre style="white-space: pre-wrap">' + discussions[0].body + '</pre>'
    })
</script>
```

See the [demo source](https://github.com/herbncrypto/dsmoke/tree/master/examples/comment-feed) for an example on how to setup a livereloading TypeScript pipeline with [wintersmith](https://github.com/jnordberg/wintersmith) and [browserify](https://github.com/substack/node-browserify).

### In node.js

With TypeScript:

```typescript
import {Client} from 'dsmoke'

const client = new Client('https://rpc.smoke.io')

for await (const block of client.blockchain.getBlocks()) {
    console.log(`New block, id: ${ block.block_id }`)
}
```

With JavaScript:

```javascript
var dsmoke = require('dsmoke')

var client = new dsmoke.Client('https://rpc.smoke.io')
var key = dsmoke.PrivateKey.fromLogin('username', 'password', 'posting')

client.broadcast.vote({
    voter: 'your-username',
    author: 'herbncrypto',
    permlink: 'the-highest-point-of-my-week',
    weight: 10000
}, key).then(function(result){
   console.log('Included in block: ' + result.block_num)
}, function(error) {
   console.error(error)
})
```

With ES2016 (node.js 7+):

```javascript
const {Client} = require('dsmoke')

const client = new Client('https://rpc.smoke.io')

async function main() {
    const props = await client.database.getChainProperties()
    console.log(`Maximum blocksize consensus: ${ props.maximum_block_size } bytes`)
    client.disconnect()
}

main().catch(console.error)
```

With node.js streams:

```javascript
var dsmoke = require('dsmoke')
var es = require('event-stream') // npm install event-stream
var util = require('util')

var client = new dsmoke.Client('https://rpc.smoke.io')

var stream = client.blockchain.getBlockStream()

stream.pipe(es.map(function(block, callback) {
    callback(null, util.inspect(block, {colors: true, depth: null}) + '\n')
})).pipe(process.stdout)
```


Bundling
--------

The easiest way to bundle dsmoke (with browserify, webpack etc.) is to just `npm install dsmoke` and `require('dsmoke')` which will give you well-tested (see browser compatibility matrix above) pre-bundled code guaranteed to JustWork™. However, that is not always desirable since it will not allow your bundler to de-duplicate any shared dependencies dsmoke and your app might have.

To allow for deduplication you can `require('dsmoke/lib/index-browser')`, or if you plan to provide your own polyfills: `require('dsmoke/lib/index')`. See `src/index-browser.ts` for a list of polyfills expected.

---

*Share and Enjoy!*
