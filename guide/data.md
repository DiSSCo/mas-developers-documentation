---
layout: page
title: Data Modelling
permalink: /guide/data-model
parent: Step-by-step Guide
nav_order: 1
---

# Data Modelling

{: .no_toc }

- TOC
{:toc}

Before you develop adapt your service to DiSSCo, you need to think about the end result. What value
does your service add?

Some questions to consider:

- Does your service target digital specimens or media objects?
- If your service returns textual information, can its results be best mapped to terms or
  classes in OpenDS? Or does your service apply information to the entire target?
- If your service returns information or predictions about a region, how can its region be mapped to
  the region of interest selector in the annotation data model?
- What information does your service need to run? What fields in openDS can you send to your
  service?
- Does your MAS return the same output for the same input every time? Is it a service that makes
  sense to apply its result to multiple resources at the same time? Then you may want to consider
  batching annotations. See [Batching Annotations](#batching-annotations) for more information.

{: .note }
You can find more about openDS on the [OpenDS terms site](https://terms.dissco.tech/)

## What makes a good annotation?

When an annotation is accepted, it updates the specimen with new, valuable information.
A good annotation can be easily integrated into the target once it is accepted.

- `oa:value`: the value of the annotation ("**What** does the annotation say?")
- `ods:hasSelector`: the part of the target you're annotating ("**Where** is the annotation going?")
- `oa:motivation`: Motivation for what why the annotation was produced (**How** is the information
  integrated)

{: .note}
> The available motivations are: `ods:adding`, `ods:deleting`, `oa:assessing`, `oa:editing`,
`oa:commenting`.
> Only the `ods:adding` may reference a part of the target that doesn't exist yet because you are
> adding information to the target.
> Read more about [motivations](/mas-developers-documentation/#why-make-an-annotation)
> and [selectors](/mas-developers-documentation/#what-can-be-annotated)

### Example

Say you have a MAS that identifies taxonomy from an image. Your MAS should target the
`ods:hasTaxonIdentifications` class of a specimen.

*Note: Some fields have been removed from the example annotations for brevity*

**Bad - Commenting on the specimen**

The following annotation comments on a specimen's taxonomy.

- This annotation targets the entire first instance of the `TaxonIdentifications` class, instead of
  a specific field
- The comment is not machine actionable - this information can not be integrated directly into the
  specimen

While it does provide useful information, this annotation can not be integrated into the specimen
without additional work.

```json
{
  "oa:motivation": "oa:commenting",
  "oa:hasTarget": {
    "@id": "https://doi.org/10.3535/XYZ-XYZ-XYZ",
    "@type": "ods:DigitalSpecimen",
    "oa:hasSelector": {
      "@type": "ods:ClassSelector",
      "ods:term": "$['ods:hasTaxonIdentifications'][0]"
    }
  },
  "oa:hasBody": {
    "@type": "oa:TextualBody",
    "oa:value": [
      "Taxonomy should be Turdus pilaris Linnaeus, 1758"
    ],
    "ods:score": 0.9
  }
}
```

**Good - Editing or Adding information**

The following annotation targets a specific term (using the `ods:TermSelector`) to modify a specific
taxonomic field:

```json
{
  "oa:motivation": "ods:editing",
  "oa:hasTarget": {
    "@id": "https://doi.org/10.3535/XYZ-XYZ-XYZ",
    "@type": "ods:DigitalSpecimen",
    "oa:hasSelector": {
      "@type": "ods:TermSelector",
      "ods:term": "$['ods:hasTaxonIdentifications'][0]['dwc:scientificName']"
    }
  },
  "oa:hasBody": {
    "@type": "oa:TextualBody",
    "oa:value": [
      "Turdus pilaris Linnaeus, 1758"
    ]
  }
}
```

The following annotation adds a full TaxonIdentification:

```json

{
  "oa:motivation": "ods:editing",
  "oa:hasTarget": {
    "@id": "https://doi.org/10.3535/XYZ-XYZ-XYZ",
    "@type": "ods:DigitalSpecimen",
    "oa:hasSelector": {
      "@type": "ods:ClassSelector",
      "ods:term": "$['ods:hasTaxonIdentifications'][0]"
    }
  },
  "oa:hasBody": {
    "@type": "oa:TextualBody",
    "oa:value": {
      "@id": "https://www.catalogueoflife.org/data/taxon/92GWM",
      "@type": "ods:TaxonIdentification",
      "dwc:taxonID": "https://www.catalogueoflife.org/data/taxon/92GWM",
      "dwc:scientificName": "Nymphalis Kluk, 1802",
      "ods:scientificNameHTMLLabel": "<i>Nymphalis</i> Kluk, 1802",
      "dwc:scientificNameAuthorship": "Kluk, 1802",
      "dwc:namePublishedInYear": "1802",
      "dwc:taxonRank": "GENUS",
      "dwc:kingdom": "Animalia",
      "dwc:phylum": "Arthropoda",
      "dwc:class": "Insecta",
      "dwc:order": "Lepidoptera Linnaeus, 1758",
      "dwc:family": "Nymphalidae",
      "dwc:subfamily": "Nymphalinae",
      "dwc:genus": "Nymphalis",
      "dwc:taxonomicStatus": "ACCEPTED",
      "dwc:nomenclaturalCode": "ICZN",
      "dwc:superfamily": "Papilionoidea",
      "dwc:tribe": "Nymphalini"
    }
  }
}
```

In the above examples, the annotations can easily be integrated into the existing specimen.

{: .note }
When developing your MAS wrapper, consider the combination of selector, motivation, and content to
ensure your annotations can be integrated in the future.

# Data Schemas

JSON Schemas are hosted on our [schemas site](https://schemas.dissco.tech/schemas/). Refer to the
schemas site for the most up to-date information.

## Requests

Requests sent to the MAS will follow the following structure:

```json
{
  "jobId": "20.5000.1025/AAA-111-BBB",
  "object": {
  },
  "batchingRequested": true
}
```

- `jobId` is a persistent identifier (a Handle) generated by DiSSCo to track job state (e.g.
  running, failed, scheduled) for the
  user. **It must be returned unaltered to the DiSSCo architecture.**
- `object` contains the full digital specimen or media object, the target of the annotation.
- `batchingRequested`: While [batching](#batching-annotations) may reduce computational load, a MAS
  must be specifically designed with it in mind, as it requires additional metadata to support this
  functionality.

## Sending Information to DiSSCo

The response from the MAS to the DiSSCo Architecture must follow the following schema:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.dissco.tech/schemas/developer-schema/annotation/0.4.0/annotation-processing-event.json",
  "type": "object",
  "description": "Schema specific to Services (MAS or DiSSCover) providing information to DiSSCo's annotation processing service.",
  "properties": {
    "jobId": {
      "type": "string",
      "description": "Handle of the job record, if the annotation was produced by a Machine Annotation Service"
    },
    "annotations": {
      "type": "array",
      "description": "List of annotations produced by the MAS",
      "items": {
        "$ref": "https://schemas.dissco.tech/schemas/developer-schema/annotation/0.4.0/annotation-processing-request.json"
      }
    },
    "batchMetadata": {
      "type": "array",
      "description": "Object containing batch information, if result is a batch",
      "items": {
        "$ref": "https://schemas.dissco.tech/schemas/developer-schema/annotation/0.4.0/annotation-batch-metadata.json"
      }
    }
  },
  "additionalProperties": false,
  "required": [
    "jobId",
    "annotations"
  ]
}
```

- `jobId` is the unaltered jobId provided to the MAS in the request.
- `annotations` is the list of annotations produced by the MAS. A MAS may produce multiple
  annotations per job on the same object. The annotations must follow
  the [annotation schema for MAS developers](https://schemas.dissco.tech/schemas/developer-schema/annotation/0.4.0/annotation-processing-request.json)

# Filters

Filters are a valuable tool for ensuring that only relevant digital objects are processed by the
MAS. Based on the OpenDS specification, filters define the criteria an object must meet for the MAS
to be applied. A MAS can filter objects on any term in OpenDs. Filters are provided in JSON Path
Block notation and support wildcards.

When registering a MAS, providers can define filters to specify which objects their MAS will
annotate. If a MAS has a filter applied, it will only be available for resources that meet the
specified criteria. Therefore, it is strongly recommended to include filters to enhance the accuracy
and efficiency of the annotation process. This targeted approach helps maintain the quality and
relevance of annotations within the DiSSCo ecosystem.

## Example Filter Configuration

```json
{
  "$['ods:fdoType']": [
    "https://doi.org/21.T11148/894b1e6cad57e921764e"
  ],
  "$['dwc:basisOfRecord']": [
    "MaterialEntity"
  ],
  "$['ods:hasIdentifiers']": [
    "*"
  ]
}
```

In the example above, the MAS will only process objects that meet the following conditions:

- The FDO type is https://doi.org/21.T11148/894b1e6cad57e921764e (Digital Specimen).
- The basis of record is MaterialEntity.
- The object contains at least one identifier (non-null).

## Including Both Specimen and Media Filters

Your MAS may target media objects, but it may have limitations as to which types of subjects it can
process. For example, a MAS that looks at images of herbarium sheets. In that case, the target (the
media object) will not have information on what the specimen is.

**If your MAS requires information about the specimen and the media object, the specimen-specific
filters need to begin with `$['digitalSpecimen']`.**

{: .note}
To see which fields are in the media data model, and which fields are in the specimen data model,
see the [OpenDS terms page](https://terms.dissco.tech/)

A filter for a MAS that targets media objects, but needs the subject to be a botany specimen, may
have the following filters:

```json
{
  "$['ods:fdoType']": [
    "https://doi.org/21.T11148/bbad8c4e101e8af01115"
  ],
  "$['ac:accessURI']": [
    "*"
  ],
  "$['digitalSpecimen']['ods:topicDiscipline']": [
    "Botany"
  ]
}
```

This indicates the target fdo type is `https://doi.org/21.T11148/bbad8c4e101e8af01115` (media
object), the target has an access URI, and it is linked to a digital specimen with a topic
discipline "Botany".

## ods:FDOType

FDO (FAIR Digital Object) Types are blueprints for digital objects. In OpenDS, a unique, resolvable
identifier is used to specify Type of an object. Filtering by FDO Type is particularly useful, as it
helps distinguish between different kinds of objects in DiSSCo.

The two key Types in DiSSCo are:

- Digital
  Specimen: [https://doi.org/21.T11148/894b1e6cad57e921764e](https://doi.org/21.T11148/894b1e6cad57e921764e)
- Digital
  Media: [https://doi.org/21.T11148/bbad8c4e101e8af01115](https://doi.org/21.T11148/bbad8c4e101e8af01115)

# Batching Annotations

{: .warning }
Batching is still an experimental feature. Batching may produce an unintended number of
annotations without the proper batch metadata. Use with caution.

{: .note }
Batching a MAS annotation is a way to apply the **same annotation** to **multiple targets**. It is a
way to reduce wasted resources by only running the MAS once for a given output.

MAS providers can opt to enable their service for batch annotations, a feature particularly useful
for services that can be applied to multiple specimens simultaneously. Batch annotations help reduce
computational overhead by allowing DiSSCo to identify resources with identical input data. Instead
of running the MAS separately for each resource, DiSSCo applies the annotation across all relevant
resources, streamlining the process and conserving resources. For example, a MAS may take locality
information from a specimen and return a set of georeferenced coordinates. If this service is
batched, then all specimens with the same locality string will receive the same coordinate
annotation, with the original calculation only made once. Batching reduces load on the
original service, as it does not need to redundant calculations.

To enable this process, MASs can include batchMetadata in their response. The information provided
in the batchMetadata allows DiSSCo to generate search queries that identify resources matching the
criteria originally used to produce the annotation. This allows the system to apply the same
annotation across multiple resources without needing to re-run the MAS for each individual object.

{: .warning }
Batching is NOT suited for services which may produce different responses for the same input, e.g.
AI services. It is also not suited for services for which it doesn't make sense to apply to multiple
services. For instance, a service which finds identifiers in other infrastructures can not apply the
same result to different objects, as the result of the service is by nature unique to its original
target.

The schema for batchMetadata can be found here, and it includes the following key fields:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.dissco.tech/schemas/developer-schema/annotation/0.4.0/annotation-batch-metadata.json",
  "properties": {
    "ods:placeInBatch": {
      "description": "For batching only. Links batch metadata to specific annotations in an event. Value must correspond to an annotation's ods:placeInBatch",
      "type": "integer"
    },
    "searchParams": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "inputField": {
            "type": "string",
            "description": "JsonPath (mixed notation) of field used to base annotation reasoning on. Indexes must be wildcards",
            "examples": [
              "digitalSpecimenWrapper.occurrences[*].location.dwc:country"
            ]
          },
          "inputValue": {
            "type": "string",
            "description": "Value stored at the field indicated in inputField",
            "examples": [
              "Netherlands"
            ]
          }
        },
        "required": [
          "inputField",
          "inputValue"
        ],
        "additionalProperties": false
      },
      "minItems": 1
    }
  },
  "required": [
    "ods:placeInBatch",
    "searchParams"
  ],
  "additionalProperties": false
}
```

`placeInBatch`: Integer that indicates which annotation this batch metadata corresponds to. There
MUST be a corresponding "placeInBatch" value in one annotation in the event. If more than one
annotation
have the same placeInBatch value, only the first annotation will be used to create a base
annotation.

`inputField`: The full JSONPath of the field used to generate MAS annotation, in JSONPath block
notation,
e.g. ['ods:DigitalSpecimen']['ods:hasIdentifications'][*]['ods:hasTaxonIdentifications'][*]['dwc:taxonRank'].
Array indexes must be omitted - instead, use wildcards.

`inputValue`: value stored at the specified JSONPath.

Batching can only be done if the MAS sends annotations of one Type of object in one event - either
Digital Specimens OR Media Objects.

# Moving Forward - Checklist

- You've determined the most appropriate motivation for your service
- You've determined which openDS fields (which selector) should be used as input for your value service
- You've mapped the output of your value service to openDS terms
- You've established filters on what kind of objects your MAS will accept
