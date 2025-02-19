<p align="center">
    <img src="https://user-images.githubusercontent.com/9938337/31054113-7cac93d8-a6a3-11e7-84ae-e98c57129a72.png" alt="Leaf Error Middleware">
    <br>
    <br>
    <a href="https://swift.org">
        <img src="http://img.shields.io/badge/Swift-5.2-brightgreen.svg" alt="Language">
    </a>
    <a href="https://github.com/brokenhandsio/leaf-error-middleware/actions">
        <img src="https://github.com/brokenhandsio/leaf-error-middleware/workflows/CI/badge.svg?branch=main" alt="Build Status">
    </a>
    <a href="https://codecov.io/gh/brokenhandsio/leaf-error-middleware">
        <img src="https://codecov.io/gh/brokenhandsio/leaf-error-middleware/branch/main/graph/badge.svg" alt="Code Coverage">
    </a>
    <a href="https://raw.githubusercontent.com/brokenhandsio/leaf-error-middleware/main/LICENSE">
        <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License">
    </a>
</p>

Leaf Error Middleware is a piece of middleware for [Vapor](https://github.com/vapor/vapor) which allows you to return custom 404 and server error pages.

Note that this middleware is designed to be used for Leaf front-end websites only - it should not be used for providing JSON error responses for an API, for example.

# Usage

First, add LeafErrorMiddleware as a dependency in your `Package.swift` file:

```swift
dependencies: [
    // ...,
    .package(name: "LeafErrorMiddleware", url: "https://github.com/brokenhandsio/leaf-error-middleware.git", from: "3.0.0")
],
targets: [
    .target(name: "App", dependencies: ["Vapor", ..., "LeafErrorMiddleware"]),
    // ...
]
```

## Default Context

To use the LeafErrorMiddleware with the default context passed to templates, register the middleware service in `configure.swift` to your `Application`'s middleware (make sure you `import LeafErrorMiddleware` at the top):

```swift
app.middleware.use(LeafErrorMiddlewareDefaultGenerator.build())
```

Make sure it appears before all other middleware to catch errors.

## Custom Context

Leaf Error Middleware allows you to pass a closure to `LeafErrorMiddleware` to generate a custom context for the error middleware. This is useful if you want to be able to tell if a user is logged in on a 404 page for instance.

Register the middleware as follows:

```swift
let leafMiddleware = LeafErrorMiddleware() { status, error, req -> EventLoopFuture<SomeContext> in
    return req.eventLoop.future(SomeContext())
}
app.middleware.use(leafMiddleware)
```

The closure receives three parameters:

* `HTTPStatus` - the status code of the response returned.
* `Error` - the error caught to be handled.
* `Request` - the request currently being handled. This can be used to log information, make external API calls or check the session.

# Setting Up

You need to include two [Leaf](https://github.com/vapor/leaf) templates in your application:

* `404.leaf`
* `serverError.leaf`

When Leaf Error Middleware catches a 404 error, it will return the `404.leaf` template. Any other error caught will return the `serverError.leaf` template. 

## Default Context

If using the default context, the `serverError.leaf` template will be passed up to three parameters in its context:

* `status` - the status code of the error caught
* `statusMessage` - a reason for the status code
* `reason` - the reason for the error, if known. Otherwise this won't be passed in.

The `404.leaf` template will get a `reason` parameter in the context if one is known.
