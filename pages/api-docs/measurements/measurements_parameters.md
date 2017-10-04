---
title: Measurement Parameters
folder: api-docs
position: 1.1
type: 
description:
---

#### Top-Level Tags

The following payload demonstrates submitting tags at the top-level of the payload. This may be common for a collection agent that tags all metrics the same based on the identification of the collection host parameters.  If you add any tags to the measurement, those tags will replace the top-level tags.

This will result in two data streams, `cpu` and `memory`. Both metrics will contain the tags `region=us-west` and `name=web-prod-3`.

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

queue = Librato::Metrics::Queue.new(
  tags: {
    region: 'us-west',
    name: 'web-prod-3'
  }
)
queue.add cpu: 4.5
queue.add memory: 10.5

queue.submit
~~~
{: title="Ruby" }

~~~ python
import librato
api = librato.connect('email', 'token')

q = api.new_queue(tags={'region': 'us-west', 'name': 'web-prod-3'})
q.add('cpu', 4.5)
q.add('memory', 10.5)
q.submit()
~~~
{: title="Python" }


##### Embedded Measurement Tags

You can override top-level tags with per-measurement tags. In the following example, the `cpu` metric will replace the top-level tags `region:us-west` and `name:web-prod-3`with `name:web-prod-1`, while the `memory` metric will use the embedded tags `az:e` and `db:db-prod-1`.

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
        "value": 4.5,
        "tags": {
          "name": "web-prod-1"
        }
      },
      {
        "name": "memory",
        "value": 34.5,
        "tags": {
          "az": "e",
          "db": "db-prod-1"
        }
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

queue = Librato::Metrics::Queue.new(
  tags: {
    region: 'us-west',
    name: 'web-prod-3'
  }
)
queue.add cpu: {
  value: 4.5,
  tags: {
    name: "web-prod-1"
  }
}
queue.add memory: {
  value: 34.5,
  tags: {
    az: "e",
    db: "db-prod-1"
  }
}

queue.submit
~~~
{: title="Ruby" }

~~~ python
import librato
api = librato.connect('email', 'token')

q = api.new_queue(tags={'region': 'us-west', 'name': 'web-prod-3'})
q.add('cpu', 4.5, tags={'name': 'web-prod-1'})
q.add('memory', 34.5, tags={'az': 'e', 'db': 'db-prod-1'})
q.submit()
~~~
{: title="Python" }

##### Full Measurement Sample

Submit a single measurement that contains a full summary statistics fields. This includes embedded tag names and the metric attribute (which specifies not to enable [SSA](/docs/kb/data_processing/ssa/)) that is saved when the initial metric (cpu) is created.

~~~ bash
curl \
  -u $LIBRATO_USERNAME:$LIBRATO_TOKEN \
  -H "Content-Type: application/json" \
  -d '{
    "measurements": [
      {
        "name": "cpu",
        "time": 1421530163,
        "period": 60,
        "attributes": {
          "aggregate": false
        },
        "sum": 35.0,
        "count": 3,
        "min": 4.5,
        "max": 6.7,
        "last": 2.5,
        "stddev": 1.34,
        "tags": {
          "region": "us-east-1",
          "az": "b",
          "role": "kafka",
          "environment": "prod",
          "instance": "3"
        }
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
queue.add cpu: {
  time: 1421530163,
  period: 60,
  sum: 35,
  count: 3,
  min: 4.5,
  max: 6.7,
  last: 2.5,
  stddev: 1.34,
  attributes: {
    aggregate: false
  },
  tags: {
    region: "us-east-1",
    az: "b",
    role: "kafka",
    environment: "prod",
    instance: "3"
  }
}

queue.submit
~~~
{: title="Ruby" }

~~~ python
import librato
api = librato.connect('email', 'token')

#must set the value attribute to None
api.submit(
  "my.custom.metric",
  None,
  period=60,
  sum=35,
  count=3,
  min=4.5,
  max=6.7,
  last=2.5,
  stddev=1.34,
  attributes= {
    'aggregate': False
  },
  time= 1484613483,
  tags={
    'region': 'us-east-1',
    'role': 'kafka',
    'environment': 'prod',
    'instance': '3',
    'az': 'b'
  }
)
~~~
{: title="Python" }
#### Individual Sample Parameters

The request must include at least one measurement. Measurements are collated under the top-level parameter **measurements**. The minimum required fields for a measurement are:

name
: The unique identifying name of the property being tracked. The metric name is used both to create new measurements and query existing measurements. Must be 255 or fewer characters, and may only consist of `A-Za-z0-9.:-_`. The metric namespace is case insensitive.

value
: The numeric value of a single measured sample.

tags
: A set of key/value pairs that describe the particular data stream. Tags behave as extra dimensions that data streams can be filtered and aggregated along. Examples include the region a server is located in, the size of a cloud instance or the country a user registers from. The full set of unique tag pairs defines a single data stream. Tags are merged between the top-level tags and any per-measurement tags, see the section Tag Merging for details.

#### Global or per-sample parameters

In addition, the following parameters can be specified to further define the measurement sample. They may be set at the individual measurement level or at the top-level. The top-level value is used unless the same parameter is set at an individual measurement.

Parameter | Definition
--------- | ----------
time | Unix Time (epoch seconds). This defines the time that a measurement is recorded at. It is useful when sending measurements from multiple hosts to align them on a given time boundary, eg. time=floor(Time.now, 60) to align samples on a 60 second tick.
period | Define the period for the metric. This will be persisted for new metrics and used as the metric period for metrics marked for Service-Side Aggregation.

#### Summary fields

Measurements can contain a single floating point value or they can support samples that have been aggregated prior to submission (eg. with statsd) with the following summary fields:

Parameter | Definition
--------- | ----------
count | Indicates the request corresponds to a multi-sample measurement. This is useful if measurements are taken very frequently in a closed loop and the metric value is only periodically reported. If count is set, then sum must also be set in order to calculate an average value for the recorded metric measurement. Additionally min, max, and stddev/stddev_m2 may also be set when count is set. The value parameter should not be set if count is set.
sum | If count was set, sum must be set to the summation of the individual measurements. The combination of count and sum are used to calculate an average value for the recorded metric measurement.
max | If count was set, max can be used to report the largest individual measurement amongst the averaged set.
min | If count was set, min can be used to report the smallest individual measurement amongst the averaged set.
last | Represents the last value seen in the interval. Useful when tracking derivatives over points in time.
stddev | Represents the standard deviation of the sample set. If the measurement represents an aggregation of multiple samples, standard deviation can be calculated and included with the sample. Standard deviations are averaged when downsampling multiple samples over time.
stddev_m2 | Represents the current mean value when aggregating samples using the [alternative standard deviation method](#standard-deviation). The current mean value allows an accurate standard deviation calculation when aggregating multiple samples over time. Only one of stddev and stddev_m2 may be set.

#### Optional Parameters

**NOTE**: The [optional parameters](#update-a-metric) listed in the metrics PUT operation can be used with POST operations, but they will be ignored if the metric already exists. To update existing metrics, please use the PUT operation.

### Overriding Top-Level Tags

Measurements with embedded tags (specified per measurement) will override and prevent any top-level tags from being recorded for the specific measurement. In order to merge both top-level and embedded tags, all tags will need to be embedded with the measurement.

### Rate Limiting

Every response will include headers that define the current API limits related to the request made, the current usage of the account towards this limit, the total capacity of the limit and when the API limit will be reset.

### Measurement Restrictions

#### Name Restrictions

Metric names must be 255 or fewer characters, and may only consist of `A-Za-z0-9.:-_`. The metric namespace is case insensitive.

Tag names must match the regular expression `/\A[-.:_\w]+\z/{1,64}`. Tag names are always converted to lower case.

Tag values must match the regular expression `/\A[-.:_\\/\w ]{1,255}\z`. Tag values are always converted to lower case.

Data streams have a default limit of **50** tag names per measurement.

Users should be mindful of the maximum cardinality of their full
tag set over all measurements. Each unique set of <tag name, tag
value> pairs is a new unique stream and is billed as such. The
full cardinality of a metric is the permutation of all possible
values of tags over the billing period. For example, if you have
two tags on your measurements and the first tag has 20 possible
values and the second tag has 30 possible values, then your potential
tag cardinality could be 20 * 30 => 600 data streams. This would be
billed as 600 individual streams over the billing duration of one
hour.

If you plan to have a tag cardinality over 40,000 unique tag
sets per hour, please let us know ahead of time at support@librato.com. To
prevent accidental cardinality explosions our API may
automatically reject metrics with a cardinality exceeding this.

#### Float Restrictions

Internally all floating point values are stored in double-precision format. However, Librato places the following restrictions on very large or very small floating point exponents:

* If the base-10 exponent of any floating point value is larger than `1 x 10^126`, the request will be aborted with a 400 status error code.

* If the base-10 exponent of any floating point value is smaller than `1 x 10^-130`, the value will be truncated to zero (`0.0`).