# Viết mã nguồn Universal

Before going further, let's take a moment to discuss the constraints when writing "universal" code - that is, code that runs on both the server và the client. Due to use case và platform API differences, the behavior of our code will not be exactly the same when running in different environments. Here we will go over the key things you need to be aware of.

## Data Reactivity phía Server

In a client-only app, every user will be using a fresh instance of the app in their browser. For server-side rendering we want the same: each request should have a fresh, isolated app instance so that there is no cross-request state pollution.

Because the actual rendering process needs to be deterministic, we will also be "pre-fetching" data on the server - this means our application state will be already resolved when we start rendering. This means data reactivity is unnecessary on the server, so it is disabled by default. Disabling data reactivity also avoids the performance cost of converting data into reactive objects.

## Vòng đời Component

Since there are no dynamic updates, of all the lifecycle hooks, only `beforeCreate` và `created` will be called during SSR. This means any code inside other lifecycle hooks such as `beforeMount` or `mounted` will only be executed on the client.

Another thing to note is that you should avoid code that produces global side effects in `beforeCreate` và `created`, ví dụ như setting up timers with `setInterval`. In client-side only code we may setup a timer và then tear it down in `beforeDestroy` or `destroyed`. However, because the destroy hooks will not be called during SSR, the timers will stay around forever. To avoid this, move your side-effect code into `beforeMount` or `mounted` instead.

## Truy cập vào các API đặc trưng trên các nền tảng

Universal code cannot assume access to platform-specific APIs, so if your code directly uses browser-only globals like `window` or `document`, they will throw errors when executed in Node.js, và vice-versa.

For tasks shared between server và client but use different platform APIs, Khuyến khích to wrap the platform-specific implementations inside a universal API, or use libraries that do this for you. For example, [axios](https://github.com/axios/axios) is an HTTP client that exposes the same API for both server và client.

For browser-only APIs, the common approach is to lazily access them inside client-only lifecycle hooks.

Note that if a 3rd party library is not written with universal usage in mind, it could be tricky to integrate it into an server-rendered app. You *might* be able to get it working by mocking some of the globals, but it would be hacky và may interfere with the environment detection code of other libraries.

## Chỉ thị tùy biến

Most custom directives directly manipulate the DOM, và therefore will cause error during SSR. There are two ways to work around this:

1. Prefer using components as the abstraction mechanism và work at the Virtual-DOM level (e.g. using render functions) instead.

2. If you have a custom directive that cannot be easily replaced by components, you can provide a "server-side version" of it using the [`directives`](./api.md#directives) option when creating the server renderer.
