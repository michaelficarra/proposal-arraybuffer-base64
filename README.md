# ArrayBuffer to/from base64

base64 is the de-facto standard way to represent arbitrary binary data as ASCII. JavaScript has ArrayBuffers (and other wrapping types) to work with binary data, but no way built-in mechanism to encode that data as base64, nor to take base64'd data and produce a corresponding ArrayBuffer. We should fix that.

## Possible API

```js
let buffer = new ArrayBuffer(42);
// populate buffer here
console.log(buffer.toBase64());
```
and
```js
let buffer = ArrayBuffer.fromBase64('AQMDBw=='); // returns a new length-4 ArrayBuffer
```

That is, I propose to add `ArrayBuffer.prototype.toBase64` and `ArrayBuffer.fromBase64` methods. The latter would throw if given a string which is not properly base64 encoded.

## Why not just use `atob` and `btoa`?

Those methods are take and consume strings, rather than translating between a string and an ArrayBuffer.

## Why not TextEncoder?

base64 is not a text encoding format; there's no [code points](https://unicode.org/glossary/#code_point) involved. So despite fitting with the type signature of TextEncoder/TextDecoder, base64 encoding and decoding is not a conceptually appropriate thing for those APIs to do.

That's also been the consensus when it's come up [previously](https://discourse.wicg.io/t/base64-with-textencoder-textdecoder/1307/2).

## But _which_ base64?

An excellent question. Unfortunately, I think we're probably stuck with using the thing called "base64" in [the RFC](https://datatracker.ietf.org/doc/html/rfc4648#section-4) as the default; that is, using `+` and `/` to represent 62 and 63, and mandatory padding with `=`. I don't think these are good choices, but interop requires they be the default.

We could potentially add an options bag argument allowing these details to be configured, as in
```js
buffer.toBase64({ variant: 'base64url', padding: false });

ArrayBuffer.fromBase64('_w', { variant: 'base64url', padding: false });
```