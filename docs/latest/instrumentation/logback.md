---
layout: docs
title: 'Logging with Context | Kamon Documentation'
redirect_from:
  - /integrations/logback/mdc-in-an-asyncronous-environment/
---

{% include toc.html %}

Logback Instrumentation
=======================

The Logback instrumentation provides a few useful features that help include Kamon's Context information in your
logging practices: it makes it easy to include Context information in your log patterns and on the MDC, it preservers
Context when using an `AsyncAppender` and finally, comes with an appender that can be used to count log entries by level.


Logging with Context
--------------------

There are four converters included in the Logback instrumentation that can help you include Context information in your
log patterns:

- **TraceIDConverter** outputs the current trace identifier, or `undefined` if none is present.
- **SpanIDConverter** outputs the current span identifier, or `undefined` if none is present.
- **ContextTagConverter** outputs the value of a specified Context tag, or a default value if the tag is not present. It
  accepts one parameter that specifies the name of the Context tag to use.
- **ContextEntryConverter** outputs the value of a specified Context entry, or a default value if the tag is not
  present. It accepts one parameter that specifies the name of the Context entry to use.

Converters need to be registered manually by adding these conversion rulers to your `logback.xml` file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="false" debug="false">
  <conversionRule conversionWord="traceID" converterClass="kamon.instrumentation.logback.tools.TraceIDConverter" />
  <conversionRule conversionWord="spanID" converterClass="kamon.instrumentation.logback.tools.SpanIDConverter" />
  <conversionRule conversionWord="contextTag" converterClass="kamon.instrumentation.logback.tools.ContextTagConverter" />
  <conversionRule conversionWord="contextEntry" converterClass="kamon.instrumentation.logback.tools.ContextEntryConverter" />

  <!-- the rest of your config... -->

</configuration>
```

Once your conversion rules are in place, all you need to do is decide where you include them in your log patter. For
example, in the pattern bellow we are including the current Trace and Span identifiers, as well as the value of the
`user.id` Context tag:


Inserting a `conversionRule` allows you to incorporate the trace ID for a request into your [Logback Layout][logback layout].
Here is a simple example `logback.xml` configuration that does this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="false" debug="false">
  <conversionRule conversionWord="traceID" converterClass="kamon.instrumentation.logback.tools.TraceIDConverter" />

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d | %level | %traceID %spanID | %contextTag{user.id} | %m%n</pattern>
    </encoder>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```


Copying Context to the MDC
--------------------------

Since many diagnostic tools might rely on the MDC to gather additional information, Kamon includes instrumentation that
can automatically copy data from Kamon's Context into the current MDC while processing a log statement, so that this
data can be transparently captured and processed by other tools. Here is an extract from the module configuration:

```text
kamon.instrumentation.logback {

  # Controls if and how Context data should be copied into the MDC while events
  # are being logged.
  #
  mdc {

    # MDC keys used to store the current trace and span identifiers. These keys
    # will only be copied if there is a non-empty Span in the Context associated
    # with the logged event.
    trace-id-key = "kamonTraceId"
    span-id-key = "kamonSpanId"

    # Enables copying of Context information into the MDC. Please note that if
    # you only want to include certain Context information in your log patterns
    # you are better off by simply using the conversion rules available under
    # the "tools" package. Copying data into the MDC is required only in cases
    # where third-party tooling expects data from the MDC to be extracted.
    #
    copy {

      # Controls whether Context information should be copied into the MDC
      # or not.
      enabled = yes

      # Controls whether Context tags should be copied into the MDC.
      tags = yes

      # Contains the names of all Context entries that should be copied into
      # the MDC.
      entries = [ ]
    }
  }
}
```


Counting Log Events
-------------------

Finally, there is a very simple appender that can be used to count how many log events where generated by your
application by adding the `EntriesCounterAppender` to your configuration:


```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="false" debug="false">

  <!-- your conversion rulers, appenders and so on -->

  <appender name="COUNTER" class="kamon.instrumentation.logback.tools.EntriesCounterAppender"/>

  <root level="DEBUG">
    <appender-ref ref="REAL_APPENDER" />
    <appender-ref ref="COUNTER" />
  </root>
</configuration>
```

Once the appender is added, you will start to see the following metric:

{%  include metric-detail.md name="log.events" %}


Manual Installation
-------------------

In case you are not using the Kamon Bundle, add the dependency bellow to your build.

{% include dependency-info.html module="kamon-logback" version=site.data.versions.latest.logback %}
{% include instrumentation-agent-notice.html %}