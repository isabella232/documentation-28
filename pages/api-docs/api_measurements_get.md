---
title: /measurements
folder: api-docs
layout: api-docs
sidebar: api-docs_sidebar
permalink: api_measurements_get.html
position: 1
type: get
description: Get a list of measurement
---
name
: The unique identifying name of the property being tracked. The metric name is used both to create new measurements and query existing measurements.

tags
: A set of key/value pairs that describe the particular data stream. Tags behave as extra dimensions that data streams can be filtered and aggregated along. Examples include the region a server is located in, the size of a cloud instance or the country a user registers from. The full set of unique tag pairs defines a single data stream.

value
: The numeric value of a single measured sample.

time
: Unix Time (epoch seconds). This defines the time that a measurement is recorded at. It is useful when sending measurements from multiple hosts to align them on a given time boundary, eg. time=floor(Time.now, 60) to align samples on a 60 second tick.

period
: Define the period for the metric. This will be persisted for new metrics and used as the metric period for metrics marked for Service-Side Aggregation.

Measurements represent the individual time-series samples sent to the Librato service. They are associated with a metric, one or more tag pairs, and a point in time.

Each measurement has a value that represents an observation at one point in time. The value of a measurement will typically vary between some minimum or maximum value (ex. CPU usage moving from 0% to 100%).
