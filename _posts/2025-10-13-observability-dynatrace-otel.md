---
layout: post
title:  "Observability with Dynatrace and OpenTelemetry(OTEL) instrumentations"
date:   2025-10-12 10:00:00
categories: [Architecture, Observability, OpenTelemetry]
tags:
  - observability
  - dynatrace
  - open telemetry 
  - otel
read_time: true
excerpt: Dynatrace offers robust support for collecting OpenTelemetry (OTEL) data, including logs, from AWS Lambda functions and ECS containers
---
Dynatrace offers robust support for collecting OpenTelemetry (OTEL) data, including logs, from AWS Lambda functions and ECS containers. Here are examples of how to instrument these environments for logging with OTEL for Dynatrace.

1. AWS Lambda Log Enrichment with OpenTelemetry for Dynatrace:
To enrich your Lambda logs with trace and span IDs for contextualized logging in Dynatrace, you can modify your logging function to include this information.

Python Example:
``` python
import logging
from opentelemetry import trace

def dt_log(self, record):
    if (not self.disabled) and self.filter(record):
        ctx = trace.get_current_span().get_span_context()
        if ctx.is_valid:
            trace_id = "{0:032X}".format(ctx.trace_id)
            span_id = "{0:016X}".format(ctx.span_id)
            record.msg = f"[! dt dt.trace_id={trace_id},dt.span_id={span_id}] - {record.msg}"
        self.callHandlers(record)

logging.Logger.handle = dt_log

def lambda_handler(event, context):
    logger = logging.getLogger()
    logger.warning("Hello world from Lambda!")
    return {
        "statusCode": 200,
        "body": "Hello from lambda"
    }
  ```

  Node JS Example:

  ``` nodejs
  const opentelemetry = require('@opentelemetry/api');

function dtLog(msg) {
    const spanContext = opentelemetry.trace.getSpan(opentelemetry.context.active()).spanContext();
    if (spanContext.isValid()) {
        const traceId = spanContext.traceId;
        const spanId = spanContext.spanId;
        console.log(`[! dt dt.trace_id=${traceId},dt.span_id=${spanId}] - ${msg}`);
    } else {
        console.log(msg);
    }
}

exports.handler = async (event) => {
    dtLog("Hello world from Lambda!");
    return {
        statusCode: 200,
        body: 'Hello from Lambda!'
    };
};
```
2. ECS Container Logging with OpenTelemetry and FireLens for Dynatrace:
For ECS containers, you can leverage AWS FireLens with Fluent Bit to route logs and enrich them with OTEL data before sending them to Dynatrace.

ECS Task Definition (Partial Example):
```
{
  "containerDefinitions": [
    {
      "name": "your-app-container",
      "image": "your-app-image",
      "logConfiguration": {
        "logDriver": "awsfirelens",
        "options": {
          "Name": "stdout",
          "Match": "*",
          "Format": "json",
          "Labels": "trace_id=${OTEL_TRACE_ID},span_id=${OTEL_SPAN_ID}", // Example labels for enrichment
          "Output_Format": "json",
          "Output_Keys": "log,trace_id,span_id" // Include enriched fields in output
        }
      },
      "environment": [
        {
          "name": "OTEL_SERVICE_NAME",
          "value": "your-service-name"
        },
        {
          "name": "OTEL_EXPORTER_OTLP_ENDPOINT",
          "value": "your-dynatrace-otlp-endpoint"
        }
      ]
    },
    {
      "name": "log_router",
      "image": "amazon/aws-for-fluent-bit:latest",
      "firelensConfiguration": {
        "type": "fluentbit",
        "options": {
          "enable-ecs-log-metadata": "true"
        }
      }
    }
  ]
}
```

Further Reading:

Ingest from OTEL: 
https://docs.dynatrace.com/docs/ingest-from/opentelemetry

Lambda OTEL
https://docs.dynatrace.com/docs/ingest-from/amazon-web-services/integrate-into-aws/aws-lambda-integration/aws-lambda-classic/aws-lambda-otel-integration/aws-lambda-otel-setup

ECS OTEL
https://docs.dynatrace.com/docs/ingest-from/amazon-web-services/integrate-into-aws/aws-ecs
