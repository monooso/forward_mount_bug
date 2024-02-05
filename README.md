# Assigns bug in forwarded routes
A single-file Phoenix application which demonstrates a bug when using the `Phoenix.router.forward/4` macro with a LiveView.

## Summary
When forwarding a route to another router, values assigned to a socket in a `live_session` `on_mount` handler are discarded when the socket is connected.
The values are present when the LiveView is first mounted.

## Detailed explanation
The repository contains two routers. The first router (`Example.Router`) simply forwards all requests for the `/forwarded` route to the second router (`Example.ForwardedRouter`).

`Example.ForwardedRouter` defines a single `live` route, inside a `live_session` block. The `live_session` block sets the `on_mount` option to `{Example.MountHandler, :default}`.

The `Example.MountHandler.on_mount/4` function sets the value of `socket.assigns.test_key` to "Test value".

The `Example.HomeLive.mount/3` function sets the value of `socket.assigns.test_key` to "Fallback value" _if it is not already assigned_ (using `assign_new`).

When loading the `/forwarded` route in the browser, you will briefly see the text "Test value". This is replaced by "Fallback value" once the socket is connected.

It is important to note that the bug described above only manifests under the following specific circumstances:

1. The route is forwarded from the main router to a second router. If you use a single router, everything works.
2. The forwarded route is not `/`. If you forward the `/` route, everything works.
3. The second router defines the `on_mount` handler as a `live_session` option. If the LiveView itself defines the `on_mount` handler, everything works.

