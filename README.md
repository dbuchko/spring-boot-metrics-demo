# Spring Boot with PCF Metrics Demo

Application demonstrating Logging, Metrics, and Tracing functionality with Spring Boot 2.0 and PCF Metrics.

## Demo Steps

### 1. Deploy to PCF

Build the application:

```sh
./gradlew build
```

Deploy to PCF using the CLI:

Note: using random route in-case of pre-existing route.

```sh
cf push metrics-demo --random-route -p build/libs/spring-boot-metrics-demo-0.0.1-SNAPSHOT.jar
```

### 2. Create and Bind the Forwarder Service

#### Ensure *Metric Forwarder* service is available in the CF MarketPlace

```sh
cf marketplace
```

Contact your PCF Cloud Ops team if it is not.

#### Create the Service

You can use a *plan* and *name* of your choice.

```sh
cf create-service metrics-forwarder unlimited myforwarder
```

#### Bind the Service to your Application

```sh
cf bind-service metrics-demo myforwarder
```

#### Restage your Application

```sh
cf restage metrics-demo
```

### 3. Create a Slack Incoming WebHook URL

Required for Alerting Functionality.

One can be created by logging into your Slack account at www.slack.com, browsing the *App Directory* for *Incoming WebHooks* and adding your own configuration.

You should then be able to send Slack messages to yourself by *posting* to that URL.

Example: (note INSERT_YOUR_WEB_HOOK_URL -- update this with your URL)

```sh
curl -s -d "payload={\"text\":\"Test Message\"}" INSERT_YOUR_WEB_HOOK_URL_HERE
```

To demo the Application Error Level Log Alerting with deployed PCF apps, make sure your PCF app instance has the *SLACK_INCOMING_WEB_HOOK* environment variable set to your URL.

To add this value using the CLI (update INSERT_YOUR_WEB_HOOK_URL_HERE accordingly)

```sh
cf set-env metrics-demo SLACK_INCOMING_WEB_HOOK INSERT_YOUR_WEB_HOOK_URL_HERE
```

You will need to re-stage after this .. 

### 4. Demo the functionality

Functionality is available from the default application path / route.

  1. Metrics
  2. Tracing
  3. Logging

## Metrics - Spring Boot

### Background

As of Spring Boot 2.0, the *Micrometer* application metrics facade is used under-the-hood to provide multidimensional (MDM) vendor-neutral metrics.

Built-in support includes: Prometheus, Netflix Atlas, CloudWatch, Datadog, Graphite, Ganglia, JMX, Influx/Telegraf, New Relic, StatsD, SignalFx, Wavefront, and PCF Metrics.

### Requirements

The *org.springframework.boot:spring-boot-starter-actuator* dependency is required in your build script for Application Metrics in your application.

### The Metrics Actuator Endpoint

Metrics are exposed via the *actuator/metrics* endpoint.

Individual metrics are viewable via HTTP , for example:

actuator/metrics/http.server.requests

A variety of application metrics are included by default.

Note that most actuator endpoints are restricted / locked down by default , hence the change in the *application.properties* file to expose all of them (not recommended in production)

```properties
management.endpoints.web.exposure.include=*
```

With Micrometer multi-dimensionality (MDM) is supported via *tag* , for example we can view metrics for all http server requests with status of 500 using:

actuator/metrics/http.server.requests?tag=status:500

### Custom Application Metrics

Custom metrics can be added by using the *Metrics* object. See XX for code examples.

## Metrics - PCF

Basic application metrics support is included in PCF Metrics. This includes dashboard support for build-in application metrics as well as custom ones.

Note MDM is currently not supported.

### Requirements

To forward application metrics, the PCF Metrics Forwarder Service is needed, and needs to be bound to your application.

## Metrics - PCF - Alerts

As of PCF Metrics 1.5, basic alerting is supported on metrics and events (including App crashes).

## Tracing - Spring Boot

### Requirements

The *org.org.springframework.cloud:spring-cloud-sleuth* dependency is required in your build script for trace information in your logfiles.

## Tracing - PCF

PCF Metrics will automatically recognize the trace and span headers generated by Spring Cloud Sleuth, and will use them for visualization information in the Trace Explorer.

To access the Trace Explorer - click on the "View in Trace Explorer" in the Log View in PCF Metrics.

Note: Custom Span are NOT supported.

## Logging - Spring Boot

### Requirements

The *org.springframework.boot:spring-boot-starter-actuator* dependency is required in your build script for Actuator Logfile endpoint to work.

### The Logfile Actuator Endpoint

For the logfile endpoint to be enabled, to application needs to be configured to save logs to a local flat file (not a default). This can be configured either in application.properties or logback.xml.

## Logging - Sprint Boot - Alerts

At the application level, you can configure logback to send alerts on specific log conditions.

In the case of this example, we have configured the *logabck.xml* file to send ERROR level application log message alerts to a Slack Incoming WebHook.

Note the usage of VCAP environment variables in the *logback.xml* file to show PCF environment values in the message alerts (i.e. Application and Space name).

For local testing of Slack Alerting , set the SLACK_INCOMING_WEB_HOOK environment variable.

### Requirements

The *com.github.maricn:logback-slack-appender:1.4.0* dependency is required in your build script for alerting to Slack in your application.

## Logging - PCF

All STDIO logs from applications deployed to PCF are automatically are stored in centralized logging and are viewable in PCF Metrics.

Default storage retention is 2 weeks.

### Local Machine Access to PCF application logs

You can also easily view *tailed* application logs, by running the following from the command line.

```sh
cf logs APP_NAME
```



