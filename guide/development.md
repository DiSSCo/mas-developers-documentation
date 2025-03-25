---
layout: page
title: Development
permalink: /guide/development
parent: Step-by-step Guide
nav_order: 2
---

# Development Guide

{: .no_toc }

Once you know your inputs and outputs of your MAS middleware, you're ready to start developing.

- TOC
{:toc}

# Apache Kafka

DiSSCo communicates with MASs through an Apache Kafka messaging queue. Kafka allows events, such as
tasks or data updates, to be sent between systems in a highly reliable and
scalable way. When a user schedules a MAS via the DiSSCover platform, DiSSCo dispatches a Kafka
message to the designated MAS, initiating the annotation process.

Kafka topics are unique names used to organize messages. Kafka producers write data to topics, and
consumers read data from topics. You will need to set up a kafka topic provided by
the DiSSCo team. Your MAS will send its result as a kafka message of the same topic. It is best to
store this topic name in an environment variable.

In python, you can easily set up a producer like so:

```python
import os
import json
from kafka import KafkaConsumer, KafkaProducer

consumer = KafkaConsumer(os.environ.get('KAFKA_CONSUMER_TOPIC'),
                         group_id=os.environ.get('KAFKA_CONSUMER_GROUP'),
                         bootstrap_servers=[os.environ.get('KAFKA_CONSUMER_HOST')],
                         value_deserializer=lambda m: json.loads(m.decode('utf-8')),
                         enable_auto_commit=True)
producer = KafkaProducer(bootstrap_servers=[os.environ.get('KAFKA_PRODUCER_HOST')],
                         value_serializer=lambda m: json.dumps(m).encode('utf-8'))
```

The environmental variables (`KAFKA_CONSUMER_TOPIC`, `KAFKA_CONSUMER_GROUP`, `KAFKA_CONSUMER_HOST`,
`KAFKA_PRODUCER_HOST`) are injected into the service by DiSSCo when the service is deployed.

You can capture incoming messages using the `consumer`. The following code will only run when a
message is sent with the topic defined previously.

```python

for msg in consumer:
  json_value = msg.value
  object_data = json_value['data']
```

# Templates

To get started on development, you can fork
the [MAS Template](https://github.com/DiSSCo/machine-annotation-service-template) on GitHub. The
`annotation` package contains code that will format a result forom an API to the openDS annotation
model. Two templates are provided: a default template and a batch template.

There are also some functional MASs available
on [GitHub](https://github.com/diSSCo/demo-enrichment-service-image/) you may use as a reference.

# `/running` Endpoint

As an added value service to the user, DiSSCo tracks the progress of a job through states:

- SCHEDULED
- RUNNING
- COMPLETED
- FAILED

When a MAS receives a message, it is strongly recommended to call the `/running` endpoint. This
indicates to DiSSCo the message has been received by the mas and the job is running. DiSSCo can then
inform the user of the development.

The endpoint has no body, and is reached at `/api/mjr/v1/{JOB-ID}`.

In deployment, DiSSCo automatically populates the `RUNNING_ENDPOINT` environmental variable with the
correct endpoint, depending on the environment is being run on.

- Test: `https://dev.dissco.tech/api/mjr/v1/{JOB-ID}`
- Acceptance: `https://sandbox.dissco.tech/api/mjr/v1/{JOB-ID}`
- Production: `https://api.dissco.eu/mjr/v1/{JOB_ID}`

# How Many Annotations Can You Return?

Your MAS may produce one, multiple, or no annotations on one target. This section explains how to
handle multiple or no annotations.

## If your MAS has Multiple Insights

Your MAS may have multiple, distinct contributions to a target. There are two ways to handle this:

1. **In an array**: `oa:value` is a field in the Body of the annotation which contains the insights
   your MAS produces. This field is an array, so multiple values may be added. However, all values
   in the body are part of the same annotation, meaning they have the same motivation, target,
   selector (what part of the target does the annotation apply to), and other parameters.

   **Use multiple bodies when**: Your MAS has multiple insights on the same part of the target, with
   the same motivation. Example: An AI service that provides two different classifications on the
   same region of interest.

2. **Multiple annotations**: The kafka message sent by your MAS must adhere to the annotation
   processing
   event ([schema](https://schemas.dissco.tech/schemas/developer-schema/annotation/latest/annotation-processing-event.json)).
   This event contains an array of annotations on the same target.

   **Use a list annotations when**: Your MAS has multiple insights on different parts of the target,
   or produces annotations with different motivations. For example, an AI service that classifies
   different segments of an image, or a taxonomic service that assesses different taxonomic fields (
   e.g. dwc:genus and dwc:species).

## If Your MAS has No Insights

If your MAS finds no results, that is still useful information for the user. A "no annotation"
annotation may provide useful insights into the target. If a plant organ detection tool finds no
plant organs, a species recognition tool can not identify the specimen, or a locality can not be
georeferenced, that information should still be captured in an annotation.

Qualities of a "no annotation" annotation

* **oa:motivation**: The motivation should be `oa:commenting`
* **ods:hasSelector**: The selector determines which field(s) of the target are targeted. Note that
  a `commenting` annotation may not be on a field that doesn't exist in the target.
    * The selector type may either be `ods:ClassSelector` or `ods:TermSelector`
    * Which field or class you target in this kind of annotation depends on your MAS, but
* **oa:value**: A simple message for the user indicating this job has no results: Examples:
    * "Unable to find a match"
    * "Too many potential matches"

## If your MAS Fails (Exception Handling)

If your MAS experiences an exception for whatever reason, that information should still be passed to
DiSSCo. That information is used to mark the job as `FAILED` and inform the user of any errors.

{: .note }
Send the message to the kafka topic `mas-failed`, not the topic specific to your MAS.

The failure message has the following structure:

```json
{
  "jobId": "20.5000.1025/AAA-BBB-CCC",
  "errorMessage": "Client error"
}
```

Where `jobId` is the job ID provided to your MAS in the request, and `errorMessage` is the error
message you want to pass to DiSSCo.

# Testing

Before your MAS is integrated into the DiSSCo architecture, you may test it locally. The easiest way
is to run your MAS on a target from DiSSCover, and compare the results against
the [annotation event schema](https://schemas.dissco.tech/schemas/developer-schema/annotation/latest/annotation-processing-request.json).
You can see `run_local()` methods in the demo enrichment services on GitHub, or use the following
example code as an example:

```python
import requests
import json
import logging


def run_local():
  response = requests.get(
    'https://sandbox.dissco.tech/api/digital-specimen/v1/SANDBOX/3L8-AS3-E1T')
  specimen = json.loads(response.content).get("data").get(
    'attributes')  # Extract data from API call
  result = run_mas(specimen)  # Run your MAS service
  annotation_event = map_to_annotation_event(specimen_data, result,
                                             str(
                                               uuid.uuid4()))  # Turn result into an annotation event
  logging.info("Created annotations: ", annotation_event)  # Validate this against schema
```

# Moving Forward - Checklist

- You know what selector to use, and what part of the target you're annotating
- You have a python script that accepts a target and outputs a valid annotation event
- Your MAS wrapper still captures "no result" information appropriately
- Your MAS sends an error message if an exception is raised
- You've tested your MAS locally, and it validates against the relevant schemas

