.. :changelog:

History
-------

0.12.0 (2016-08-22)
-------------------

* Renamed the browser global symbol from `Tracer` to `opentracing`

0.10.3 (2016-06-12)
-------------------

- Use the `referee` terminology within `Reference`


0.10.2 (2016-06-12)
-------------------

- Fix a bug with the treatment of `startSpan`'s `fields.childOf`


0.10.1 (2016-06-12)
-------------------

- Move to SpanContext, Reference, and `s/join/extract/g`


0.10.0
------

This release makes the `opentracing-javascript` package conformant with the ideas proposed in https://github.com/opentracing/opentracing.github.io/issues/99. The API changes can be summarized as follows:

- Every `Span` has a `SpanContext`, available via `Span.context()`. The `SpanContext` represents the subset of `Span` state that must propagate across process boundaries in-band along with the application data.
- `Span.setBaggageItem()` and `Span.getBaggageItem()` have moved to `SpanContext`. Calls can be migrated trivially: `Span.context().{set,get}BaggageItem()`.
- The first parameter to `Tracer.inject()` is now a `SpanContext`. As a convenience, a `Span` may be passed instead.
- There is a new concept called a `Reference`; a reference describes a relationship between a newly started `Span` and some other `Span` (via a `SpanContext`). The common case is to describe a reference to a parent `Span` that depends on the child `Span` ('REFERENCE_CHILD_OF`).
- `Tracer.startSpan(operation, fields)` no longer accepts `fields.parent`; it now accepts either `fields.childOf`, a `SpanContext` or `Span` instance, or `fields.references`, an array of one or more `Reference` objects. The former is just a shorthand for the latter.
- `Tracer.join(operationName, format, carrier)` has been removed from the API. In its place, use `Tracer.extract(format, carrier)` which returns a `SpanContext`, and pass that `SpanContext` as a reference in `Tracer.startSpan()`.

TL;DR, to start a child span, do this:

::

    let parentSpan = ...;
    let childSpan = Tracer.startSpan('child op', { childOf : parentSpan });

... and to continue a trace from the server side of an RPC, do this:

::

    let format = ...;  // same as for Tracer.join()
    let carrier = ...;  // same as for Tracer.join()
    let extractedCtx = Tracer.extract(format, carrier);
    let serverSpan = Tracer.startSpan('...', { childOf : extractedCtx });
