---
layout: page
title: Step-by-step Guide
permalink: /guide
nav_order: 2
---

# Let's get started!

This guide will walk you through the steps of developing MAS middleware for your existing value
service. The process for developing a MAS middleware can be summarized as follows:

| Step                                                                                            | Section                                                                         
|:------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------|
| 1. Receive the target as JSON                                                                   | [Data Modelling](/mas-developers-documentation/guide/data-model)                |
| 2. Extract values from the target you need as input for your service, e.g. some OpenDS fields   | [Data Modelling](/mas-developers-documentation/guide/data-model)                | 
| 3. Create a function that adds value, usually calling an external API that is the value service | [Development](/mas-developers-documentation/guide/development)                  | 
| 4. Publish the output as an annotation event                                                    | [Development](/mas-developers-documentation/guide/development)                  |
| 5. Package your code into a Docker container                                                    | [Registration and Deployment](/mas-developers-documentation/guide/registration) |

{: .note }
> When an existing service is adapted to work within DiSSCo, there are two components involved:
>
> - **Value Service**: This is the original service being adapted to DiSSCo. It is deployed on
    infrastructure separate from the core DiSSCo architecture, and should be accessible through
    APIs.
> - **MAS Middleware**: This is a lightweight component containerized and deployed on the DiSSCo
    core
    architecture.

# Thank you!

Thank you for your commitment to enhancing the DiSSCo community through the development and
deployment of MASs. By following the guidelines outlined in this guide, you are playing a crucial
role in advancing biodiversity research. Whether through novel machine learning approaches,
georeferencing tools, or automated data checks, the contribution of help improve natural science
collection data quality across Europe. Machine Annotation Services not only support ongoing
research, but also empower the scientific community to engage in meaningful post-publication
curation. This collaborative approach enhances the value of digitized collections, ensuring they
remain relevant and up to-date long after their initial publication.