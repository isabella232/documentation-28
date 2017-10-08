---
title: /measurements
folder: api-docs
layout: api-docs
sidebar: api-docs_sidebar
permalink: api_measurements_post.html
position: 1.2
type: post
description: Create a Measurement
right_code: |
  ~~~ bash
  curl \
    -u $LIBRATO_USERNAME:$LIBRATO_TOKEN \
    -H "Content-Type: application/json" \
    -d '{
      "tags": {
        "region": "us-west",
        "name": "web-prod-3"
      },
      "measurements": [
        {
          "name": "cpu",
          "value": 4.5
        },
        {
          "name": "memory",
          "value": 10.5
        }
      ]
    }' \
  -X POST \
  https://metrics-api.librato.com/v1/measurements
  ~~~
  {: title="Curl" }

  ~~~ ruby
  require 'librato/metrics'
  Librato::Metrics.authenticate 'email', 'api_key'

  queue = Librato::Metrics::Queue.new
  queue.add "my.custom.metric" {
    value: 65,
    tags: {
      region: 'us-east-1',
      az: 'a'
    }
  }
  queue.submit
  ~~~
  {: title="Ruby" }

  ~~~ python
  import librato
  api = librato.connect('email', 'token')

  api.submit("my.custom.metric", 65, tags={'region': 'us-east-1', 'az': 'a'})
  ~~~
  {: title="Python" }
---

The code example shows how to create a new measurement `my.custom.metric` with the tag `region: "us-east-1", az: "a"`.

#### HTTP Request

`POST https://metrics-api.librato.com/v1/measurements`

This action allows you to submit measurements for new or existing metric data streams. You can submit measurements for multiple metrics in a single request.

If the metric referenced by the name property of a measurement does not exist, it will be created prior to saving the measurements. Any metric properties included in the request will be used when creating this initial metric. However, if the metric already exists the properties will not update the given metric.

For truly large numbers of measurements we suggest batching into multiple concurrent requests. It is recommended that you keep batch requests under 1,000 measurements/post. As we continue to tune the system this suggested cap will be updated.

#### Headers

The only permissible content type is JSON at the moment. All requests must include the following header:

content-type
: application/json
