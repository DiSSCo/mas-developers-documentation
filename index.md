---
layout: page
title: Home
permalink: /
nav_order: 1
---
# Welcome
{: .no_toc }

- TOC
{:toc}

Thank you for your interest in offering a Machine Annotation Service (MAS) for the DiSSCo community!
Here you'll find information on how to adapt existing services for the DiSSCo platform with minimal
effort.

# About DiSSCo

DiSSCo is a research infrastructure supporting **digitized natural science** collections. With over
200 partners in 23 countries, DiSSCo aims to digitally unify all European natural science assets.
It provides digital specimens and digital media objects from diverse collections in a **single
harmonised data model**, openDS. These harmonised data are made available through a user-friendly
platform, [DiSSCover](https://disscover.dissco.eu/), or through a [REST API]().

The DiSSCo core architecture aims to go beyond harmonising the data, however. Data within DiSSCo can
be evaluated, enriched, and extended by the community, improving data quality across institutional
boundaries. The way DiSSCo achieves this is through **annotations**.

# What is an Annotation?

An annotation is an additional piece of information associated with a specimen or media object. An
annotation is initially separate from its target. The annotation is then evaluated by experts, who
may reject or accept the annotation. An accepted annotation may then change the specimen or media
object in the source system. This approach allows for widespread community curation of the entire
digitized European collection.

*Also see
the [annotation JSON Schema](https://schemas.dissco.tech/schemas/fdo-type/annotation/latest/annotation.json)*

## What is a Machine Annotation Service?

A Machine Annotation Service (MAS) is an automated service that annotates a target in DiSSCo. These
services are scheduled by users on individual specimen or media in DiSSCo. What a MAS offers is
broad. From sophisticated AI services to taxonomic services to linking to other infrastructures,
MASs add value to natural science collections data in all sorts of ways. 

## What can be annotated?

In DiSSCo, two kinds of objects can be annotated: **digital specimens** or **media objects**. The
specific object being annotated is the "target" of the annotation.

What part of the target is being annotated is another important aspect of the annotation. The
annotation may be as broad, or as narrow, as necessary. This is called the "selector", and relates
specifically to the data according to the OpenDS specification.

- **Whole object**: The whole specimen or media is being annotated
- **Class**: A whole class (e.g. TaxonIdentification or Event) is being annotated
- **Term**: A specific, individual property (e.g. dwc:genus or dwc:locality)
- **Region of Interest**: (Media only) A specific area of the image is being annotated

*What terms can be annotated?* The [DiSSCo Terms Site](https://terms.dissco.tech/) has the most up
to-date information on openDS terms.

## Why make an annotation?

The "motivation" of an annotation, or the "why" of it all, is an important part of the annotation.
It determines how the annotation may change its target. The possible motivations are:

- **Adding**: The user wants to add new information to the specimen/media
- **Deleting**: A piece of information is no longer relevant to a specimen/media, and should be
  removed
- **Assessing**: The user wants to assess the quality of the data in the specimen/media (e.g. data
  quality
  flags)
- **Editing**: The user wants to edit an existing value of the specimen/media
- **Commenting**: The user wants to make a generic comment on the specimen/media

## Who can make annotations?

Human experts can manually annotate resources using DiSSCo's DiSSCover
platform. However, machines may annotate on a much larger scale. Machine Annotation Services (MASs)
are computational services that automatically produce annotations on digital objects. These services
may be triggered
automatically, when a digital object is first ingested into the system, or may be scheduled by a
user in DiSSCover. MASs often use external APIs or AI to produce their annotations.

# Why develop a MAS?

If you've developed a service that analyzes specimen data, you may be wondering why bother with
publishing it on
DiSSCo. Haven't you done enough work?

- **One data model to rule them all**: one service can be applied to data from hundreds of
  institutions
- **Modular design**: The DiSSCo architecture is designed to allow existing services to “plug in”
  easily, meaning anyone, not just those directly tied to DiSSCo, can develop a MAS
- **Ease of use**: Through the DiSSCover interface, even the least tech-savvy user can schedule
  sophisticated machine services on their data

These three factors mean that a single service can have a much wider reach through DiSSCo. Through
collaboration, we can further our shared mission of fighting the biodiversity crisis.