# Zipkin

## 文档

* [zipkin - Github主页](https://github.com/openzipkin/zipkin)
  * [zipkin-server](https://github.com/openzipkin/zipkin/tree/master/zipkin-server) - zipkin-server
    is a Spring Boot application, packaged as an executable jar. You need JRE 8+ to start
    zipkin-server.
  * [zipkin-collector](https://github.com/openzipkin/zipkin/tree/master/zipkin-collector)
    * `ActiveMQ`
    * `Core`
    * `Kafka`
    * `RabbitMQ`
    * `Scribe` - 不推荐
  * [zipkin-junit](https://github.com/openzipkin/zipkin/tree/master/zipkin-junit) - This contains
    ZipkinRule, a JUnit rule to spin-up a Zipkin server during tests
  * [Zipkin-lens](https://github.com/openzipkin/zipkin/tree/master/zipkin-lens) - Zipkin-lens is an
    alternative UI for Zipkin, which based on React, Netflix/vizceral and chartjs.
  * [zipkin-storage](https://github.com/openzipkin/zipkin/tree/master/zipkin-storage)
    * `Cassandra`
    * `ElasticSearch`
    * `MySql` - 不推荐
  * [Zipkin Docker](https://github.com/openzipkin/zipkin/tree/master/docker/examples) -
    使用`docker-compose`简易运行Zipkin需要的各个组件，包含了`Dockerfile`及`examples`
  * [Open Zipkin Example](https://github.com/openzipkin?utf8=%E2%9C%93&q=example) -
    包含部分语言集成Zipkin的Demo

* [Zipkin官网](https://zipkin.io/)
  * [Quickstart](https://zipkin.io/pages/quickstart.html)
  * [Architecture](https://zipkin.io/pages/architecture.html)
  * [Tracers and Instrumentation](https://zipkin.io/pages/tracers_instrumentation.html) - The
    libraries provide instrumentation on various platforms.
  * [Server extensions and choices](https://zipkin.io/pages/extensions_choices.html) -
    Server端的扩展及可替换产品
    * `Apache SkyWalking`
    * `Jaeger`
    * `Pitchfork`
  * [Instrumenting a library](https://zipkin.io/pages/instrumenting.html)

* [zipkin-dependencies](https://github.com/openzipkin/zipkin-dependencies) - Spark job that
  aggregates zipkin spans for use in the UI.

  Zipkin Dependencies collects spans from storage, analyzes links between services, and stores them
  for later presentation in the web UI (ex. `http://localhost:8080/dependency`).

  This process is implemented as an `Apache Spark job`. This job parses all traces in the current
  day in `UTC time`. This means you should schedule it to run just prior to `midnight UTC`.

* [Distributed Tracing with Zipkin and ELK](https://logz.io/blog/zipkin-elk/)
