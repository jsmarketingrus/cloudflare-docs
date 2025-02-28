---
title: ReadableStream BYOBReader
pcx-content-type: configuration
---
<!-- The space in the title was introduced to create a pleasing line-break in the title in the sidebar. -->

# ReadableStreamBYOBReader

<!-- TODO: See EW-2105. Should we document this if it isn’t effectively using buffer space? -->

## Background

`BYOB` is an abbreviation of bring your own buffer. A `ReadableStreamBYOBReader` allows reading into a developer-supplied buffer, thus minimizing copies.

An instance of `ReadableStreamBYOBReader` is functionally identical to [`ReadableStreamDefaultReader`](/runtime-apis/streams/readablestreamdefaultreader) with the exception of the `read` method.

A `ReadableStreamBYOBReader` is not instantiated via its constructor. Rather, it is retrieved from a [`ReadableStream`](/runtime-apis/streams/readablestream):

```js
const { readable, writable } = new TransformStream()
const reader = readable.getReader({ mode: "byob" })
```

## Methods

<Definitions>

- <Code>read(buffer<ParamType>ArrayBufferView</ParamType>)</Code> <TypeLink href="https://streams.spec.whatwg.org/#dictdef-readablestreambyobreadresult">Promise&lt;ReadableStreamBYOBReadResult></TypeLink>

  - Returns a promise with the next available chunk of data read into a passed-in buffer.

- <Code>readAtLeast(minBytes, buffer<ParamType>ArrayBufferView</ParamType>)</Code> <TypeLink href="https://streams.spec.whatwg.org/#dictdef-readablestreambyobreadresult">Promise&lt;ReadableStreamBYOBReadResult></TypeLink>

  - Returns a promise with the next available chunk of data read into a passed-in buffer. The promise will not resolve until at least `minBytes` have been read. 

</Definitions>

## Common issues

  <Aside type="warning" header="Warning">

  `read` provides no control over the minimum number of bytes that should be read into the buffer. Even if you allocate a 1MiB buffer, the kernel is perfectly within its rights to fulfill this read with a single byte, whether or not an EOF immediately follows.

  In practice, the Workers team has found that `read` typically fills only 1% of the provided buffer.

  `readAtLeast` is a non-standard extension to the Streams API which allows users to specify that at least `minBytes` bytes must be read into the buffer before resolving the read.

  </Aside>

## Related resources

- [Using Streams](/learning/using-streams)
- [Background about BYOB readers in the Streams API WHATWG specification](https://streams.spec.whatwg.org/#byob-readers)
