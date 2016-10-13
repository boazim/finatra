---
layout: user_guide
title: "Implement a Server \"Warmup\" Handler"
comments: false
sharing: false
footer: true
---

<ol class="breadcrumb">
  <li><a href="/finatra/user-guide">User Guide</a></li>
  <li><a href="/finatra/user-guide/build-new-http-server">Building a New HTTP Server</a></li>
  <li class="active">Implement a Server "Warmup" Handler</li>
</ol>

## Basics
===============================

There may be occasions where we want to exercise specific code paths before accepting traffic to the server (e.g.w, for triggering JIT in the JVM). In this case you can implement a [`c.t.inject.utils.Handler`](https://github.com/twitter/finatra/blob/develop/inject/inject-utils/src/main/scala/com/twitter/inject/utils/Handler.scala). Your handler should be constructed with an [`c.t.finatra.http.routing.HttpWarmup`](https://github.com/twitter/finatra/blob/develop/http/src/main/scala/com/twitter/finatra/http/routing/HttpWarmup.scala) instance.

```scala
import com.twitter.finatra.http.routing.HttpWarmup
import com.twitter.finatra.httpclient.RequestBuilder._
import com.twitter.inject.utils.Handler
import javax.inject.Inject

class ExampleWarmupHandler @Inject()(
  httpWarmup: HttpWarmup)
  extends Handler {

  override def handle() = {
    httpWarmup.send(
      get("/ping"))
  }
}
```
<div></div>

The handle above simply tries to hit the `/ping` endpoint of the server.

You can then run this handler in the `warmup` lifecycle method:

```scala
import DoEverythingModule
import ExampleController
import ExampleFilter
import ExampleWarmupHandler
import MalformedURLExceptionMapper
import com.twitter.finatra.http.HttpServer
import com.twitter.finatra.http.filters.{CommonFilters, LoggingMDCFilter, TraceIdMDCFilter}
import com.twitter.finatra.http.routing.HttpRouter

object ExampleServerMain extends ExampleServer

class ExampleServer extends HttpServer {

  override val modules = Seq(
    DoEverythingModule)

  override def configureHttp(router: HttpRouter): Unit = {
    router
      .filter[LoggingMDCFilter[Request, Response]]
      .filter[TraceIdMDCFilter[Request, Response]]
      .filter[CommonFilters]
      .add[ExampleFilter, ExampleController]
      .exceptionMapper[MalformedURLExceptionMapper]
  }

  override def warmup() {
    handle[ExampleWarmupHandler]()
  }
}
```
<div></div>

Once again, the warmup handler is added *by type* allowing the framework to construct the instance.

The [`c.t.inject.app.App#warmup`](https://github.com/twitter/finatra/blob/develop/inject/inject-app/src/main/scala/com/twitter/inject/app/App.scala#L122) lifecycle method is called before the server's external HTTP port(s) are bound and thus before the TwitterServer [Lifecycle Management](http://twitter.github.io/twitter-server/Features.html#lifecycle-management) `/health` endpoint responds with `OK`. See [here](/finatra/user-guide/getting-started#lifecycle) for more information on the lifecycle of a Finatra server.

## <a class="anchor" name="more-information" href="#more-information">More information</a>
===============================

For more information, we encourage you to take a look at the full [`finatra/examples`](https://github.com/twitter/finatra/tree/master/examples) in the [github](https://github.com/twitter/finatra) source.

<nav>
  <ul class="pager">
    <li class="previous"><a href="/finatra/user-guide/build-new-http-server/exceptions.html"><span aria-hidden="true">&larr;</span>&nbsp;Add&nbsp;an&nbsp;ExceptionMapper</a></li>
    <li class="next"><a href="/finatra/user-guide/json">Working&nbsp;with&nbsp;JSON&nbsp;<span aria-hidden="true">&rarr;</span></a></li>
  </ul>
</nav>
