---
order: 6
pcx-content-type: concept
---

import NetworkMap from "../../components/network-map"
import ArchitectureDiagram from "../../components/architecture-diagram"

# How Workers works

Though Cloudflare Workers behave similar to [JavaScript](https://www.cloudflare.com/learning/serverless/serverless-javascript/) in the browser or in Node.js, there are a few differences in how you have to think about your code. Under the hood, the Workers runtime uses the [V8 engine](https://www.cloudflare.com/learning/serverless/glossary/what-is-chrome-v8/) — the same engine used by Chromium and Node.js. The Workers runtime also implements many of the standard [APIs](/runtime-apis) available in most modern browsers.

The differences between JavaScript written for the browser or Node.js happen at runtime. Rather than running on an individual's machine (for example, [a browser application or on a centralized server](https://www.cloudflare.com/learning/serverless/glossary/client-side-vs-server-side/)), Workers functions run on [Cloudflare's Edge Network](https://www.cloudflare.com/network) - a growing global network of thousands of machines distributed across hundreds of locations.

<figure><NetworkMap/></figure>

Each of these machines hosts an instance of the Workers runtime, and each of those runtimes is capable of running thousands of user-defined applications. This guide will review some of those differences.

The three largest differences are: Isolates, Compute per Request, and Distributed Execution.

## Isolates

[V8](https://v8.dev) orchestrates isolates: lightweight contexts that provide your code with variables it can access and a safe environment to be executed within. You could even consider an isolate a sandbox for your function to run in.

A single runtime can run hundreds or thousands of isolates, seamlessly switching between them. Each isolate's memory is completely isolated, so each piece of code is protected from other untrusted or user-written code on the runtime. Isolates are also designed to start very quickly. Instead of creating a virtual machine for each function, an isolate is created within an existing environment. This model eliminates the cold starts of the virtual machine model.

<figure><ArchitectureDiagram/></figure>

Unlike other serverless providers which use [containerized processes](https://www.cloudflare.com/learning/serverless/serverless-vs-containers/) each running an instance of a language runtime, Workers pays the overhead of a JavaScript runtime once on the start of an edge container. Workers processes are able to run essentially limitless scripts with almost no individual overhead by creating an isolate for each Workers function call. Any given isolate can start around a hundred times faster than a Node process on a container or virtual machine. Notably, on startup isolates consume an order of magnitude less memory.

A given isolate has its own scope, but isolates are not necessarily long-lived. An isolate may be spun down and evicted for a number of reasons:

- resource limitations on the machine.
- a suspicious script - anything seen as trying to break out of the Isolate sandbox.
- individual [resource limits](/platform/limits).

Because of this, it is generally advised that you not store mutable state in your global scope unless you have accounted for this contingency.

If you are interested in how Cloudflare handles security with the Workers runtime, you can [read more about how Isolates relate to Security and Spectre Threat Mitigation](/learning/security-model).

## Compute per request

Most Workers scripts are a variation on the default Workers flow:

```js
addEventListener("fetch", event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  return new Response("Hello worker!", { status: 200 })
}
```

When a request to your `*.workers.dev` subdomain or to your Cloudflare-managed domain is received by any of Cloudflare's runtimes, the Workers script is passed a [`FetchEvent`](/runtime-apis/fetch-event) argument to the event handler defined in the script. From there you can generate a [`Response`](/runtime-apis/response) by computing a response on the spot, calling to another server using [`fetch`](/runtime-apis/fetch), etc.. The CPU cycles needed to get to the point of the `respondWith` call all contribute to the compute time. For example, a `setInterval` timeout does not consume CPU cycles while waiting.

## Distributed execution

Isolates are resilient and continuously available for the duration of a request, but in rare instances isolates may be evicted. When a script hits official [limits](/platform/limits) or when resources are exceptionally tight on the machine the request is running on, the runtime will selectively evict isolates after their events are properly resolved.

Like all other JavaScript platforms, a single Workers instance may handle multiple requests including concurrent requests in a single-threaded event loop. There is no guarantee whatsoever whether any two requests will land in the same instance; therefore it is inadvisable to set or mutate global state within the event handler.

## Related resources

Learn more about:

- [FetchEvents](/runtime-apis/fetch-event)
- [Request context](/runtime-apis/request)
- [Runtime limitations](/platform/limits)
