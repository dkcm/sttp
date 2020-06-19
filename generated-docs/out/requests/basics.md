# Request definition basics

As mentioned in the [quickstart](../quickstart.md), the following import will be needed:

```scala
import sttp.client._
```

This brings into scope `basicRequest`, the starting request. This request can be customised, each time yielding a new, immutable request definition (unless a mutable body is set on the request, such as a byte array). As the request definition is immutable, it can be freely stored in values, shared across threads, and customized multiple times in various ways.

For example, we can set a cookie, `String` -body and specify that this should be a `POST` request to a given URI:

```scala
val request = basicRequest
    .cookie("login", "me")
    .body("This is a test")
    .post(uri"http://endpoint.com/secret")
```

The request parameters (headers, cookies, body etc.) can be specified **in any order**. It doesn't matter if the request method, the body, the headers or connection options are specified in this sequence or another. This way you can build arbitrary request templates, capturing all that's common among your requests, and customizing as needed. Remember that each time a modifier is applied to a request, you get a new immutable object.

There's a lot of ways in which you can customize a request, which are covered in this guide. Another option is to just explore the API: most of the methods are self-explanatory and carry scaladocs if needed.

Using the modifiers, each time we get a new request definition, but it's just a description: a data object; nothing is sent over the network until the `send()` method is invoked.

## Query parameters and URIs

Query parameters are specified as part of the URI, to which the request should be sent. The URI can only be set together with the request method (using `.get(Uri)`, `.post(Uri)`, etc.).

The URI can be created programatically (by calling methods on the `Uri` class), or using the `uri` interpolator, which also allows embedding (and later escaping) values from the environment. See the documentation on [creating URIs](../model/uri.md) for more details.

## Sending a request

A request definition can be created without knowing how it will be sent. But to send a request, a backend is needed. A default, synchronous backend based on Java's `HttpURLConnection` is provided out-of-the box.

To invoke the `send()` method on a request description, an implicit value of type `SttpBackend` needs to be in scope:

```scala
implicit val backend = HttpURLConnectionBackend()
val response: Response[String] = request.send()
```        

The default backend doesn't wrap the response into any container, but other asynchronous backends might do so. See the section on [backends](../backends/summary.md) for more details.

Alternatively, if you prefer to pass the backend explicitly, instead of using implicits, you can also send the request the following way:

```scala
val backend = HttpURLConnectionBackend()
val response = backend.send(request)     
```

```eval_rst
.. note::

  Only requests with the request method and uri can be sent. If trying to send a request without these components specified, a compile-time error will be reported. On how this is implemented, see the documentation on the `type of request definitions <type.md>`_.
```

## Initial requests

sttp provides two initial requests:

* `basicRequest`, which is an empty request with the `Accept-Encoding: gzip, deflate` header added. That's the one that is most commonly used.
* `emptyRequest`, a completely empty request, with no headers at all.

Both of these requests will by default read the response body into a UTF-8 `String`. How the response body is handled is also part of the request definition. See the section on [response body specifications](../responses/body.md) for more details on how to customize that.

## Debugging requests

sttp comes with builtin request to curl converter. To convert request to curl invocation use `.toCurl` method.

For example converting given request:

```scala
basicRequest.get(uri"http://httpbin.org/ip").toCurl
```

will result in following curl command:

```scala
curl -L --max-redirs 32 -X GET "http://httpbin.org/ip"
```

Note that the `Accept-Encoding` header, which is added by default to all requests (`Accept-Encoding: gzip, deflate`) is filtered out from the generated command, so that when running a request from the command line, the result has higher chance of being human-readable, and not compressed.