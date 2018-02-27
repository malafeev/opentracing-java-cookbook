# Java OpenTracing Cookbook

## Setup

OpenTracing stack consists of two components:
1. Server (UI, collector, storage) 
1. Client (tracer)  

Tracer sends information about spans to server via grpc, thrift or any other transport.


### Jaeger
One of the popular java OpenTracing implementation is [Jaeger](http://jaeger.readthedocs.io/en/latest/).  
Server part can be started in a docker: http://jaeger.readthedocs.io/en/latest/getting_started/#all-in-one-docker-image  
Tracer is created via `Builder` or `Configuration`: https://github.com/jaegertracing/jaeger-client-java/blob/master/jaeger-core/README.md  

## GlobalTracer
Some instrumented libraries requires GlobalTracer which is located in `opentracing-util` package:
```xml
<dependency>
    <groupId>io.opentracing</groupId>
    <artifactId>opentracing-util</artifactId>
    <version>VERSION</version>
</dependency>
```

Register your tracer with GlobalTracer:
```java
GlobalTracer.register(tracer);
```

## TracerResolver
[TracerResolver API](https://github.com/opentracing-contrib/java-tracerresolver) is designed 
to fix problem with dynamic loading of tracer.  
Right now only [Jaeger](https://github.com/jaegertracing/jaeger-client-java/tree/master/jaeger-tracerresolver) support it. 

## Manual Span Creation
### Scope API
Scope API allows to create span encapsulated into Scope object via ScopeManager:
```java
Scope scope = tracer.buildSpan("operationName").startActive(true);
Span span = scope.span();
```
`startActive()` method takes boolean flag to determine whether span should automatically be finished when `scope.close()` is called.
 
Active scope is accessible via `tracer.scopeManager().active()`.   
Active span is accessible via `tracer.activeSpan()` which is shorthand for `tracer.scopeManager().active().span()`.
  

### Without registration in ScopeManager
```java
Span span = tracer.buildSpan("operationName").start();
```
Created span can be activated via 
```java
Scope scope = tracer.scopeManager().activate(span, true));
```

## Inject/Extract SpanContext
To propagate span context between processes it should be injected/extracted into/from carrier.  
Example: http client injects span context into http request headers:
```java
tracer.inject(span.context(), Format.Builtin.HTTP_HEADERS, new TextMapInjectAdapter(request.getHeaders()));
```
Http server extracts span context from http request headers and builds child span:
```java
SpanContext parentContext = tracer.extract(Format.Builtin.HTTP_HEADERS, new TextMapExtractAdapter(request.getHeaders()));
Scope scope = tracer.buildSpan("receive").asChildOf(parentContext).startActive(true);
```

## Thread safety
Specification doesn't say anything about thread safety on span operations.  
Therefore it depends on tracer implementation.  
But usually spans are transferred to another thread.  
Although Scope should never be passed to another thread. 

## Instrumented libraries
Instrumented libraries are hosted in [OpenTracing API Contributions](https://github.com/opentracing-contrib/).


