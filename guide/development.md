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

# RabbitMQ

DiSSCo communicates with MASs through an RabbitMQ messaging queue. RabbitMQ allows events, such as
tasks or data updates, to be sent between systems in a highly reliable and
scalable way. When a user schedules a MAS via the DiSSCover platform, DiSSCo dispatches a
message to the queue of the designated MAS. The auto-scaler will recognise there is a message pending and will start an
instance of them MAS.

For each MAS a specific RabbitMQ topic will be created. This topic is created when a new MAS is added through the
orchestration portal. For the MAS it is important that on start up the MAS starts to listen to the queue.
It can then start consuming messages one at a time. The resulting annotation of the MAS is also published to RabbitMQ.
Based on this message the annotation-processing service will be triggered which start processing the new annotations.

We have tried to make the integration with RabbitMQ as easy as possible.
The following boiler plate code can be used to start the listening:

```python
def run_rabbitmq() -> None:
    """
    Start a RabbitMQ consumer and process the messages by unpacking the image.
    When done, it will publish an annotation to annotation processing service
    """
    connection = pika.BlockingConnection(
        pika.ConnectionParameters(
            os.environ.get("RABBITMQ_HOST"),
            credentials=pika.PlainCredentials(os.environ.get("RABBITMQ_USER"), os.environ.get("RABBITMQ_PASSWORD")),
        )
    )
    channel = connection.channel()
    channel.basic_consume(queue=os.environ.get("RABBITMQ_QUEUE"), on_message_callback=process_message, auto_ack=True)
    channel.start_consuming()
```

The environmental variables (`RABBITMQ_HOST`, `RABBITMQ_USER`, `RABBITMQ_PASSWORD`,
`RABBITMQ_QUEUE`) are injected into the service by DiSSCo when the service is deployed.

This starts up the listener. Messages can then be processed in the process_message method:

```python

def process_message(channel: BlockingChannel, method: Method, properties: Properties, body: bytes) -> None:
    """
    Callback function to process the message from RabbitMQ. This method will be called for each message received.
    We publish this annotation through the channel on a RabbitMQ exchange.
    :param channel: The RabbitMQ channel, which we will use to publish the resulting annotation
    :param method: The method used to send the message, not currently used
    :param properties: Properties of the message, not currently used
    :param body: The message body in bytes
    :return:
    """
    json_value = json.loads(body.decode("utf-8"))
    try:
        shared.mark_job_as_running(json_value.get("jobId"))
        specimen_data = json_value.get("object")
        result = run_api_call(specimen_data)
        annotation_event = map_to_annotation_event(specimen_data, result, json_value.get("jobId"))
        publish_annotation_event(annotation_event, channel)
    except Exception as e:
        shared.send_failed_message(json_value.get("jobId"), str(e), channel)
```

Once the annotations are created the can be published by added the
`publish_annotation_event(annotation_event, channel)`:

```python

def publish_annotation_event(annotation_event: Dict, channel: BlockingChannel) -> None:
    """
    Send the annotation to the RabbitMQ queue
    :param annotation_event: The formatted annotation event
    :param channel: A RabbitMQ BlockingChannel to which we will publish the annotation
    :return: Will not return anything
    """
    logging.info("Publishing annotation: " + str(annotation_event))
    channel.basic_publish(
        exchange=os.environ.get("RABBITMQ_EXCHANGE", "mas-annotation-exchange"),
        routing_key=os.environ.get("RABBITMQ_ROUTING_KEY", "mas-annotation"),
        body=json.dumps(annotation_event).encode("utf-8"),
    )
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

2. **Multiple annotations**: The RabbitMQ message sent by your MAS must adhere to the annotation
   processing
   event
   ([schema](https://schemas.dissco.tech/schemas/developer-schema/annotation/latest/annotation-processing-event.json)).
   This event contains an array of annotations on the same target.

   **Use a list annotations when**: Your MAS has multiple insights on different parts of the target,
   or produces annotations with different motivations. For example, an AI service that classifies
   different segments of an image, or a taxonomic service that assesses different taxonomic fields (e.g. dwc:genus and
   dwc:species).

## If Your MAS has No Insights

If your MAS finds no results, that is still useful information for the user. A "no annotation"
annotation may provide useful insights into the target. If a plant organ detection tool finds no
plant organs, a species recognition tool can not identify the specimen, or a locality can not be
georeferenced, that information should still be captured in an annotation.

Qualities of a "no annotation" annotation

* **oa:motivation**: The motivation should be `oa:commenting`
* **ods:hasSelector**: The selector determines which field (s) of the target are targeted. Note that
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
Send the message to the RabbitMQ topic `mas-annotation-failed-exchange` with topic `mas-annotation-failed`, not the
topic specific to your MAS.

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

